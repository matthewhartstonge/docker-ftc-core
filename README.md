# docker-ftc-core :money_with_wings:
*This comprises a work in progress*

`docker-ftc-core` provides a small docker image to get up and running as
quick as possible when working with the feathercoin daemon.

## Debian
*in progress*

## Alpine Linux
Comprises a minimilistic alpine-linux musl compiled container containing:

* feathercoind
* feathercoin-cli
* feathercoin-tx

### Known Issues
Currently, there is a segfault when calling `neoscrypt()` on first load, 
but if you BYOB (bring your own blockchain) it loads and runs... ? 

If you have any ideas around how to fix this, please feel free to submit 
a PR or issue detailing how to work around the issue. If you are looking 
into this, check out [the forum where the issue has been documented][forumissue]
and help the community out! :ok_hand:

### Debug
****
```bash
$ docker run --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -it ftcd sh
/ # apk add gdb
/ # gdb feathercoind
GNU gdb (GDB) 8.0.1
...

(gdb) run
Starting program: /usr/bin/feathercoind
2018-12-07T06:24:10Z Feathercoin Core version v0.17.0.1 (release build)
~ debuggy output things ~

```

[forumissue]: https://forum.feathercoin.com/topic/9872/alpine-segfault-when-calling-neoscrypt