* `WalletBatch`
  * == abstract modifier object -- for the -- wallet database /
    * 's methods
      * update
      * act | database
    * requirements
      * agnostic -- to the -- database implementation
    * access (== opens + R/W access) -- to the -- wallet database
      * OWN transaction / EACH read and write 
    * if you want MULTIPLE operation transactions
      * `TxnBegin()`
        * start
      * `TxnCommit()`
        * commit 
    * TODO: Otherwise the transaction will be committed when the object goes out of scope.
  * Optionally (on by default) it will flush to disk on close.
  * AUTOMATICALLY trigger a flush to disk / EACH 1000 writes
  * "*IC"
    * == incremental counter
      * -- based on -- write OR erase pass or fail

* `static const int VERSION_WITH_HDDATA=10;`    // TODO: why this number? 1. NOT found any specified convention, 2. HDD was deployed | v0.13.0