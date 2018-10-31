# Secure Function Defintion Language

This repository introduces EthSnarks support for SFDL (the Secure Function Definition Language, version 1) as defined by [Fairplay](Fairplay-whitepaper.pdf). SFDL programs are translated into SHDL (Secure Hardware Definition Language), which can then be translated directly into a format supported by EthSnarks.


## Example Program

```
/*
 * Compute AND of two byte 
 */
program And {
	const N=8;
	type Byte = Int<N>;
	type AliceInput = Byte;
	type BobInput = Byte;
	type AliceOutput = Byte; 
	type BobOutput = Byte; 
	type Input = struct {AliceInput alice,	BobInput bob};
	type Output = struct {AliceOutput alice, BobOutput bob};

	function Output output(Input input) {
           output.alice = (input.bob & input.alice);
           output.bob = (input.bob & input.alice);
	}
}
```

Which is compiled and optimised to produce a SHDL file:

```
0 input		//output$input.bob$0
1 input		//output$input.bob$1
2 input		//output$input.bob$2
3 input		//output$input.bob$3
4 input		//output$input.bob$4
5 input		//output$input.bob$5
6 input		//output$input.bob$6
7 input		//output$input.bob$7
8 input		//output$input.alice$0
9 input		//output$input.alice$1
10 input		//output$input.alice$2
11 input		//output$input.alice$3
12 input		//output$input.alice$4
13 input		//output$input.alice$5
14 input		//output$input.alice$6
15 input		//output$input.alice$7
16 output gate arity 2 table [ 0 0 0 1 ] inputs [ 8 0 ]	//output$output.alice$0
17 output gate arity 2 table [ 0 0 0 1 ] inputs [ 9 1 ]	//output$output.alice$1
18 output gate arity 2 table [ 0 0 0 1 ] inputs [ 10 2 ]	//output$output.alice$2
19 output gate arity 2 table [ 0 0 0 1 ] inputs [ 11 3 ]	//output$output.alice$3
20 output gate arity 2 table [ 0 0 0 1 ] inputs [ 12 4 ]	//output$output.alice$4
21 output gate arity 2 table [ 0 0 0 1 ] inputs [ 13 5 ]	//output$output.alice$5
22 output gate arity 2 table [ 0 0 0 1 ] inputs [ 14 6 ]	//output$output.alice$6
23 output gate arity 2 table [ 0 0 0 1 ] inputs [ 15 7 ]	//output$output.alice$7
24 output gate arity 1 table [ 0 1 ] inputs [ 16 ]	//output$output.bob$0
25 output gate arity 1 table [ 0 1 ] inputs [ 17 ]	//output$output.bob$1
26 output gate arity 1 table [ 0 1 ] inputs [ 18 ]	//output$output.bob$2
27 output gate arity 1 table [ 0 1 ] inputs [ 19 ]	//output$output.bob$3
28 output gate arity 1 table [ 0 1 ] inputs [ 20 ]	//output$output.bob$4
29 output gate arity 1 table [ 0 1 ] inputs [ 21 ]	//output$output.bob$5
30 output gate arity 1 table [ 0 1 ] inputs [ 22 ]	//output$output.bob$6
31 output gate arity 1 table [ 0 1 ] inputs [ 23 ]	//output$output.bob$7
```

In the context of zero-knowledge proofs all of the program inputs should be considered as 'private' with the outputs being public. That is to say, you provide a zero-knowledge proof that executing the program with an unknown set of inputs results in a publicly verifiable set of outputs.

The concepts of private input and public output distinguishes the use of SFDL in this context (zero-knowledge proofs for Ethereum) versus the multi-party computation it was originally designed for where Alice and Bob input their private parameters (which neither know about) and the outputs are visible to both.

## Optimised output packing

SFDL (and SHDL) deal with individual bits as the value for each wire, 'bitness constraints' must be enforced for every private input. The outputs of the program are considered the 'public inputs' for verifiction, these will be provided by the smart contract.

However, every input provided by the smart contract requires a G1 scalar multiply operation, these are very expensive. To reduce the on-chain cost all of the output bits will be packed into the smallest number of field elements as possible. In the example above there are 16 output bits, these can be packed into a single field element. This will be done automatically by the SHDL -> EthSnarks transpiler.
