# macOS Build Guide

**Updated for MacOS [15](https://www.apple.com/macos/macos-sequoia/)**

* goal
  * how to build 
    * bitcoind,
    * CL utilities,
    * GUI

## Requirements

### 1. Xcode Command Line Tools

* Xcode Command Line Tools
  * == collection of build tools
  * allows
    * 👀build Bitcoin Core -- from -- source👀
  * `xcode-select --install`
    * install it

### 2. Homebrew Package Manager

* install it

### 3. Install Required Dependencies

* `brew install cmake boost pkgconf libevent`

### 4. Clone Bitcoin repository

``` bash
git clone https://github.com/bitcoin/bitcoin.git
```

### 5. Install Optional Dependencies

#### Wallet Dependencies

* ❌if you want to run `bitcoind` OR `bitcoin-qt` -> NOT necessary to build wallet functionality❌ 

###### Descriptor Wallet Support

* `sqlite`
  * allows
    * support descriptor wallets
  * shipped by macOS

#### GUI Dependencies

###### Qt

* cross-platform Qt Framework
  * allows
    * 👀building GUI / included | Bitcoin Core👀  

* steps to build GUI
  * install 
    * Qt -- `brew install qt@6` --
    * [libqrencode](#libqrencode)
  * pass `-DBUILD_GUI=ON` 

* build with Qt binaries / downloaded -- from the -- Qt website
  * ❌NOT officially supported❌
    * [#7714](https://github.com/bitcoin/bitcoin/issues/7714)

###### libqrencode

* allows
  * GUI can encode addresses -- in -- QR codes
    * if you want to disable it -> pass `-DWITH_QRENCODE=OFF` 
* install libqrencode

    ``` bash
    brew install qrencode
    ```

#### ZMQ Dependencies

``` bash
brew install zeromq
```

* [further configuration](#further-configuration)
* [zmq.md](zmq.md)

### IPC Dependencies

* TODO: Compiling IPC-enabled binaries with `-DENABLE_IPC=ON` requires the following dependency.
Skip if you do not need IPC functionality.

```bash
brew install capnp
```

For more information on IPC, see: [multiprocess.md](multiprocess.md).

---

#### Test Suite Dependencies

* allows
  * | develop, 
    * test code changes
* requirements
  * `brew install python`

---

#### Deploy Dependencies

You can [deploy](#3-deploy-optional) a `.zip` containing the Bitcoin Core application.
It is required that you have `python` and `zip` installed.

## Building Bitcoin Core

### 1. Configuration

* EXIST MANY ways

##### Wallet (only SQlite) + GUI Support

``` bash
cmake -B build -DBUILD_GUI=ON
```
* 👀generated | ["build/"](/build)👀
* Problems:
  * Problem1: "Could NOT find QRencod"
    * Solution: `brew install qrencode`
  * Problem2: "Could NOT find Qt (missing: Qt6_DIR) (Required is at least version "6.2""
    * Solution: `brew install qt@6`

##### No Wallet or GUI

``` bash
cmake -B build -DENABLE_WALLET=OFF
```

##### Further Configuration

* AVAILABLE configuration options

    ``` bash
    cmake -B build -LH
    ```

### 2. Compile

* `cmake --build build`
  * if you want to use N parallel jobs -> use "-j N"
  * Problems:
    * Problem1: "/src/util/strencodings.h:390:50: error: no member named 'bit_cast' in namespace 'std'"
      * Solution: TODO:

* `ctest --test-dir build`
  * if you want to use N parallel tests -> use "-j N"
  * ⚠️if Python 3 is NOT available -> SOME tests are disabled⚠️  


### 3. Deploy (optional)

* create a  ".zip" / == `.app` bundle

``` bash
cmake --build build --target deploy
```

## Running Bitcoin Core

* create an EMPTY configuration file
  * BEFORE running

    ```shell
    mkdir -p "/Users/${USER}/Library/Application Support/Bitcoin"
    
    touch "/Users/${USER}/Library/Application Support/Bitcoin/bitcoin.conf"
    
    chmod 600 "/Users/${USER}/Library/Application Support/Bitcoin/bitcoin.conf"
    ```

* AVAILABLE | 
  * "./build/bin/bitcoind"
  * "./build/bin/bitcoin-qt"
    * support for the GUI 

* "./build/bin/bitcoin"
  * == multifunction CL interface /
    * support
      * `bitcoin node`,
      * `bitcoin gui`,
      * `bitcoin rpc`,
      * `bitcoin help`

* `bitcoind` or `bitcoin-qt`
  * FIRST time, download the blockchain
    * it could take [hours, days]

* blockchain & wallet data files
  * by default, stored |

    ``` bash
    /Users/${USER}/Library/Application Support/Bitcoin/
    ```

* monitor the download process

    ```shell
    tail -f $HOME/Library/Application\ Support/Bitcoin/debug.log
    ```

## Other commands:

```shell
./build/bin/bitcoind -daemon      # Starts the bitcoin daemon.
./build/bin/bitcoin-cli --help    # Outputs a list of command-line options.
./build/bin/bitcoin-cli help      # Outputs a list of RPC commands when the daemon is running.
./build/bin/bitcoin-qt -server # Starts the bitcoin-qt server mode, allows bitcoin-cli control
```
