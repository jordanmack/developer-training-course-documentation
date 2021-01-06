# Lab Exercise Setup

### Setup Introduction

Our examples are all created using a Linux environment, but it should also work for MacOS and Windows.

### Install Node.js

Our lessons and lab exercises all rely on Node.js, so it will need to be installed prior to starting.

Installing vanilla Node.js is fine, or you can use a tool like NVM to manage the installation.

* Vanilla Node.js \(All Platforms\): [https://nodejs.org/en/download/](https://nodejs.org/en/download/)
* NVM \(Linux\): [https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)
* NVM \(Windows\): [https://github.com/coreybutler/nvm-windows](https://github.com/coreybutler/nvm-windows) 

### Install Rust

We will be using the Rust programming language to create on-chain scripts and install the required tooling. Using `rustup` is generally recommended, but there are several methods available.

* Rust \(All Platforms\): [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install)

### Install Git

You will need Git to clone the example code and lab exercises from GitHub in order to complete each lesson.

* Git \(All Platforms\): [https://git-scm.com/downloads](https://git-scm.com/downloads)

### Clone the Developer Training Course Materials

Use the command below to clone the Developer Training Course materials, which includes the example code and lab exercises we will use in the lessons.

```text
git clone https://github.com/jordanmack/developer-training-course.git
```

### Setup a CKB Dev Blockchain

You will need to have a CKB Dev Blockchain node running locally for our code to interact with. This is a full Nervos CKB node that will run on your computer with a private testnet.

You will need to complete the setup instruction from the URL below for the sections "Setup a Dummy-Worker Blockchain" and "Adding the Genesis Issued Cells".

* CKB Dev Blockchain Setup Instructions: [https://docs.nervos.org/docs/basics/guides/devchain](https://docs.nervos.org/docs/basics/guides/devchain)

### ~~Setup a CKB Indexer~~

~~You will need to setup a CKB Indexer. This tool works along-side the CKB Dev Blockchain to help speed up processing of transactions.~~

~~~~[~~https://github.com/nervosnetwork/ckb-indexer~~](https://github.com/nervosnetwork/ckb-indexer)~~~~

