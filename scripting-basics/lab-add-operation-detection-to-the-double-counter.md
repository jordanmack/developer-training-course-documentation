# Lab: Add Operation Detection to the Double Counter

Complete the exercise in `entry.rs` in the folder `Lab-ODDoubleCounter-Exercise/contracts/oddoublecounter/src` by adding code and values as necessary.

The DoubleCounter type script ensures that the two u64 values contained in the data area of the cell are incremented each time the cell is transferred or updated. The first value must be incremented by exactly one, and the second value must be incremented by exactly 2.

The ODDoubleCounter type script is the same as the DoubleCounter type script, but it uses operation detection that includes the ability to burn.

The `entry.rs` file contains the source code for the DoubleCounter type script. Your task is to modify the functionality of this script to match that of ODDoubleCounter.

1. The ODDoubleCounter takes no script args and is not aggregatable.
2. Both counter values stored in the cell's data area should be u64 LE values \(8 bytes each\).
3. You will not need to modify `error.rs`. This already contains the errors required for the ODDoubleCounter type script.
4. If you need more examples of operation detection, feel free to review the code in the `odcounter` example.

Build your code by opening a terminal to the `Lab-ODDoubleCounter-Exercise` folder and running `capsule build`, then test your code using `capsule test` after the build is successful. If you get stuck you can find the solution in the `Lab-ODDoubleCounter-Solution` folder.

Note: Be sure to always build your code after modifying it before it is tested again so that changes are properly reflected.

Once your code successfully compiles and all tests are passed, you have completed this lab. The test output will contain the following to indicate a successful test.

```text
running 16 tests
test tests::test_oddoublecounter_burn ... ok
test tests::test_oddoublecounter_burn_multiple ... ok
test tests::test_oddoublecounter_create ... ok
test tests::test_oddoublecounter_create_invalid_output_data_value ... ok
test tests::test_oddoublecounter_create_no_output_data ... ok
test tests::test_oddoublecounter_create_multiple ... ok
test tests::test_oddoublecounter_transfer ... ok
test tests::test_oddoublecounter_transfer_high_value ... ok
test tests::test_oddoublecounter_transfer_invalid_input_data ... ok
test tests::test_oddoublecounter_transfer_invalid_output_data ... ok
test tests::test_oddoublecounter_transfer_multiple ... ok
test tests::test_oddoublecounter_transfer_multiple_mismatch_cell_count_too_few_outputs ... ok
test tests::test_oddoublecounter_transfer_overflow ... ok
test tests::test_oddoublecounter_transfer_multiple_mismatch_cell_count_too_many_outputs ... ok
test tests::test_oddoublecounter_transfer_value_1_plus_2 ... ok
test tests::test_oddoublecounter_transfer_value_2_plus_9000 ... ok

test result: ok. 16 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Note: Our tests expect certain output values and also test error cases. If your script contains different logic for error handling, it may be reported as a failed test. If you have included better error handling than was expected by our tests, disregard the error. Good job!





