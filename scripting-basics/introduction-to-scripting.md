# Introduction to Scripting

Nervos allows for on-chain programmability using small programs known as   
"scripts". This is what allows Nervos to achieve smart contract functionality that is similar to other blockchain platforms. However, Nervos' approach is significantly different than most other platforms.

Nervos is based on the Cell Model, which we first introduced earlier. This is significantly different than the Account Model, which is used by most other platforms, including Ethererum. Both models can be used to create the same type of functionality, and build many of the same applications, but the approach that must be taken is conceptually very different.

### Account Model vs. Cell Model

Ethereum uses the Account Model, which is similar to having an account at the bank. Your account has a single number that represents your balance. You can also have balances for other tokens. Ultimately, every balance amounts to a single number that is attached to an account and every account is a representation of a user's public key or an on-chain smart contract. Every action that occurs on the blockchain can be described in a simplified way as a change to a balance on an account.

Nervos uses the Cell Model, which cannot be compared to an account at the bank. It's more like having multiple smaller sums of money stored in multiple different safes. Each safe might have a different amount of money, and there isn't a single number that represents the total amount you have. Just as we demonstrated earlier, your total balance is the total amount of value that is stored in all cells.

### Contract Oriented vs. Transaction Oriented

Ethereum's programming model is sometimes described as being part of a contract-oriented paradigm. The developer builds programs, known as contracts, which dictate how on-chain state changes occur. Methods on the contract are used to control state changes. An example would be the `transfer()` method of an ERC20 token contract, which would be used to send tokens from one account to another. A developer initiates a state change by creating a transaction that includes the method to call and supplies the necessary arguments to the method.

Nervos' programming model is transaction-oriented. There are no methods on a contract to call. The developer builds programs, known as scripts, which validate how an on-chain state change is allowed to occur. A developer initiates a state change by submitting a transaction that describes a valid state change.

![](../.gitbook/assets/contract-vs-transaction.png)

This image shows how a basic counter would be implemented on a contract-oriented model vs a transaction-oriented model. In both models, the counter can be increased by exactly 1 per transaction. In the contract-oriented model, a transaction contains a function call, and that call is executed on-chain to change the state. In the transaction-oriented model, the transaction contains both the old state and the new state that we want to change it to. The transaction is validated on-chain, and if it passes the state is updated.

In the example image, we don't use cells or talk about inputs and outputs. This is because we are describing the process at a high conceptual level so it can be more easily compared. On Nervos, the transaction would look more like this.

![](../.gitbook/assets/nervos-counter-transaction.png)

The transaction fully describes the end result, even before the transaction has been broadcasted to the network. In the left cell, you see a data value of 5. This is the state that currently exists on the blockchain. On the right is the new cell, with an increased value of 6. You could put whatever value you want in the right cell, but only a transaction with a value of 6 would confirm. This is because the details of the transaction are being validated by scripts. The scripts being used are indicated by the `Lock` and `Type` fields on a cell, which are short for Lock Script and Type Script, which we will explain more about momentarily. 

If this seems confusing or counter-intuitive, don't be discouraged. This is a brand new way of creating smart contracts, so it may be difficult to understand at first. It will become clear as we continue to work with it.

### Lock Scripts vs. Type Scripts

Nervos has two kinds of scripts, known as lock scripts and type scripts. Both are used to validate transactions, but their concerns different. 

The concern of a lock script is ownership. It verifies that the owner of the cell authorizes it to be used in a transaction. This is commonly done using signatures, but more advanced transactions can use other means to unlock a cell for use when special conditions are met. 

An example is the Anyone Can Pay lock script, or ACP for short. When you send CKBytes to someone, you must unlock the cell to extract the value. However, you must also unlock a cell to receive CKBytes from someone else in the same cell. The ACP lock allows this process to occur automatically when it detects that the cell value will increase rather than decrease. When taking value out of the cell, the ACP lock requires a signature to prove that they own the cell. When adding value to the cell, the ACP lock automatically unlocks and does not require a signature.

The concern of a type script is state transition. It verifies that the rules set by the developer are being followed in how state changes are allowed to occur in a transaction.

An example of type script usage is the creation of User Defined Tokens, or UDT for short. The type script includes all of the logic necessary to validate the formatting of the data inside a cell that represents a token, and the monetary policy of the token itself. 

When describing smart contract functionality, the logic must be divided between the lock script and type script. Sometimes you only need one or other and sometimes both are required. However, it's not an even split. Type scripts tend to include most of the functionality that is typically associated with a smart contract, while lock scripts are more commonly used.

Let's look at the counter transaction image again.

![](../.gitbook/assets/nervos-counter-transaction.png)

The lock script is assuring that only `address1` has the ability to unlock the value in this cell. They must provide a signature to unlock it. The `counter` type script is assuring that the data value is being updated by exactly 1 in each transaction. Since the current value is 5, only a value of 6 would be accepted.

Every cell requires a lock script, because every cell must have an owner. The owner could be a single person, multiple persons, or even no one, but lock script must always be present regardless. Every cell can also include a type script, but unlike the lock script, it is optional. A cell without a type script can hold CKBytes and data, but generally nothing more complex. A cell that holds an asset, such as a token, always has a type script to govern how it works.  

### Generation vs. Validation 

Any transaction on Nervos undergoes both generation and validation. A transaction describes a state change and is generated off-chain. When the transaction is broadcast to the network, it is validated on-chain by the lock scripts and type scripts.

Let's look at the counter transaction one last time.

![](../.gitbook/assets/nervos-counter-transaction.png)

This transaction would be generated off-chain. This could be done in several languages, but Javascript and Typescript are the most common. The transaction describes what should happen by indicating the current state \(inputs\) and the desired resulting state \(outputs\).

Once it is broadcasted to the network, the lock scripts and type scripts attached to the cells in the transaction will execute and validate the transaction.

Let's describe it another way by looking at the contract-oriented vs transaction-oriented image again.

![](../.gitbook/assets/contract-vs-transaction.png)

In the contract-oriented version, the generator creates a transaction that describes what on-chain method should be called to update the state. It creates a transaction which indicates that the `contract.inc()` method should be called. When the transaction is executed on-chain, it calls the `contract.inc()` method to change the state. This method will retrieve the current state value, increase it by 1, then update the state.

In the transaction-oriented version, that same logic is still used, but it distributed differently between the off-chain and on-chain components. The off-chain generator calls the `contract.inc()` method which will retrieve the current state value, and increase it by 1. But it can't save it directly since this is executing off-chain. To save the new state value, it must form a transaction that describes the state change, then broadcast it to the network. The on-chain scripts then validate that the indicated change is valid, then update the state.

This approach is less intuitive at first, but it has several distinct advantages.

* **Scalability** - The process of generation is usually more computationally intensive than validation. We can take advantage of this asymmetry to achieve better scalability. Moving generation off-chain will allow higher on-chain efficiency, resulting in higher TPS on equal hardware.
* **Deterministic** - The resulting state is created by the generator and exists in the transaction before it is sent to the network. There is no possibility of side-effects, and there are no surprises. We can always be assured of the desired outcome once the transaction is executed on-chain.
* **No Erroneous Transactions** - Any transaction that does not validate is immediately rejected by the network. The invalid transaction is not stored on-chain, and no transaction fees are paid. This reduces the storage cost of the blockchain and eliminates scenarios where a user submits a transaction that results in an error, but they still had to pay a transaction fee that isn't refunded.

### Scripts as Identification

Lock scripts and type scripts are used to add functionality to cells, but they also provide a second purpose, to provide a means of identifying and locating cells.

A cell is a relatively simple structure that contains the four fields we mentioned earlier:

* Capacity
* Data
* Lock Script
* Type Script

Looking at these four fields, it might seem like there is no way of distinguishing one type of cell from another. Every cell has an out point, but that is a location in the blockchain. There are millions of cells in the blockchain, and searching through all the out points would be infeasible.

When searching for cells that exist on the blockchain, the lock script and type script are the most common way of locating cells.

The lock script validates authority. In more simplistic terms, a lock script represents a particular owner. If we search for cells with a specific lock script, we are searching for cells with a specific owner.

The lock script validates state transitions. In simplistic terms, a type script defines the behavior of a cell. If we search for cells with a specific type script, we are searching for cells that all exhibit common behavior; a specific type of cell.

