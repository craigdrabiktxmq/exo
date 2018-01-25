Exo Transaction Logging
=======================

Exo provides an easy-to-use transaction logger that can automatically log transactions to persistent storage in a blockchain data structure.  The logger groups transactions into blocks and writes those blocks to storage.  Each block is signed with an SHA-256 hash, and the block incorporates the hash of the previous block to maintain data integrity in the chain.

The logger uses a plug-in mechanism to allow the logger to work with any type of data storage.  A CouchDB plugin is provided with more implementations on the way.

## Initializing the logger
To initialize the logger, you create an instance of your storage-specific logger plugin and pass it to the logger.  The logger should be initialized in the init() method of your SwirldMain and can be done as part of the platform initialization:

```Java
String[] transactionProcessorPackages = {"com.txmq.exo.messaging.rest", "com.txmq.socketdemo.transactions"};
CouchDBBlockLogger blockLogger = new CouchDBBlockLogger(
        "zoo-" + platform.getAddress().getSelfName().toLowerCase(),
        "http",
        "couchdb",
        //"localhost",
        5984);
ExoPlatformLocator.init(platform, SocketDemoTransactionTypes.class, transactionProcessorPackages, blockLogger);
```

It can also be initialized separately, but should still be done in the init() method of your SwirldMain:
```java
ExoPlatformLocator.getBlockLogger.setLogger(blockLogger, "zoo-" + platform.getAddress().getSelfName().toLowerCase());
```