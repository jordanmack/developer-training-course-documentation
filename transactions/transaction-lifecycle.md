# Transaction Lifecycle

Most transactions on Nervos have a very similar lifecycle. Regardless of which framework and tools are used, the general process of working with transactions is similar. 

#### Create an Empty Transaction

Most frameworks you will work with will start with some kind of a scaffold to produce a transaction. Across different frameworks and libraries, this may be called by different names. Some may call it a "skeleton",  "builder", "raw transaction", or simply a "transaction". The syntax may be different, but the purpose is generally the same. It is an empty box into which the components of a transaction will be placed.

#### Add Input Cells

The next step is usually to add Input Cells. I say "usually" because sometimes Output Cells are added first. The order doesn't matter to the framework, but depending on the particulars of the transaction it may be easier to add one before the other to calculate capacity requirements and tx fees.

Live Cells are gathered through the process of Cell Collection and added as Inputs within the transaction. These Cells are already on-chain, so they are represented by their Out Point; the transaction ID and index from where the Cell originates. Often times frameworks will abstract this away allowing the developer to work with some kind of a Live Cell instance.

#### Add Output Cells

Output Cells are Cells that will be created after the transaction confirms. They do not exist on-chain yet, so there is no Out Point to reference. The developer must configure the Cell with all details as necessary and then add it to the transaction.

After all the Output Cells are added to the transaction the total capacity required by the outputs will be known. If the capacity provided by the Input Cells is not enough, then a second round of Cell Collection may need to occur to add more input capacity. 

#### Add Change and TX Fees

Once the Input Cells and Output Cells are added to the transaction the capacity can be totaled for the Inputs and Outputs. In most cases, the capacity provided by the Input Cells and required by the Output Cells will not be an exact match. The capacity of the Input Cells will exceed the requirements of the Output Cells and the transaction fees. The left over capacity needs to be sent back to the original owner by creating a Change Cell and adding it to the transaction. 

#### Add Dependencies

All transactions will have at least one dependency in the form of a Cell Dep or Header Dep. We will cover what these are and how to use them in the later lessons. For now, know that they are resources needed by the transaction. This can come in the form of smart contract binaries, libraries, modules, information about the blockchain itself, or many forms of data, like oracles.

#### Add Signatures

After the Input Cells have been added to the transaction, authorization needs to be provided for those Cells. To do this, the transaction is serialized and hashed by the library or framework to create a signing message. This message is signed by the private keys which own the Input Cells, and the resulting signature is added to the Witnesses of the transaction. If there are multiple Input Cells in the transaction that have different owners, then one signature is required from each of the owner's private keys.

Since the transaction is serialized and hashed before generating the signing message, any change to the transaction after signing would invalidate the signatures provided. A new signing message would need to be generated and signed again by the owners of the Input Cells. This is important because it ensures that a signature cannot be copied from one transaction to another in a way that the signer didn't intend for.

#### Broadcast the Transaction

The transaction may be completed, but until it is sent to the network no changes will occur on-chain. This is done by submitting the transaction to a CKB node using the RPC. The CKB node will validate the transaction, then broadcast it to the rest of the CKB nodes on the network.

#### Wait for Confirmation

When the transaction is broadcasted it isn't confirmed immediately. It will reside in the mempool, which is kind of like a waiting room for transactions that haven't been added to the blockchain yet. When a miner finds a block, they include transactions from the mempool, then broadcast the completed block to the network. Only after the block has been propagated and accepted by the rest of the network can it be considered confirmed.

