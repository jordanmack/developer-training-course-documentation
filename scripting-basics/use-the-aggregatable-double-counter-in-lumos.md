# Use the Aggregatable Double Counter in Lumos

Complete the exercise in `index.js` in the folder `Lab-AggDoubleCounter-Exercise` by adding code and values as necessary.

The `index.js` file contains Lumos code to deploy, create, and consume cells using the **Double Counter type script**. Your task is to modify the functionality of this code to use the **Aggregatable Double Counter type script**.

1. Compile the `aggdoublecounter` binary from the previous lab in release mode using `capsule build --release`, then copy it from `build/release/` to the `files/` folder. 
2. Change the binary that is used from `doublecounter` to `aggdoublecounter` in `index.js`.
3. Update the `createCells()` function to create three cells using the `aggdoublecounter` type script.
   1. The cells should contain only the minimum capacity necessary to hold the cell data.
   2. The initial counter data values should be \[0, 0\], \[42, 42\], and \[9000, 9000\].
   3. The data field takes a hex string that must be prefixed with "0x". There should only be a single "0x" prefix even if there are two values included.
4. Update the `updateCells()` function to update the Aggregatable Double Counter cells that were created.
   1. The first counter value should be increased by 1, the second value should increase by 2.
   2. The data field of the input cell and output cell contains a single hex string for all values, even if there are multiple values contained within. Your code must properly encode and decode this value. 

Run your code by opening a terminal to the `Lab-AggDoubleCounter-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-AggDoubleCounter-Solution` folder.

Once your code successfully executes, the resulting transaction ID will be printed on the screen.

