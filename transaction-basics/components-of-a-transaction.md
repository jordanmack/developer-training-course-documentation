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



