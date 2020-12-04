# Sending a Transaction with Multiple Inputs and Outputs

### Lesson Introduction

In this lesson, we will introduce Nervos' Cell Model, and use it to generate a transaction using code. You will need the outpoints you verified from the last lab exercise, so make sure you have them handy.

### The Cell Model

In the last lab exercise, you may have noticed that the command you used to verify that your outputs is called `get_live_cell`. It's called this because, in Nervos' terminology, both inputs and outputs are canonically referred to as "cells". Below is are the terms used for the Nervos Cell Model.

| Bitcoin | Nervos Cell Model |
| :--- | :--- |
| Input | Input Cell |
| Output | Output Cell |
| Unspent Output | Live Cell |
| Spent Output | Dead Cell |
| Spend | Consume |

A Cell is the most basic structure needed to represent a single piece of state data. The design is inspired by Bitcoin's outputs, but Cells have more powerful functionality.

A Cell is composed of four basic components: Capacity, Data, Lock Script, and Type Script.

* **Capacity** is the number of CKBytes held within the Cell. It's called capacity because it represents the amount of on-chain space that the Cell can occupy. If your Cell takes up 100 bytes of space, then it needs to have a capacity of at least 100, which means it must contain at least 100 CKBytes.
* **Data** is any kind of data you want to store in the on-chain state. This can be data in any form, but the two most common are smart contract binaries and smart contract data.
* **Lock Script** is a small program that determines who owns and has control over the Cell. The default Lock Script on Nervos uses the Secp256k1 algorithm, which is the same as Bitcoin and Ethereum. But unlike Bitcoin and Ethereum, a developer has complete control over the algorithms, and can change them if needed.
* **Type Script** is a small program that validates state changes any time the Cell is included in a transaction. A Type Script is what enables small contract like functionality within the Cell Model.

All on-chain assets, such as tokens, NFTs, and wrapped assets, are all represented as Cells.

![](../.gitbook/assets/cell-model.png)

### 

