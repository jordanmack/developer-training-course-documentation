# Lab: Convert DataCap to DataRange

Complete the exercise in `entry.rs` in the folder `Lab-DataRange-Exercise/contracts/datarange/src` by adding code and values as necessary.

The DataCap type script ensures that the data contained in the cell has a size that is equal to or less than the value specified in the script args.

The DataRange type script ensures that the data contained in the cell has a size that falls within a range specified by minimum and maximum values which are specified in the script args.

The `entry.rs` file contains the source code for the DataCap type script. Your task is to modify the functionality of this script to match that of DataRange.

1. The script args of the DataRange script takes two values, the minimum data size, and maximum data size.
2. The data size values specify the limits in bytes.
3. Both the minimum and maximum data size values are u32 LE values \(4 bytes each\).

Build your code by opening a terminal to the `Lab-DataRange-Exercise` folder and running `capsule build`, then test your code using `capsule test` after the build is successful. If you get stuck you can find the solution in the `Lab-DataRange-Solution` folder.

Note: Be sure to always build your code after modifying it before it is tested again so that changes are properly reflected.

Once your code successfully compiles and all tests are passed, you have completed this lab. The test output will contain the following to indicate a successful test.

```bash
running 5 tests
test tests::test_datarange_data_limit_exceeded ... ok
test tests::test_datarange_burn ... ok
test tests::test_datarange_empty_args ... ok
test tests::test_datarange_empty_data ... ok
test tests::test_datarange_valid_data ... ok

test result: ok. 5 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```





