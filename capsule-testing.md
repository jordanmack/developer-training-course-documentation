# Capsule Testing

### Testing

Capsule comes with a robust testing framework that allows you to run automated tests against a CKB node simulator. This tool is so powerful that many developers use it as the primary way to develop applications, using test-driven development \(TDD\).

Let's look at how tests are structured. Open the `myproject/tests/src` folder. You will see the following files.

* **lib.rs** - This file contains boilerplate code used for the test suite.
* **tests.rs** - This file contains all test cases for the project.

Feel free to open `lib.rs` and take a look at what it contains, but we're not going to go through it here. This file is all boilerplate code that is needed for testing. We won't need to touch anything in this file. 

Let's take a look at `tests.rs`. Starting at the top you see this.

```rust
use super::*;
use ckb_testtool::context::Context;
use ckb_tool::ckb_types::{
    bytes::Bytes,
    core::TransactionBuilder,
    packed::*,
    prelude::*,
};
use ckb_tool::ckb_error::assert_error_eq;
use ckb_tool::ckb_script::ScriptError;
```

These are the basic imports for the test suite.

```rust
const MAX_CYCLES: u64 = 10_000_000;
```

This is the maximum amount of cycles \(CPU resources\) that a script can use. If you're coming from Ethereum, you can think of this as a gas limit. This is a fairly high number, so it doesn't generally need to be changed.

```rust
// error numbers
const ERROR_EMPTY_ARGS: i8 = 5;
```

Here are the error numbers that are being tested for. These should match the values from the `error.rs` file. Remember, the `test.rs` file is for the entire project. It might contain multiple contracts \(scripts\). These errors should be named appropriately so they don't get confused.

```rust
#[test]
fn test_success() {
    // deploy contract
    let mut context = Context::default();
    let contract_bin: Bytes = Loader::default().load_binary("myproject");
    let out_point = context.deploy_cell(contract_bin);

    // prepare scripts
    let lock_script = context
        .build_script(&out_point, Bytes::from(vec![42]))
        .expect("script");
    let lock_script_dep = CellDep::new_builder()
        .out_point(out_point)
        .build();

    // prepare cells
    let input_out_point = context.create_cell(
        CellOutput::new_builder()
            .capacity(1000u64.pack())
            .lock(lock_script.clone())
            .build(),
        Bytes::new(),
    );
    let input = CellInput::new_builder()
        .previous_output(input_out_point)
        .build();
    let outputs = vec![
        CellOutput::new_builder()
            .capacity(500u64.pack())
            .lock(lock_script.clone())
            .build(),
        CellOutput::new_builder()
            .capacity(500u64.pack())
            .lock(lock_script)
            .build(),
    ];

    let outputs_data = vec![Bytes::new(); 2];

    // build transaction
    let tx = TransactionBuilder::default()
        .input(input)
        .outputs(outputs)
        .outputs_data(outputs_data.pack())
        .cell_dep(lock_script_dep)
        .build();
    let tx = context.complete_tx(tx);

    // run
    let cycles = context
        .verify_tx(&tx, MAX_CYCLES)
        .expect("pass verification");
    println!("consume cycles: {}", cycles);
}
```

This is the first test that will execute. There is a lot of code here. This is deploying the contract, building cells with scripts, forming a transaction, then executing the transaction in a virtual environment that simulates a blockchain. As the `test_success()` function name indicates, this is testing for an expected success scenario.

Line 5 loads the `myproject` contract binary that was built in the `myproject/build/debug` directory. You must always compile your script using `capsule build` before it is available for testing. Line 6 deploys the code and gets the out point.

Line 9 to 11 build a lock script from the deployed `myproject` binary. On line 10, the `args` are being specified as the second parameter in the `build_script()` method. It's being set to a single byte with the value `42`. Lines 12 to 14 deploy the lock script.

Lines 17 to 26 create an input cell. The script has a capacity of 1,000 Shannons and uses the lock script we just deployed. This is far below the requirements of a real cell, but in this simulator, the rules are relaxed.

Lines 27 to 36 create two output cells. Both cells have 500 Shannons of capacity and use the same lock script. Line 38 defines the data that is in our cells. Both cells have no data.

Lines 41 to 47 populate the transaction and build it. Our lock script doesn't require a signature, so none are included here. If we did need one, we would populate it using `.witnesses()`.

Lines 50 to 53 execute the transaction, expecting that it will pass. The cycles are printed to the screen showing how many were consumed during execution.

```rust
#[test]
fn test_empty_args() {
    // deploy contract
    let mut context = Context::default();
    let contract_bin: Bytes = Loader::default().load_binary("myproject");
    let out_point = context.deploy_cell(contract_bin);

    // prepare scripts
    let lock_script = context
        .build_script(&out_point, Default::default())
        .expect("script");
    let lock_script_dep = CellDep::new_builder()
        .out_point(out_point)
        .build();

    // prepare cells
    let input_out_point = context.create_cell(
        CellOutput::new_builder()
            .capacity(1000u64.pack())
            .lock(lock_script.clone())
            .build(),
        Bytes::new(),
    );
    let input = CellInput::new_builder()
        .previous_output(input_out_point)
        .build();
    let outputs = vec![
        CellOutput::new_builder()
            .capacity(500u64.pack())
            .lock(lock_script.clone())
            .build(),
        CellOutput::new_builder()
            .capacity(500u64.pack())
            .lock(lock_script)
            .build(),
    ];

    let outputs_data = vec![Bytes::new(); 2];

    // build transaction
    let tx = TransactionBuilder::default()
        .input(input)
        .outputs(outputs)
        .outputs_data(outputs_data.pack())
        .cell_dep(lock_script_dep)
        .build();
    let tx = context.complete_tx(tx);

    // run
    let err = context
        .verify_tx(&tx, MAX_CYCLES)
        .unwrap_err();
    // we expect an error raised from 0-indexed cell's lock script
    let script_cell_index = 0;
    assert_error_eq!(
        err,
        ScriptError::ValidationFailure(ERROR_EMPTY_ARGS).input_lock_script(script_cell_index)
    );
}
```

This is the second test that will execute. As the `test_empty_args()` function name indicates, this is testing for an expected failure scenario due to there not being any args supplied to the script. A lot of this code is very similar to the previous code block, so we will only look at the most important lines.

Line 10 creates our lock script. The second parameter is the `args` to the script, and it is empty this time. 

Lines 50 to 58 execute the transaction. This is expected to fail, and line 57 specifies `ERROR_EMPTY_ARGS` as the expected error code. As long as the script returns this specific error code, this test will validate successfully.

