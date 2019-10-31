---
layout: post
title: Add Samsung KVSSD to FIO ioengine
categories: Storage
tags: [kvssd, io stack, fio]
---

## Background
The motivation of this is to profile the latency profile of Samsung KVSSD (since currently I am collabrate with Samsung and have access to their latest KVSSD).  I did a little bit research on how to add new io-engine to FIO benchmark.  From this [post](https://www.spinics.net/lists/fio/msg05370.html), I found it fairly easy to add a new I/O engine.  I added a source file in **engine** folder (reference the mmap.c and sync.c to have a idea of the code).

## The code
I put the engine for KVSSD source code here.

```c
s = "Python syntax highlighting"
/*
 * KVssd engine
 *
 * Samsung KVSSD engine
 *
 */

#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include "kvs_api.h"

#include "../fio.h"

int counter = 0; 
extern pthread_mutex_t kvssd_lock; 

struct fio_kvssd_data {
    kvs_device_handle dev;
    kvs_container_handle cont_handle;
};

struct fio_kvssd_data *fkd;

static kvs_result kv_store(kvs_container_handle *cont_handle, char *key, int ksize, char *val, int vsize) {
    kvs_result ret ;
    kvs_store_option option;
    const kvs_key  kvskey = { (void *)key, (uint8_t)ksize};
    const kvs_value kvsvalue = { (void *)val, vsize, 0, 0 /*offset */};
    kvs_store_context put_ctx;
    option.st_type = KVS_STORE_POST;
    option.kvs_store_compress = false;

    put_ctx.option = option;
    put_ctx.private1 = NULL;
    put_ctx.private2 = NULL;
    ret = kvs_store_tuple(*cont_handle, &kvskey, &kvsvalue, &put_ctx);

    if (ret != KVS_SUCCESS) {
        printf("STORE tuple failed with err %s\n", kvs_errstr(ret));
        exit(1);
    }
    return ret;
}

static kvs_result kv_get(kvs_container_handle *cont_handle, const char *key, int ksize, char *vbuf, int vsize) {
    kvs_result ret ;
    const kvs_key  kvskey = { (void *)key, (uint8_t)ksize };
    kvs_retrieve_context ret_ctx;
    kvs_value kvsvalue = { vbuf, vsize , 0, 0 /*offset */}; //prepare initial buffer
    kvs_retrieve_option option;
    memset(&option, 0, sizeof(kvs_retrieve_option));
    option.kvs_retrieve_decompress = false;
    option.kvs_retrieve_delete = false;
    ret_ctx.option = option;
    ret_ctx.private1 = NULL;
    ret_ctx.private2 = NULL;
    ret = kvs_retrieve_tuple(*cont_handle, &kvskey, &kvsvalue, &ret_ctx);
    if(ret == KVS_ERR_KEY_NOT_EXIST) {
        printf("RETREIVE tuple failed with err %s\n", kvs_errstr(ret));
        return ret;
    }
    return ret;
}

static enum fio_q_status fio_kvssdio_queue(struct thread_data *td,
					  struct io_u *io_u)
{
    int ret;
	struct fio_file *f = io_u->file;
	struct fio_kvssd_data *fkd_l = (struct fio_kvssd_data *)FILE_ENG_DATA(f);

	fio_ro_check(td, io_u);

	if (io_u->ddir == DDIR_READ) {
        char key[16] = {0};
        sprintf(key, "%0*llu", 15, io_u->offset);
        kv_get(&(fkd_l->cont_handle), key, 16, io_u->xfer_buf, io_u->xfer_buflen);
    }

	else if (io_u->ddir == DDIR_WRITE) {
        char key[16] = {0};
        sprintf(key, "%0*llu", 15, io_u->offset);
        kv_store(&(fkd_l->cont_handle), key, 16, io_u->xfer_buf, io_u->xfer_buflen);
    }
	else if (io_u->ddir == DDIR_TRIM) {
		do_io_u_trim(td, io_u);
		return FIO_Q_COMPLETED;
	} else
		ret = do_io_u_sync(td, io_u);

	return FIO_Q_COMPLETED;
}

static int fio_kvssdio_open_file(struct thread_data *td, struct fio_file *f) {
    pthread_mutex_lock(&kvssd_lock); 
    if (counter++ == 0) {
        kvs_init_options options;
        kvs_container_context ctx;
        const char *configfile = "kvssd_emul.conf";
        fkd = (struct fio_kvssd_data*)calloc(1, sizeof(struct fio_kvssd_data));

        kvs_init_env_opts(&options);
        options.memory.use_dpdk = 0;
        // options for asynchronized call
        options.aio.iocoremask = 0;
        options.aio.queuedepth = 64;

        options.emul_config_file =  configfile;
        kvs_init_env(&options);

        kvs_open_device(f->file_name, &(fkd->dev));
        if (kvs_open_container(fkd->dev, "test", &(fkd->cont_handle)) == KVS_ERR_CONT_NOT_EXIST) {
            kvs_create_container(fkd->dev, "test", 4, &ctx);
            kvs_open_container(fkd->dev, "test", &(fkd->cont_handle));
        }
    }
    
    FILE_SET_ENG_DATA(f, fkd);
    pthread_mutex_unlock(&kvssd_lock); 
    return 0;
}

static int fio_kvssdio_close_file(struct thread_data *td, struct fio_file *f) {
    pthread_mutex_lock(&kvssd_lock); 
    if (--counter == 0) {
        kvs_close_container(fkd->cont_handle);
        kvs_close_device(fkd->dev);

	    free(fkd);
    }
    pthread_mutex_unlock(&kvssd_lock); 
    return 0;
}

static int fio_kvssdio_get_file_size(struct thread_data *td, struct fio_file *f) {
    f->real_file_size = (uint64_t)16 << 30;
    return 0;
}

static struct ioengine_ops ioengine = {
	.name		= "kvssd",
	.version	= FIO_IOOPS_VERSION,
	.queue		= fio_kvssdio_queue,
	.open_file	= fio_kvssdio_open_file,
	.close_file	= fio_kvssdio_close_file,
	.get_file_size	= fio_kvssdio_get_file_size,
	.flags		= FIO_SYNCIO ,
};

static void fio_init fio_kvssdio_register(void)
{
	register_ioengine(&ioengine);
}

static void fio_exit fio_kvssdio_unregister(void)
{
	unregister_ioengine(&ioengine);
}

```

## How to compile
I modify the Makefile a little bit to add the KVSSD library header. (not pretty)
```shell
make EXTLIBS="-lkvapi -L./kvssd" #./kvssd includes the libkvapi.so
```

## FIO manual
[FIO mannual](https://fio.readthedocs.io/en/latest/fio_man.html)