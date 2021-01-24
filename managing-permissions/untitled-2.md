# Using the Multi-Sig Lock Script

Multi-sig functionality allows a single cell to be owned and unlocked by multiple users. Just like with the Secp256r1 example, any cell can use a multi-sig lock by specifying a multi-sig binary executable in the lock script and providing the correct arguments.

Open the `index.js` file from the `Using-the-Multi-Sig-Lock-Script-Example` directory. Scroll down to the bottom and find the `main()` function. There are three main sections:

1. Initialize - In the first three lines of code in `main()`, we initialize the Lumos configuration, start the Lumos Indexer, and initialize the lab environment.
2. Create Cell - The `createMultisigCell()` function creates a cell that uses the multi-sig lock.
3. Consume Cell - The `consumeMultisigCell()` function consumes the cell with the multi-sig lock that we just created.





