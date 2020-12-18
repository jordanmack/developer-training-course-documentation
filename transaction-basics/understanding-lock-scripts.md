# Understanding Lock Scripts

Lock Scripts are one of the many powerful features that differentiate Nervos from most other blockchain platforms. A Lock Script is a small program that is used to define ownership of a Cell. This program has the power to examine the transaction it is included within. This gives the developer a tremendous amount of flexibility on how to manage access.

The default Lock Script is based on Secp256k1 cryptography, making it nearly identical to Bitcoin and Ethereum. This allows a Cell to be owned and unlocked by any user who possesses the private key. However, a Lock Script can do much more. A Cell can be owned by multiple people, or by a smart contract. A Cell can also be owned by no one, but unlock when specific conditions are met in a transaction.



