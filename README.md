Bitcoin Core integration/staging tree
=====================================

https://bitcoincore.org

* [Bitcoin binary version](https://bitcoincore.org/en/download/)

What is Bitcoin Core?
---------------------

* Bitcoin Core 
  * connects -- to the -- Bitcoin peer-to-peer network /
    * download blocks
    * fully validate blocks & transactions
  * == wallet + graphical UI
  * [doc](/doc)

Development Process
-------------------

* "master" branch
  * regularly 
    * built
      * see "doc/build-*.md"
    * tested
  * ❌NOT guarantee stable❌

* [GUI](https://github.com/bitcoin-core/gui)

Testing
-------

* bottleneck | development
  * testing
  * code review
* ⚠️security-critical project ⚠️

### Automated Testing

* | developers
  * write [unit tests](src/test/README.md) 

* types of tests
  * unit tests
    * `ctest`
      * run 
  * [regression and integration tests](/test)
    * written | Python
    * `build/test/functional/test_runner.py`
      * run tests
      * requirements
        * "build" == your build directory

* CI 
  * | onPR,
    * built for Windows, Linux, and macOS

### Manual Quality Assurance (QA) Testing

* test made by somebody != developer 

* recommended |
  * large or high-risk changes


Translations
------------

* [translations changes](https://www.transifex.com/bitcoin/bitcoin/)
