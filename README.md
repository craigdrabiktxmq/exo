Exo Framework
=============

Exo is a framework for developing applications on the Swirlds Hashgraph platform.  Primarily, Exo offers features and an application architecture for getting transactions into a Hashgraph, processing those transactions once they've been accepted, and logging transactions out to persistent storage.

Exo was developed to provide the features that Swirlds' SDK lacks that I knew I would need to develop real-world applications.  I'm sure other folks have needs and ideasthat aren't on my radar yet, so I would encourage feature requests and development submissions from other Hashgraph developers.  The framework will only improve for everyone as more people get involved.

Features
--------

Exo implements the following feature set:
- Socket-based messaging for Java applications
- REST-based messaging for anything that can make HTTP requests
- An architecture for structuring transaction processing logic and automated routing of transactions to transaction processors
- A framework for logging processed transactions to blockchain-based (blockchain the data structure, not blockchain the platform)

How To Build Applications on Exo
--------------------------------

### Getting Started
The easiest way to get started is to replicate or repurpose one of the Swirlds SDK demo applications.  Download the SDK from (https://www.swirlds.com/download/) and unzip to the directory of your choice.  You can install Exo in one of two ways:
* Download zipped source and extract the zip file's contents.  Copy the "exo" directory into your source tree - it should be copied to <my source root>/com/txmq so that the directory structure lines up with the package structure.
* Use git subtrees to link the exo source to your project.  Check out https://medium.com/@porteneuve/mastering-git-subtrees-943d29a798ec for a tutorial on how git subtrees work.  Again, you want the exo folder to fall under <source root>/com/txmq when you set up your subtree.

**The ability to add the framework components as Maven dependencies is coming**

### Initialize the platform
Exo needs a reference to the Swirlds platform to run, and it also needs some configuration to happen before it's ready for use.  Each Swirlds application has a main class that inheirits from SwirldsMain.  In your application's main class, initialize the platform in the init() method:
```java
ExoPlatformLocator.init(platform);
```

#### Block Logging
Exo provides functionality for logging transactions to a blockchain (the data structure, not the platform).  Exo includes an adaptor for CouchDB.  The logging mechanism is pluggable if you want to build support for other types of data stores.  You can initialize the logger for CouchDB like so:
```java
CouchDBBlockLogger blockLogger = new CouchDBBlockLogger(
    "zoo-" + platform.getAddress().getSelfName().toLowerCase(),
    "http",
    "localhost",
    5984);

ExoPlatformLocator.getBlockLogger().setLogger(blockLogger, platform.getAddress().getSelfName());
```
#### Transaction Routing
Exo can automatically route messages to the methods that process them.  Annotate each method with the transaction type it handles:
```java
@ExoTransaction(ExoTransactionType.ANNOUNCE_NODE)
public void announceNode(ExoMessage message, ExoState state) { 
	state.addEndpoint((String) message.payload);
}
```

Then in the SwirldsMain init() method, initialize the router with a list of packages your handlers reside in.  
```java
new SocketDemoTransactionType(); //Ensures that the keys for all your transactions have been defined before initializing.
ExoPlatformLocator.getTransactionRouter().addPackage("com.txmq.socketdemo.transactions");
```

#### REST API
Exo can expose REST API endpoints on your Hashgraph nodes, e.g. for web clients.  Set up the mechanisms you'll use to accept incoming messages from client applications.  No Java Enterprise Edition or web application server is required.  Annotate your REST handler methods with the standard javax.ws.rs.* metadata.  You must specify the port for the API to be exposed on, and a list of packages that hold your API processing code.  
```java
ExoPlatformLocator.initREST(12345, new String[] {"com.txmq.socketdemo.rest"});
```

#### Socket API
Exo supports a Java socket-based mechanism for communicating between Java applications and the Hashgraph.  Socket message routing works on the same mechanism as transaction routing, using the same @ExoTransaction annotation.  Then in the SwirldsMain init() method, initialize the socket mechanism with a port number to listen on, and a list of packages your handlers reside in:
```java
ExoPlatformLocator.initSocketMessaging(23456m new String[] {"com.txmq.socketdemo.socket"});
```

ExoPlatformLocator contains several constructors which can be used as shorthand when initializing the platform.
```java
//Initialize the platform and transaction routing
ExoPlatformLocator.init(platform, SocketDemoTransactionType, new String[] {"com.txmq.socketdemo.transactions"});

//Initialize the platform, transaction routing, and block logging
String[] transactionProcessorPackages = {"com.txmq.exo.messaging.rest", "com.txmq.socketdemo.transactions"};
CouchDBBlockLogger blockLogger = new CouchDBBlockLogger(
		"zoo-" + platform.getAddress().getSelfName().toLowerCase(),
		"http",
		"couchdb",
		//"localhost",
		5984);
ExoPlatformLocator.init(platform, SocketDemoTransactionTypes.class, transactionProcessorPackages, blockLogger);
```

### More Information
There's more to learn about building Exo applications.  First, check out the (Exo Demo Application)[https://github.com/craigdrabiktxmq/exo-demo] and see how the demo is put together.  It illustrates all of the concepts currently supported by Exo, and is dockerized so you can easily run the application stack.

When you're ready to dive into development, check the package folders for additional README files that describe how to use each feature in more detail.
