# Storing Data in a Cell

A Cell can be used to store any kind of data that is desired by the developer. However, the cost of storing state data on a globally decentralized blockchain is high when compared to normal or cloud-based storage. This means that some types of data are more appropriate than others.

| Typically Appropriate | Typically Inappropriate |
| :--- | :--- |
| Script Code \(Smart Contracts\) | Images |
| Dapp State Data | Movies |
| Token Balances | Music |
| Oracle Data | PDFs |
| Data Hashes |  |

### Costs of State Storage

Bitcoin often uses the comparison of blockchain storage to prime real estate. This comparison is also used for Nervos, but in an even stronger sense. Possessing one CKByte gives the holder the right to store one byte of data in the blockchain state. This makes a CKByte similar to a real estate deed. 

In order to store one megabyte of data on Nervos, you would need to hold 1,048,576 CKBytes. This makes the storage of large data prohibitively expensive for large files. Notice that I said _hold_ CKBytes and not _pay_ CKBytes. This is because those CKBytes can be reclaimed once the data is removed from the state.

CKBytes are used to pay state rent while a Cell occupies blockchain state. A Cell must have at least enough capacity \(CKBytes\) for the space the Cell occupies in the blockchain state, including all data within it. The CKBytes are effectively locked in the Cell until the Cell is consumed. During this time, the CKBytes locked in a Cell are subject to targeted inflation. This inflation pays the state rent indirectly, requiring no action from the owner of the Cell.

When CKBytes are locked they are ineligible to use the NervosDAO, which pays users interest on their CKBytes. This interest is equal to the inflation which pays the state rent, effectively negating it, and making the CKByte a deflationary currency similar to Bitcoin.

For more information on this topic, you can read the [Crypto-Economic White Paper](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0015-ckb-cryptoeconomics/0015-ckb-cryptoeconomics.md). This is optional but will give much deeper insight into how the economics of Nervos work.





