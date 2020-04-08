---
layout: post
title: Some tricks for bash script
categories: GNU
tags: [GNU, autotools, autoconf, automake]
---

This blog summarizes some tricks when writing bash script.

## set -euxo pipefail

### set -e

The -e option will cause a bash script to exit immediately when a command fails.

### set -o pipefail

This particular option sets the exit code of a pipeline to that of the rightmost command to exit with a non-zero status, or to zero if all commands of the pipeline exit successfully.

### set -u

This option causes the bash shell to treat unset variables as an error and exit immediately.

### set -x

The -x option causes bash to print each command before executing it.  Example usecase as follows.

``` bash
set -euxo pipefail

DEFAULT=5
RESULT=${VAR:-$DEFAULT}
echo "$RESULT"

```

## timeout command

This command help for cases we need to set up timeout for certain commands.  Examples as follows.

``` bash
timeout 600s some_command arg1 arg2
```

## Reference
[https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail/](https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail/)

[https://zhuanlan.zhihu.com/p/123989641](https://zhuanlan.zhihu.com/p/123989641)