# docker-ftc-core :money_with_wings:
*This comprises a work in progress*

`docker-ftc-core` provides a small docker image to get up and running as
quick as possible when working with the feathercoin daemon.

run
```bash
docker run \
    -p 9336:9336 \
    -p 9337:9337 \
    -p 19336:19336 \
    -p 19337:19337 \
    matthewhartstonge/ftc-core:debian
```

## Debian
The Debian image fires up and runs default `feathercoind` config straight 
out of the box.

### Configuration
You can configure your daemon via mapping in a `feathercoin.conf` file.
The `feathercoin.conf` file is a key=value map containing the same 
keys as found when running `feathercoind -help` 

#### Example Configuration File
This file gets you up and running and *IS NOT PRODUCTION READY*. 

It is also worth noting that options `rpcuser` and `rpcpassword` will 
be deprecated in future in favour of `rpcauth` which requires generating 
a hashed username/password.

`feathercoin.conf`
```
server=1
txindex=1
rest=1
rpcbind=0.0.0.0
rpcallowip=0.0.0.0/0
rpcuser=tester
rpcpassword=testing123
wallet=/ftc
walletdir=/ftc/wallets
```

* Esit and make `feathercoin.conf` locally
* Make a local directory for `/ftc`
* Make a local directory for `/ftc/wallets` 

After creating the folders and config locally, we can bind it into 
our docker image:

```bash
docker run \
    -p 9336:9336 \
    -p 9337:9337 \
    -v /path/to/dev-feathercoin.conf:/root/.feathercoin/feathercoin.conf \
    -v /path/to/store/wallet/and/chain:/ftc \
    matthewhartstonge/ftc-core:debian
```


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