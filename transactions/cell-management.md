# Working with Cell Collection

One of the unique challenges with the Cell Model is how to effectively manage cells and the capacity contained within them. The Nervos CKB blockchain contains millions of live cells, and a developer must be able to locate the cells they need both for their own accounts, and for the accounts of the users they support in their dapps.&#x20;

### Indexers

An indexer is a piece of software that helps speed up the process of locating cells and allows the developer to query for cells based on their attributes.

![](../.gitbook/assets/ckb-indexer.png)

An indexer runs as separate node that continuously monitors a Nervos CKB node for new block data. As new blocks are found, the cell information is extracted and organized internally by the indexer until needed. Dapp frontends and backends can then interface directly with the indexer to query for information about cells.

{% code lineNumbers="true" %}
```javascript
const {initializeConfig} = require("@ckb-lumos/config-manager");
const {Indexer} = require("@ckb-lumos/ckb-indexer"); 
const config = require("./config.json");

const NODE_URL = "http://127.0.0.1:8114/";
const INDEXER_URL = "http://127.0.0.1:8116/";

initializeConfig(config);
const indexer = new Indexer(INDEXER_URL, NODE_URL);

(async function()
{
    console.log(await indexer.tip());
})();
```
{% endcode %}

On line 8 we start with `initializeConfig(config)`. This uses the `config.json` file in your current working directory to initialize Lumos.

On line 9 we create a new instance of `Indexer` which will pass requests to the CKB Indexer node JSON RPC which we specified.

Finally, on line 13 we use `indexer` to retrieve and display the most recent tip block on the console.

### Automated Cell Collection

Up until this point, we have been manually doing cell collection through `ckb-cli` or by using the outputs of transactions we just recently created. Of course, this is not an effective way of doing things in a real dapp. Cell collection needs to be done quickly and automatically.

Lumos has a class called `CellCollector` which is designed to help with cell collection, but it requires some additional code to be used for our purposes. Here is the `collectCapacity` function that exists in the main shared library of the Developer Training Course repo `lib/index.js`.

{% code lineNumbers="true" %}
```javascript
/**
 * Collects cells for use as capacity from the specified lock script.
 * 
 * This will search for cells with at least capacityRequired. If there is insufficient capacity available an error will be thrown.
 * 
 * @example
 * const {inputCells, inputCapacity} = await collectCapacity(indexer, addressToScript("ckt1qyqvsv5240xeh85wvnau2eky8pwrhh4jr8ts8vyj37"), ckbytesToShannons(100n));
 * 
 * @param {Object} indexer An instance of a running Lumos Indexer.
 * @param {Object} lockScript A script used to query the CellCollector to find cells to use as capacity.
 * @param {BigInt} capacityRequired The number of CKBytes necessary.
 * 
 * @returns {Object} An object with the inputCells[] found and the inputCapacity contained within the provided Cells.  
 */
async function collectCapacity(indexer, lockScript, capacityRequired)
{
	const query = {lock: lockScript, type: null};
	const cellCollector = new CellCollector(indexer, query);

	let inputCells = [];
	let inputCapacity = 0n;

	for await (const cell of cellCollector.collect())
	{
		inputCells.push(cell);
		inputCapacity += hexToInt(cell.cell_output.capacity);

		if(inputCapacity >= capacityRequired)
			break;
	}

	if(inputCapacity < capacityRequired)
		throw new Error("Unable to collect enough cells to fulfill the capacity requirements.");

	return {inputCells, inputCapacity};
}
```
{% endcode %}

This function is used to collect cells for use as capacity in a transaction. It uses a `CellCollector` instance to query the indexer to find live cells.

Looking at line 15, it takes the following arguments:

* `indexer` is an instance of the Lumos indexer that is initialized and fully synced with a Nervos CKB node.
* `lockScript` is something we will cover in one of the next lessons. For now, think of it as the owner of a cell.
* `capacityRequired` is the amount of CKBytes, in Shannons, that are needed to complete our transaction.

Looking at line 17 we see this:

```javascript
const query = {lock: lockScript, type: null};
```

This JSON object is describing attributes of cells that we want to locate. In this case, they are cells which are owned by the specified `lockScript` and do not have a Type Script.

The rest of the code should be fairly easy to understand. It continuously gathers live cells that match the query until we have the required capacity, or it errors if there are not enough cells to meet the requirement.

### Capacity Management

Let's say that Charlie wants to send Bob 100 CKBytes. If Charlie had a cell that contained exactly enough CKBytes, this would be a very straight forward transaction.

![](../.gitbook/assets/cell-capacity-management.png)

In this transaction, Charlie uses a cell that has exactly 100.0001 CKBytes. Exactly 100 CKBytes is sent to Bob, and the remaining 0.0001 CKBytes is used at the transaction fee. It is very unlikely that this scenario would occur in reality, since the exact amounts present in cell are very unlikely to match the exact amounts needed for the transaction.

![](../.gitbook/assets/cell-capacity-management-2.png)

Here is a slightly more realistic transaction example. Cell collection was performed to gather at least 100.0001 CKBytes to send to Bob and pay transaction fees. Two cells were found for 65 CKBytes and 75 CKBytes, for a total of 140 CKBytes.  Now there is enough capacity to pay Bob 100 CKBytes, pay a 0.0001 CKByte transaction fee, and the remaining 39.9999 CKBytes can be sent back to Charlie as change.

However, this transaction has a problem and would be invalid. Can you spot the problem?

The problem with this transaction is that the change cell has 39.9999 CKBytes, but as we covered earlier, the minimum capacity of a cell is 61 CKBytes. This is because a cell must have enough capacity to cover its own overhead for data storage, which is 61 bytes for a basic cell.

To solve this, another round of cell collection must occur to gather enough capacity to properly structure this transaction.

![](../.gitbook/assets/cell-capacity-management-3.png)

Cell collection continues, and a third input cell is found with 90.0001 CKBytes. Now there is enough input capacity to create the change cell and this transaction would be successful.
