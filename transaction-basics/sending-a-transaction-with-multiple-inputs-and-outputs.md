# Sending a Transaction with Multiple Inputs and Outputs

### Lesson Introduction

In this lesson, we will introduce Nervos' Cell Model, and use it to generate a transaction using code. You will need the outpoints you verified from the last lab exercise, so make sure you have them handy.

### The Cell Model

In the last lab exercise, you may have noticed that the command you used to verify that your outputs were unspent is called `get_live_cell`. It's called this because, in Nervos' terminology, both inputs and outputs are canonically referred to as "cells". An 

