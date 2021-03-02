# Introduction to Capsule

Capsule is a development framework for used to create on-chain scripts using the Rust and C programming languages. This includes both lock scripts and type scripts. Capsule provides the necessary tools to bootstrap, compile, test, debug, and deploy a new project. We will give a quickstart here to cover the basics.

![](../.gitbook/assets/capsule.jpg)

### Installation

Capsule can be installed on Linux, MacOS, and Windows via [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10). Docker and Rust will need to be installed before Capsule will function properly.

You should already have Rust installed since it was a prerequisite to this course. Installation instructions for Docker can be found on [their website](https://docs.docker.com/get-docker/). Docker must also be configured to [allow management by a non-root user](https://docs.docker.com/engine/install/linux-postinstall/).

After Rust and Docker are installed, you can type the following to install capsule:

```text
cargo install ckb-capsule
```

If you experience any problems, the full instructions can always be viewed on the [Capsule GitHub page](https://github.com/nervosnetwork/capsule).

### Creating Your First Capsule Project

From a terminal, type the following command to create a new project called `myproject` in the current directory:

```bash
capsule new myproject
```

This should display the output similar to the following.

```bash
New project "myproject"
Created file "capsule.toml"
Created file "deployment.toml"
Created file "README.md"
Created file "Cargo.toml"
Created file ".gitignore"
Initialized empty Git repository in /home/username/myproject/.git/
Created "/home/username/myproject"
Created tests
     Created library `tests` package
New contract "myproject"
     Created binary (application) `myproject` package
Rewrite Cargo.toml
Rewrite ckb_capsule.toml
Done
```

This initializes a capsule project called `myproject` and creates a single contract in the project called `myproject` within it. You can create as many contracts as you would like within a project, but we will start with just one.

Enter the project directory using `cd myproject`, then use the following command to build the project.

```text
capsule build
```

You should see output similar to this.

```bash
Building contract myproject
 Downloading crates ...
  Downloaded ckb-std v0.7.3
   Compiling cfg-if v0.1.10
   Compiling cc v1.0.41
   Compiling buddy-alloc v0.4.1
   Compiling molecule v0.6.0
   Compiling ckb-standalone-types v0.0.1-pre.1
   Compiling ckb-std v0.7.3
   Compiling myproject v0.1.0 (/code/contracts/myproject)
    Finished dev [unoptimized + debuginfo] target(s) in 6.13s
Done
```

This builds the current project binaries in debug mode. Now that our binary is built, we can test it using the following command.

```text
capsule test
```

This will execute all tests for the project. The first time this runs, it may take a while to compile because it's building all the necessary testing tools including a light-weight simulator that will be used run the compiled scripts in a simulated blockchain environment. The output should be similar to this.

```bash
    Finished test [unoptimized + debuginfo] target(s) in 0.38s
     Running target/debug/deps/tests-b429868b4af9df7d

running 2 tests
[contract debug] script args is Bytes([42])
[contract debug] script args is Bytes([])
test tests::test_empty_args ... ok
[contract debug] tx hash is [112, 109, 48, 116, 133, 71, 95, 186, 208, 254, 244, 53, 91, 80, 108, 213, 206, 112, 25, 187, 136, 180, 123, 124, 240, 31, 48, 192, 193, 175, 179, 176]
consume cycles: 322770
test tests::test_success ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests tests

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

The tests ran successfully! 🎉

### Project Structure

Next, we will look at how a project is structured. Open the `myproject/contracts/myproject/src` folder, which contains the source files for the `myproject` contract. You will see the following files.

* **entry.rs** - This file contains the logic of the smart contract. This is the main file we will work with.
* **error.rs** - This file contains error codes used in the project.
* **main.rs** - This file contains boilerplate code for the contract.



