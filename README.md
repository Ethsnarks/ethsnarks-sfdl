# Secure Function Defintion Language

This repository introduces EthSnarks support for SFDL (the Secure Function Definition Language, version 1) as defined by [Fairplay](Fairplay-whitepaper.pdf). SFDL programs are translated into SHDL (Secure Hardware Definition Language), which can then be translated directly into a format supported by EthSnarks.

[![Build Status](https://travis-ci.org/HarryR/ethsnarks-sfdl.svg?branch=master)](https://travis-ci.org/HarryR/ethsnarks-sfdl)

## Example Program

```
/*
 * Check which of two Millionaires is richer
 */
program Millionaires {
    type int = Int<3>;
	type Output = struct {Boolean alice,
            Boolean bob};
	type Input = struct {int alice,
            int bob};

	function Output output(Input input) {
            output.alice = (input.alice > input.bob);
            output.bob = (input.bob > input.alice);
	}
}

```

Which is compiled and optimised to produce a SHDL file:

```
nizkinput 0		#output$input.bob$0
nizkinput 1		#output$input.bob$1
nizkinput 2		#output$input.bob$2
nizkinput 3		#output$input.alice$0
nizkinput 4		#output$input.alice$1
nizkinput 5		#output$input.alice$2
table 2 [ 1 0 0 0 ] in 2 < 3 4 > out 1 < 6 >
table 2 [ 0 1 1 0 ] in 2 < 3 4 > out 1 < 7 >
table 2 [ 1 0 0 1 ] in 2 < 6 5 > out 1 < 8 >
table 2 [ 0 0 0 1 ] in 2 < 3 0 > out 1 < 9 >
table 3 [ 0 0 0 1 0 1 1 1 ] in 3 < 9 7 1 > out 1 < 10 >
table 2 [ 0 1 1 0 ] in 2 < 8 2 > out 1 < 11 >
table 2 [ 0 1 1 0 ] in 2 < 10 11 > out 1 < 12 >
output 13
table 1 [ 0 1 ] in 1 < 12 > out 1 < 13 >	#output$output.alice$0
table 2 [ 1 0 0 0 ] in 2 < 0 1 > out 1 < 14 >
table 2 [ 0 1 1 0 ] in 2 < 0 1 > out 1 < 15 >
table 2 [ 1 0 0 1 ] in 2 < 14 2 > out 1 < 16 >
table 2 [ 0 0 0 1 ] in 2 < 0 3 > out 1 < 17 >
table 3 [ 0 0 0 1 0 1 1 1 ] in 3 < 17 15 4 > out 1 < 18 >
table 2 [ 0 1 1 0 ] in 2 < 16 5 > out 1 < 19 >
table 2 [ 0 1 1 0 ] in 2 < 18 19 > out 1 < 20 >
output 21
table 1 [ 0 1 ] in 1 < 20 > out 1 < 21 >	#output$output.bob$0
```

In the context of zero-knowledge proofs all of the program inputs should be considered as 'private' with the outputs being public. That is to say, you provide a zero-knowledge proof that executing the program with an unknown set of inputs results in a publicly verifiable set of outputs.

The concepts of private input and public output distinguishes the use of SFDL in this context (zero-knowledge proofs for Ethereum) versus the multi-party computation it was originally designed for where Alice and Bob input their private parameters (which neither know about) and the outputs are visible to both.

## Optimised output packing

SFDL (and SHDL) deal with individual bits as the value for each wire, 'bitness constraints' must be enforced for every private input. The outputs of the program are considered the 'public inputs' for verifiction, these will be provided by the smart contract.

However, every input provided by the smart contract requires a G1 scalar multiply operation, these are very expensive. To reduce the on-chain cost all of the output bits will be packed into the smallest number of field elements as possible. In the example above there are 16 output bits, these can be packed into a single field element.
