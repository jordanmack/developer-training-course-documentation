# Components of a Valid Transaction

Transactions are the basis for anything that occurs on-chain. Let's look at what constitutes a valid transaction, and what the lifecycle is for a transaction.

### Valid Transactions

Any change in state that occurs is the result of a valid transaction being submitted and accepted by the network. On Nervos, only transactions that are valid are recorded to the blockchain. This is different than on Ethereum, and similar platforms, where even a transaction that results in error is still recorded to the blockchain.

As a developer, your goal is to always produce valid transactions. To do this consistently, you need to know the rules to follow.

#### At Least One Input Cell Must Exist

In order for a transaction to be considered valid, there must be at least one Input Cell. Remember, a transaction is how a change in state is described. Without an Input Cell, there is no state to change, and no way to pay transaction fees.

#### All Inputs Must be Authorized

All Input Cells in the transaction must be authorized to be consumed. This means they must be Live Cells, and in most cases, this means that the transaction must be signed using the private keys of the owners of the Input Cells. This is very similar to other blockchains, which also rely on private keys for authorization. However, Nervos is much more flexible in how authorization can be provided, opening new possibilities which we will cover in a later lesson.

#### Capacity Requirements Must be Met

Every Cell must have capacity equal or greater than the number of bytes occupied by the Cell on the blockchain. This includes any assets or data held within the Cell, as well as the overhead of the Cell's data structure itself. In most cases, this means that the minimum capacity required by a basic Cell is 61 bytes.

The capacity requirement exists both at the Cell level and the transaction level. In order for an Output Cell to have 61 bytes of capacity, there must be an Input Cell with at least 61 bytes of capacity. If the total capacity of the Output Cells exceeds that of the Input Cells, then the transaction is invalid.

#### All Scripts Must Execute Successfully

Nervos uses small programs called "scripts" to achieve smart contract functionality. Each Cell can include an optional "Type Script" to include custom logic. We will cover Type Scripts in detail in the later lessons. The important takeaway right now is that when a transaction executes, all Cells with Type Scripts must execute successfully without error. If even one script in the transaction returns an error, then the entire transaction is invalid.

#### Transaction Fees

Every transaction that is submitted to the network must include a fee paid in CKBytes. This fee is paid to miners for verifying and processing transactions and for providing security to the network. Fees are based both on the size of the transaction, and the amount of computing resources required to process it. Just like with other blockchains, a fee market is used to prioritize transactions that have paid a higher fee rate. However, unlike most other blockchains, transaction fees are not the only economic incentive for miners. The result is less upward pressure on transaction fees, allowing them to remain lower without sacrificing security. We'll learn how to calculate transaction fees in a later lesson.

### Transaction Lifecycle

Most transactions have a very similar lifecycle on Nervos. Regardless of which framework and tools are used, the process of working with transactions will be similar. 

#### Create an Empty Transaction

Most frameworks you will work with will start with some kind of a scaffold to produce a transaction. Across different frameworks and libraries this may be called by different names. Some may call it a "skeleton",  "builder", "raw transaction", or simply a "transaction". The syntax may be different, but the purpose is generally the same. It is an empty box into which the components of a transaction will be placed.

#### Add Input Cells

The next step is usually to add Input Cells. I say "usually" because sometimes Output Cells are added first. The order doesn't matter to the framework, but depending on the particulars of the transaction it may be easier to add one before the another to calculate capacity requirements and tx fees.

Live Cells are gathered through the process of Cell Collection and added as Inputs within the transaction. These Cells are already on-chain, so they are represented by their Out Point; the transaction ID and index from where the Cell originates. Often times frameworks will abstract this away allowing the developer to work with some kind of a Live Cell instance.

#### Add Output Cells

Output Cells are Cells that will be created after the transaction confirms. They do not exist on-chain yet, so there is no Out Point to reference. The developer must configure the Cell as necessary and then add it to the transaction.

#### Add Change and TX Fees

Once the Input Cells and Output Cells are added to the transaction the capacity can be totaled for the Inputs and Outputs. In most cases, the capacity provided by the Input Cells and required by the Output Cells will not be an exact match. The capacity of the Input Cells will exceed the requirements of the Output Cells and the transaction fees. The left over capacity needs to be sent back to the original owner by creating a Change Cell and adding it to the transaction. 

#### Add Signatures

After the Input Cells have been added to the transaction, authorization needs to be provided for the Input Cells which were added to the transaction. To do this, the transaction is serialized and hashed by the library or framework to create a signing message. This message is signed by the private keys which own the Input Cells, and the resulting signature is added to the Witnesses of the transaction. If there are multiple Input Cells in the transaction that have different owners, then one signature is required from each of the owner's private keys.

The transaction is serialized and hashed before generating the signing message. This ensures that the transaction cannot change after the signatures were provided. If the transaction does change, then the existing signatures are invalid. A new signing message would need to be generated and signed again by the owners of the Input Cells.

#### Broadcast the Transaction

The transaction may be completed, but until it is sent to the network no changes will occur on-chain. This is done by submitting the transaction to a CKB node using the RPC. The CKB node will validate the transaction, then broadcast it to the rest of the CKB nodes on the network.

#### Wait for Confirmation

When the transaction is broadcasted it isn't confirmed immediately. It will reside in the mempool, which is kind of like a waiting room for transactions that are waiting for acceptance. When a miner finds a block, they include transactions from the mempool, then broadcast the completed block to the network. Only after the block has been propagated and accepted by the rest of the network can it be considered confirmed.



