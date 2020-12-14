# Cell Management

One of the unique challenges with the Cell Model is how to effectively manage Cells and the capacity contained within them. 

### Indexers

The Nervos CKB blockchain contains millions of Live Cells, and a developer must be able to locate the Cells they need to use in transactions. An indexer is a piece of software that helps speed up the process of locating Cells and allows the developer to query for Cells based on their attributes.

![](../.gitbook/assets/ckb-indexer.png)

An indexer runs as a background process or a separate daemon that continuously monitors a Nervos CKB node for new block data. As new blocks are found, the Cell information is extracted and organized internally by the indexer until needed. Dapp frontends and backends can then interface directly with the indexer to query for information about Cells.

Lumos has a built-in indexer as part of the framework. 

