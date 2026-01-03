# Bitcoin Core file system

## Data directory location

* data directory location
  * == location | Bitcoin Core files are stored
  * 👀ALL content is chain-specific👀
    * EXCEPT TO "bitcoin.conf" 

Platform | DEFAULT Data directory path
---------|--------------------
Linux    | `$HOME/.bitcoin/`
macOS    | `$HOME/Library/Application Support/Bitcoin/`
Windows  | `%LOCALAPPDATA%\Bitcoin\` <sup>[\[1\]](#note1)</sup>

Chain option                     | Data directory path
---------------------------------|------------------------------
`-chain=main` (default)          | *path_to_datadir*`/`
`-chain=test` or `-testnet`      | *path_to_datadir*`/testnet3/`
`-chain=testnet4` or `-testnet4` | *path_to_datadir*`/testnet4/`
`-chain=signet` or `-signet`     | *path_to_datadir*`/signet/`
`-chain=regtest` or `-regtest`   | *path_to_datadir*`/regtest/`

* `-datadir` option
  * allows
    * specifying custom data directory path 


## Data directory layout

Subdirectory       | File(s)               | Description                                                                                                                                                                                                                                                                   
-------------------|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
`blocks/`          |                       | == Blocks directory </br> if you want to specify -> `-blocksdir` option </br> ❌NOT ALLOWED `blocks/index/`❌                                                                                                                                                                   
`blocks/index/`    | LevelDB database      | == Block index </br> ⚠️INDEPENDENT of `-blocksdir`⚠️                                                                                                                                                                                                                          
`blocks/`          | `blkNNNNN.dat`<sup>[\[2\]](#note2)</sup> | CURRENT Bitcoin blocks / dumped \| network format </br> 128 MiB / file                                                                                                                                                                                                        
`blocks/`          | `revNNNNN.dat`<sup>[\[2\]](#note2)</sup> | Block undo data                                                                                                                                                                                                                                                               
`blocks/`          | `xor.dat`             | Rolling XOR pattern -- for -- block & undo data files                                                                                                                                                                                                                         
`chainstate/`      | LevelDB database      | Blockchain state == ALL CURRENT UTXOs (\| compact representation) + transactions' metadata / UTXO come from                                                                                                                                                                  
`indexes/txindex/` | LevelDB database      | Transaction index </br> OPTIONAL </br> if `-txindex=1` -> use it                                                                                                                                                                                                                           
`indexes/blockfilter/basic/db/` | LevelDB database      | TODO: Blockfilter index LevelDB database for the basic filtertype; *optional*, used if `-blockfilterindex=basic`                                                                                                                                                                    
`indexes/blockfilter/basic/`    | `fltrNNNNN.dat`<sup>[\[2\]](#note2)</sup> | Blockfilter index filters for the basic filtertype; *optional*, used if `-blockfilterindex=basic`                                                                                                                                                                             
`indexes/coinstats/db/` | LevelDB database | Coinstats index; *optional*, used if `-coinstatsindex=1`                                                                                                                                                                                                                      
`wallets/`         |                       | [Contains wallets](#multi-wallet-environment); can be specified by `-walletdir` option; if `wallets/` subdirectory does not exist, wallets reside in the [data directory](#data-directory-location)                                                                           
`./`               | `anchors.dat`         | Anchor IP address database, created on shutdown and deleted at startup. Anchors are last known outgoing block-relay-only peers that are tried to re-connect to on startup                                                                                                     
`./`               | `banlist.json`        | Stores the addresses/subnets of banned nodes.                                                                                                                                                                                                                                 
`./`               | `bitcoin.conf`        | User-defined [configuration settings](bitcoin-conf.md) for `bitcoind` or `bitcoin-qt`. File is not written to by the software and must be created manually. Path can be specified by `-conf` option                                                                           
`./`               | `bitcoind.pid`        | Stores the process ID (PID) of `bitcoind` or `bitcoin-qt` while running; created at start and deleted on shutdown; can be specified by `-pid` option                                                                                                                          
`./`               | `debug.log`           | Contains debug information and general logging generated by `bitcoind` or `bitcoin-qt`; can be specified by `-debuglogfile` option                                                                                                                                            
`./`               | `fee_estimates.dat`   | Stores statistics used to estimate minimum transaction fees required for confirmation                                                                                                                                                                                         
`./`               | `guisettings.ini.bak` | Backup of former [GUI settings](#gui-settings) after `-resetguisettings` option is used                                                                                                                                                                                       
`./`               | `ip_asn.map`          | IP addresses to Autonomous System Numbers (ASNs) mapping used for bucketing of the peers; path can be specified with the `-asmap` option                                                                                                                                      
`./`               | `mempool.dat`         | Dump of the mempool's transactions                                                                                                                                                                                                                                            
`./`               | `onion_v3_private_key` | Cached Tor onion service private key for `-listenonion` option                                                                                                                                                                                                                
`./`               | `i2p_private_key`     | Private key that corresponds to our I2P address. When `-i2psam=` is specified the contents of this file is used to identify ourselves for making outgoing connections to I2P peers and possibly accepting incoming ones. Automatically generated if it does not exist.        
`./`               | `peers.dat`           | Peer IP address database (custom format)                                                                                                                                                                                                                                      
`./`               | `settings.json`       | Read-write settings set through GUI or RPC interfaces, augmenting manual settings from [bitcoin.conf](bitcoin-conf.md). File is created automatically if read-write settings storage is not disabled with `-nosettings` option. Path can be specified with `-settings` option 
`./`               | `.cookie`             | Session RPC authentication cookie; if used, created at start and deleted on shutdown; can be specified by `-rpccookiefile` option                                                                                                                                             
`./`               | `.lock`               | Data directory lock file                                                                                                                                                                                                                                                      

## Multi-wallet environment

* Wallets
  * == 💡SQLite databases💡
  1. if you use a user-defined wallet / 
     * name == "wallet_name" -> placed | "wallets/wallet_name/"
     * unnamed -> placed "wallets/"
       * if this directory does NOT exist -> placed | [data directory](#data-directory-location)
  2. if you want to specify the path -> use `-wallet` option
  3. "wallet.dat"
     * NOT share ACROSS DIFFERENT node instances
       * Reason:🧠 OTHERWISE -> it can cause: key-reuse & double-spends (NOT synchronization BETWEEN instances)🧠
  4. if you want to copy OR backup the wallet -> use `backupwallet` call 
     * Reason:🧠update & lock the wallet -> prevent file corruption🧠


### SQLite database based wallets

Subdirectory | File                | Description
-------------|---------------------|-------------
`./`         | "wallet.dat"         | == personal wallet (== SQLite database) / contains keys & transactions
`./`         | `wallet.dat-journal` | SQLite Rollback Journal file for `wallet.dat`. Usually created at start and deleted on shutdown. A user *must keep it as safe* as the `wallet.dat` file.


## GUI settings

`bitcoin-qt` uses [`QSettings`](https://doc.qt.io/qt-6/qsettings.html) class; this implies platform-specific [locations where application settings are stored](https://doc.qt.io/qt-6/qsettings.html#locations-where-application-settings-are-stored).

## Legacy subdirectories and files

These subdirectories and files are no longer used by Bitcoin Core:

Path           | Description | Repository notes
---------------|-------------|-----------------
`banlist.dat`  | Stores the addresses/subnets of banned nodes; superseded by `banlist.json` in 22.0 and completely ignored in 23.0 | [PR #20966](https://github.com/bitcoin/bitcoin/pull/20966), [PR #22570](https://github.com/bitcoin/bitcoin/pull/22570)
`blktree/`     | Blockchain index; replaced by `blocks/index/` in [0.8.0](https://github.com/bitcoin/bitcoin/blob/master/doc/release-notes/release-notes-0.8.0.md#improvements) | [PR #2231](https://github.com/bitcoin/bitcoin/pull/2231), [`8fdc94cc`](https://github.com/bitcoin/bitcoin/commit/8fdc94cc8f0341e96b1edb3a5b56811c0b20bd15)
`coins/`       | Unspent transaction output database; replaced by `chainstate/` in 0.8.0 | [PR #2231](https://github.com/bitcoin/bitcoin/pull/2231), [`8fdc94cc`](https://github.com/bitcoin/bitcoin/commit/8fdc94cc8f0341e96b1edb3a5b56811c0b20bd15)
`blkindex.dat` | Blockchain index BDB database; replaced by {`chainstate/`, `blocks/index/`, `blocks/revNNNNN.dat`<sup>[\[2\]](#note2)</sup>} in 0.8.0 | [PR #1677](https://github.com/bitcoin/bitcoin/pull/1677)
`blk000?.dat`  | Block data (custom format, 2 GiB per file); replaced by `blocks/blkNNNNN.dat`<sup>[\[2\]](#note2)</sup> in 0.8.0 | [PR #1677](https://github.com/bitcoin/bitcoin/pull/1677)
`addr.dat`     | Peer IP address BDB database; replaced by `peers.dat` in [0.7.0](https://github.com/bitcoin/bitcoin/blob/master/doc/release-notes/release-notes-0.7.0.md) | [PR #1198](https://github.com/bitcoin/bitcoin/pull/1198), [`928d3a01`](https://github.com/bitcoin/bitcoin/commit/928d3a011cc66c7f907c4d053f674ea77dc611cc)
`onion_private_key` | Cached Tor onion service private key for `-listenonion` option. Was used for Tor v2 services; replaced by `onion_v3_private_key` in [0.21.0](https://github.com/bitcoin/bitcoin/blob/master/doc/release-notes/release-notes-0.21.0.md) | [PR #19954](https://github.com/bitcoin/bitcoin/pull/19954)

### Berkeley DB database based wallets

Subdirectory | File(s)           | Description
-------------|-------------------|-------------
`database/`  | BDB logging files | Part of BDB environment; created at start and deleted on shutdown; a user *must keep it as safe* as personal wallet `wallet.dat`
`./`         | `db.log`          | BDB error file
`./`         | `wallet.dat`      | Personal wallet (a BDB database) with keys and transactions
`./`         | `.walletlock`     | BDB wallet lock file

## Notes

<a name="note1">1</a>  
* `/`
  * == U+002F
  * uses | this document,
    * path component separator / platform-independent

<a name="note2">2</a>
* `NNNNN`
  * == `[0-9]{5}` regex
