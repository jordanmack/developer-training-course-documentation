# Lab: Use an SUDT in Lumos

Complete the exercise in `index.js` in the folder `Lab-SUDT-Exercise` by adding code and values as necessary.

The `index.js` file contains incomplete Lumos code to deploy, create, transfer, and consume cells using the ****SUDT type script. Your task is to add and modify the code as necessary to complete the transactions as instructed. 

1. Compile the `sudt` binary from the previous lab in release mode using `capsule build --release`, then copy it from `build/release/` to the `files/` folder. 
2. Update the `createCells()` function to create three cells using the `sudt` type script.
   1. All three of the token cells should have Alice as the owner.
   2. Each of the three tokens cells should have the following token amounts: 100, 300, 700
   3. Each of the tokens cells should contain only the minimum capacity necessary.
3. Update the `transferCells()` function to transfer the SUDT cells that were created.
   1. Send tokens to the following people: Bob: 200, Charlie: 500, Alice: 400 \(change\)
4. Update the `consumeCells()` function to burn Alice's remaining SUDT tokens.
   1. Alice no longer wants her remaining tokens and decides to burn them.
   2. The cell used for the token still contains capacity. These CKBytes must be recovered when the token cell is consumed. 

Run your code by opening a terminal to the `Lab-SUDT-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-SUDT-Solution` folder.

Once your code successfully executes, the resulting transaction ID will be printed on the screen.

