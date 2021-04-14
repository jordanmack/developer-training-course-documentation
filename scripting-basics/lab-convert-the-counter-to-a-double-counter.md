# Lab: Convert the Counter to a Double Counter

Complete the exercise in `entry.rs` in the folder `Lab-DoubleCounter-Exercise/contracts/doublecounter/src` by adding code and values as necessary.

The Counter type script ensures that the u64 value contained in the data area of the cell is incremented by exactly one each time the cell is transferred or updated.

The DoubleCounter type script ensures that the two u64 values contained in the data area of the cell are incremented each time the cell is transferred or updated. The first value must be incremented by exactly one, and the second value must be incremented by exactly 2.

The `entry.rs` file contains the source code for the Counter type script. Your task is to modify the functionality of this script to match that of DoubleCounter.

1. The DoubleCounter takes no script args.
2. Both counter values stored in the cell's data area should be u64 LE values \(8 bytes each\).
3. You will not need to modify `error.rs`. This already contains the errors required for the DoubleCounter type script.

Build your code by opening a terminal to the `Lab-DoubleCounter-Exercise` folder and running `capsule build`, then test your code using `capsule test` after the build is successful. If you get stuck you can find the solution in the `Lab-DoubleCounter-Solution` folder.

Note: Be sure to always build your code after modifying it before it is tested again so that changes are properly reflected.

Once your code successfully compiles and all tests are passed, you have completed this lab. The test output will contain the following to indicate a successful test.

```text
running 15 tests
test tests::test_doublecounter_burn_multiple ... ok
test tests::test_doublecounter_burn ... ok
test tests::test_doublecounter_create ... ok
test tests::test_doublecounter_create_invalid_output_data ... ok
test tests::test_doublecounter_create_invalid_output_data_value ... ok
test tests::test_doublecounter_create_no_output_data ... ok
test tests::test_doublecounter_invalid_transfer_minus_1_value_1 ... ok
test tests::test_doublecounter_invalid_transfer_plus_1_value_2 ... ok
test tests::test_doublecounter_invalid_transfer_plus_9000_value_1 ... ok
test tests::test_doublecounter_invalid_transfer_plus_2_value_1 ... ok
test tests::test_doublecounter_transfer_high_value ... ok
test tests::test_doublecounter_transfer ... ok
[contract debug] panic occurred: range end index 16 out of range for slice of length 8, in file contracts/doublecounter/src/entry.rs:37
test tests::test_doublecounter_transfer_invalid_input_data_panic_expected ... ok
[contract debug] panic occurred: range end index 16 out of range for slice of length 8, in file contracts/doublecounter/src/entry.rs:46
test tests::test_doublecounter_transfer_invalid_output_data_panic_expected ... ok
[contract debug] panic occurred: attempt to add with overflow, in file contracts/doublecounter/src/entry.rs:50
test tests::test_doublecounter_transfer_overflow_panic_expected ... ok

test result: ok. 15 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Note: Our tests expect certain output values and also test error cases. If your script contains different logic for error handling, it may be reported as a failed test. If you have included better error handling than was expected by our tests, disregard the error. Good job!





