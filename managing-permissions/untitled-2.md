# Using the Multi-Sig Lock Script

Multi-sig functionality allows a single cell to be owned and unlocked by multiple users. Just like with the Secp256r1 example, any cell can use a multi-sig lock by specifying a multi-sig binary executable in the lock script and providing the correct arguments.

Open the `index.js` file from the `Using-the-Multi-Sig-Lock-Script-Example` directory. Scroll down to the bottom and find the `main()` function. There are three main sections:

1. Initialize - These are the first three lines of code in `main()`. We initialize the Lumos configuration, start the Lumos Indexer, and initialize the lab environment.
2. Deploy Code - The `deployAlwaysSuccessBinary()` function creates a cell with the contents of the RISC-V binary located in the file `./files/always_success`. This is the always success lock, an on-chain script that always grants permission to the cell in any transaction.
3. Create Cell - The `createCellWithAlwaysSuccessLock()` function creates a cell that uses the always success lock.
4. Consume Cell - The `consumeCellWithAlwaysSuccessLock()` function consumes the cell with the always success lock that we just created.





