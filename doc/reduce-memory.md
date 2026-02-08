# Reduce Memory Usage of `bitcoind`

* use cases
  * embedded systems
  * small VPSes

## In-memory caches

* As MORE memory is used by caches -> BETTER performance
* as LOWER memory is used by caches -> longer sync time
  * MAINLY | initial sync

- `-dbcache=<n>`
  - == UTXO database cache size
    - by default, `450`
    - \>= 4
  - unitS: MiB (1024)

## Memory pool

* `-maxmempool=<n>` 
  * == memory pool limiter
  * `<n>`
    * == size | MB (1000)
  * ALLOWED | Bitcoin Core 
  * by default, `300`
  * \>= 5
  * as lower -> transactions will be evicted sooner
* This will affect any uses of `bitcoind` that process unconfirmed transactions.

- The unused memory allocated to the mempool (default: 300MB) is shared with the UTXO cache, so when trying to reduce memory usage you should limit the mempool, with the `-maxmempool` command line argument.

- To disable most of the mempool functionality there is the `-blocksonly` option
* This will reduce the default memory usage to 5MB and make the client opt out of receiving (and thus relaying) transactions, except from peers who have the `relay` permission set (e.g
* whitelisted peers), and as part of blocks.

  - Do not use this when using the client to broadcast transactions as any transaction sent will stick out like a sore thumb, affecting privacy
* When used with the wallet it should be combined with `-walletbroadcast=0` and `-spendzeroconfchange=0`. Another mechanism for broadcasting outgoing transactions (if any) should be used.

## Number of peers

- `-maxconnections=<n>` - the maximum number of connections, which defaults to 125. Each active connection takes up some
  memory. This option applies only if inbound connections are enabled; otherwise, the number of connections will not
  be more than 11. Of the 11 outbound peers, there can be 8 full-relay connections, 2 block-relay-only ones,
  and occasionally 1 short-lived feeler or extra outbound block-relay-only connection.

- These limits do not apply to connections added manually with the `-addnode` configuration option or
  the `addnode` RPC, which have a separate limit of 8 connections.

## Thread configuration

For each thread a thread stack needs to be allocated. By default on Linux,
threads take up 8MiB for the thread stack on a 64-bit system, and 4MiB in a
32-bit system.

- `-par=<n>` - the number of script verification threads, defaults to the number of cores in the system minus one.
- `-rpcthreads=<n>` - the number of threads used for processing RPC requests, defaults to `16`.

## Linux specific

By default, glibc's implementation of `malloc` may use more than one arena
* This is known to cause excessive memory usage in some scenarios
* To avoid this, make a script that sets `MALLOC_ARENA_MAX` before starting bitcoind:

```bash
#!/usr/bin/env bash
export MALLOC_ARENA_MAX=1
bitcoind
```

The behavior was introduced to increase CPU locality of allocated memory and performance with concurrent allocation, so this setting could in theory reduce performance
* However, in Bitcoin Core very little parallel allocation happens, so the impact is expected to be small or absent.
