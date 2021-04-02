# Lab: Convert IC3Type to OC5Type

Complete the exercise in `entry.rs` in the folder `Lab-OC5Type-Exercise/contracts/oc5type/src` by adding code and values as necessary.

The name "IC3Type" stands for "input count 3". This type script counts the number of input cells available and only succeeds when that count is 3.

The name "OC5Type" stands for "output count 5". This type script counts the number of **output** cells available and only succeeds when that count is **5**.

The `entry.rs` file contains the source code for the IC3Type script. Your task is to modify the functionality of this script to match that of OC5Type.

Build your code by opening a terminal to the `Lab-OC5Type-Exercise` folder and running `capsule build`, then test your code using `capsule test` after the build is successful. If you get stuck you can find the solution in the `Lab-OC5Type-Solution` folder.

Note: Be sure to always build your code after modifying it before it is tested again so that changes are properly reflected.

Once your code successfully compiles and all tests are passed, you have completed this lab. The test output will contain the following to indicate a successful test.

```bash
running 3 tests
test tests::test_oc5type_invalid_too_few_cells ... ok
test tests::test_oc5type_invalid_too_many_cells ... ok
test tests::test_oc5type_valid ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```



