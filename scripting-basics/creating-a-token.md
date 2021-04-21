# Creating a Token

Tokens are one of the most common use cases for smart contracts today. As of 2021, there are over 400,000 different tokens that have been created on various platforms. In this lesson, we will learn how to create a basic token on Nervos.

To do this we will examine the SUDT standard, which stands for Simple User-Defined Token. SUDT is a minimalistic token standard, and the first official token standard to be released on Nervos. We use a Rust-based implementation that has specifically been formatted to be easier to read. The [official implementation](https://github.com/nervosnetwork/ckb-miscellaneous-scripts/blob/master/c/simple_udt.c) is written in C, but reviewing it is purely optional.

### Script Logic

Next, we will look at a flow chart of the logic that that is used in the SUDT type script.

![](../.gitbook/assets/script-logic-flow.png)

After script execution begins, the first step is to determine if it is running in owner mode. This is a form of operation detection which is used to check if the transaction is being executed by the user who created this token. If it is the owner of the token, then owner mode is enabled, and the script immediately exits successfully. This is the equivalent of a super-user mode, where all actions are permitted. If owner mode is not enabled, then validation continues.

The next step is to determine the total number of input tokens and the total number of output tokens. This is the total number of tokens in all of the inputs, and all of the outputs. To do this it must read the data of every token cell in the transaction, and check if the cell data is valid and can be converted into a number so we can tally a sum. If any cell is found to have invalid data, the script fails immediately. If all cells contain valid data, then the token totals are counted and the validation continues.

The next step is to check if the input tokens are greater than or equal to the output tokens. This single logical statement is the basis for enforcing the scarcity of the token. If the statement is not true, then there are more output tokens than there are input tokens. A user other than the owner may be trying to illegally mint tokens out of nothing. This would cause the script to immediately fail. If the statement is true, then the scarcity of the token is being preserved properly, and the script would exit successfully.

### Implementation in Rust

Next, we will review the code of the Rust implementation of SUDT. We will look at one section at a time to make it easier to understand, but you can review the full source code at any time by opening the `entry.rs` file in the directory`developer-training-course-script-examples/contracts/sudt/src`.



