# Introduction to the Cell Model

In the last lab exercise, you may have noticed that the command you used to verify that your outputs is called `get_live_cell`. It's called this because, in Nervos' terminology, both inputs and outputs are canonically referred to as "cells".

A cell is the most basic structure needed to represent a single piece of state data. The design is inspired by Bitcoin's outputs, but cells have more flexible functionality. Cells can be used to represent any kind of on-chain asset type on Nervos, such as tokens, NFTs, and wrapped assets.

![](../.gitbook/assets/cell-model.png)

Below are the terms used for the Nervos Cell Model. We will stick to the Nervos terminology going forward, but know that it is not uncommon for others to use Bitcoin terminology when speaking about Nervos.

| Bitcoin | Nervos Cell Model |
| :--- | :--- |
| Input | Input Cell |
| Output | Output Cell |
| Unspent Output | Live Cell |
| Spent Output | Dead Cell |
| Spend | Consume  |

A cell can only be used as an input to a transaction a single time, just like we covered in the last lesson. A live cell is one that has not been used as an input cell and is available to be used. A dead cell is one that has already been consumed by using it as an input cell and is no longer available for use.

Every cell has an owner, and an individual can own any number of cells. In the illustration below, Alice, Bob, and Charlie each own several cells with different balances.

* Alice has two cells for a total of 300 CKBytes.
* Bob has two cells for a total of 700 CKBytes.
* Charlie has four cells for a total of 2,000 CKBytes.

![](../.gitbook/assets/cell-owners.png)

Let's say that Charlie wants to send 700 CKBytes to Alice. To create a transaction, the relevant live cells must be gathered for use as inputs in a process called cell collection. Charlie has four cells available that could be used, but none of them have enough to send Alice 700 CKBytes, so we will need to use multiple cells.

![](../.gitbook/assets/charlie-transaction.png)

During cell collection we need at least 700 CKBytes to pay Alice, so we gather two cells to cover that amount. Our total Input cells contain 1,000 CKBytes, but Charlie only wants to send 700 CKBytes to Alice. This means Charlie needs to send 300 CKBytes back to himself as change.

After the transaction has confirmed, the cells which were used as inputs will be consumed, and two new cells will be created. The new cells created in the transaction are outlined in red below. 

![](../.gitbook/assets/cell-owners-2.png)



