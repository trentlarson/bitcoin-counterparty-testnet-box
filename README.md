


Here’s my Counterparty build process… which doesn’t work dependably: when you run counterpartyd (the last step) it now complains about non-ASCII characters in the Python output, which wasn’t happening last Friday so I don’t know what the crap is going on.  Anyway, when we’ve got something working maybe we’ll make a trading_build repo that can build all these apps.  I’m trying this in a raw Ubuntu before I give up and move on.



    git clone https://github.com/trentlarson/bitcoin-counterparty-testnet-box.git
    cd bitcoin-counterparty-testnet-box

    docker build -t xcp-testnet counterparty

... which takes about 3 minutes, then:

    docker run -i -t -p 14000:14000 -p 19001:19001 xcp-testnet

... then, while inside docker:

    bitcoind -datadir=1 -daemon
    bitcoind -datadir=2 -daemon

... and, if you want to test that it’s running, you can do this:

    bitcoin-cli -datadir=1 getinfo
    bitcoin-cli -datadir=1 setgenerate true 101

... which takes about 10 seconds, then do this to see that there are 50 Bitcoin available:

    bitcoin-cli -datadir=1 getinfo

... and note that these even work outside docker if you add this argument: -rpcconnect=`boot2docker ip`

... and now you can run counterparty, e.g.:

    counterpartyd --config-file=/home/tester/.config/counterpartyd-testnet/counterpartyd.conf --data-dir=/home/tester/.config/counterpartyd-testnet server > nohup.out 2>&1 &

... but you’ll have to watch that nohup.out file because it’ll probably grow huge with a bunch of error messages… if it says “Resuming parsing” then it’s probably working for you — if so, let me know.




# bitcoin-testnet-box

This is a private, bitcoin, testnet-in-a-box.

You must have `bitcoind` and `bitcoin-cli` installed on your system and in the
path unless running this within a [Docker](https://www.docker.io) container
(see [below](#using-with-docker)).

[![docker stats](http://dockeri.co/image/freewil/bitcoin-testnet-box)](https://registry.hub.docker.com/u/freewil/bitcoin-testnet-box/)

## Starting the testnet-box

This will start up two nodes using the two datadirs `1` and `2`. They
will only connect to each other in order to remain an isolated private testnet.
Two nodes are provided, as one is used to generate blocks and it's balance
will be increased as this occurs (imitating a miner). You may want a second node
where this behavior is not observed.

Node `1` will listen on port `19000`, allowing node `2` to connect to it.

Node `1` will listen on port `19001` and node `2` will listen on port `19011`
for the JSON-RPC server.


```
$ make start
```

## Check the status of the nodes

```
$ make getinfo
bitcoin-cli -datadir=1  getinfo
{
    "version" : 90300,
    "protocolversion" : 70002,
    "walletversion" : 60000,
    "balance" : 0.00000000,
    "blocks" : 0,
    "timeoffset" : 0,
    "connections" : 1,
    "proxy" : "",
    "difficulty" : 0.00000000,
    "testnet" : false,
    "keypoololdest" : 1413617762,
    "keypoolsize" : 101,
    "paytxfee" : 0.00000000,
    "relayfee" : 0.00001000,
    "errors" : ""
}
bitcoin-cli -datadir=2  getinfo
{
    "version" : 90300,
    "protocolversion" : 70002,
    "walletversion" : 60000,
    "balance" : 0.00000000,
    "blocks" : 0,
    "timeoffset" : 0,
    "connections" : 1,
    "proxy" : "",
    "difficulty" : 0.00000000,
    "testnet" : false,
    "keypoololdest" : 1413617762,
    "keypoolsize" : 101,
    "paytxfee" : 0.00000000,
    "relayfee" : 0.00001000,
    "errors" : ""
}
```

## Generating blocks

Normally on the live, real, bitcoin network, blocks are generated, on average,
every 10 minutes. Since this testnet-in-box uses Bitcoin Core's (bitcoind)
regtest mode, we are able to generate a block on a private network
instantly using a simple command.

To generate a block:

```
$ make generate
```

## Stopping the testnet-box

```
$ make stop
```

To clean up any files created while running the testnet and restore to the
original state:

```
$ make clean
```

## Using with docker
This testnet-box can be used with [docker](https://www.docker.io/) to run it in
an isolated container.

### Building docker image

Pull the image
  * `docker pull freewil/bitcoin-testnet-box`

or build it yourself from this directory
  * `docker build -t bitcoin-testnet-box .`

### Running docker container
The docker image will run two bitcoin nodes in the background and is meant to be
attached to allow you to type in commands. The image also exposes
the two JSON-RPC ports from the nodes if you want to be able to access them
from outside the container.

* `$ docker run -t -i freewil/bitcoin-testnet-box`
