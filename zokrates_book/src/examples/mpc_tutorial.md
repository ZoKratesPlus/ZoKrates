# Tutorial: Performing a trusted setup using the multi-party contribution (MPC) protocol

The zk-SNARK technology requires a trusted setup which is a special procedure we can run to generate the proving and verification keys.
In order to make sure this procedure is done in a secure way, we must ensure that no one is able to fake proofs and steal user funds, so it has to be done
in a decentralized way. In order to fake ZK proofs, an attacker must compromise every participant of the ceremony which is highly unlikely as the probability of it goes down as the number of participants goes up.
In this tutorial, we will walk you through the steps of the ceremony.

## Pre-requisites

Trusted setup is done in two steps. The first step, also known as phase 1, is universal for all SNARKS and is called Powers of Tau. The second step is called phase 2 and is circuit-specific, so it should
be done separately for each different SNARK. There is an existing phase 1 ceremony being conducted by the Ethereum community named [Perpetual Powers of Tau](https://github.com/weijiekoh/perpetualpowersoftau), which output we can use in our phase 2 ceremony.

## Compiling a circuit

We will start this tutorial by using ZoKrates to compile a basic circuit.
First, we create a new file named `circuit.zok` with the following content:

```zokrates
{{#include ../../../zokrates_cli/examples/book/mpc_tutorial/circuit.zok}}
```

We compile the program into an arithmetic circuit using the `compile` command.

```
{{#include ../../../zokrates_cli/examples/book/mpc_tutorial/test.sh:11}}
```

## Initializing a phase 2 ceremony

As a next step we initialize a phase 2 ceremony by running the following command:

```
$ {{#include ../../../zokrates_cli/examples/book/mpc_tutorial/test.sh:15}}

Initializing MPC...
Writing initial parameters to `mpc.params`
```

Using the `-r` flag we pass a path to the directory that contains parameters for various `2^m` circuit depths (`phase1radix2m{0..=m}`).
These files can be computed from the phase 1 ceremony or downloaded from [here](https://example.com).

## Making a contribution

In this example, we will conduct a ceremony that has 3 participants: Alice, Bob, and Charlie.
Participants will run the contributions in sequential order, managed by the coordinator (us).

Firstly, our initial `mpc.params` file is given to Alice who runs the following command:

```
$ {{#include ../../../zokrates_cli/examples/book/mpc_tutorial/test.sh:18}}

Contributing to `mpc.params`...
Contribution hash: 4ebf1359416fbc4231af64769173cb3332b8c2f9475d143a25634a5ce461eb675f7738b16478a0207ec9d3659170bca6154b31dfd307b78eca0c025f59c5a7fb
Your contribution has been written to `alice.params`
```

Alice must give some randomness to the contribution, which is done by the `-e` flag.

Examples of entropy sources:
* `/dev/urandom` from one or more devices,
* The most recent block hash,
* Randomly mashing keys on the keyboard etc.

Secondly, the output file `alice.params` is sent to Bob (managed by the coordinator) who runs his contribution:

```
$ {{#include ../../../zokrates_cli/examples/book/mpc_tutorial/test.sh:21}}

Contributing to `alice.params`...
Contribution hash: 1a4e0d17449b00ecf31d207259bc173cf30f6dbd78c81921869091a8e40f454e8c8d72e8395bf044cd777842b6ab1d889e24cf7f7d88b4732190fb0c730fb6fc
Your contribution has been written to `bob.params`
```

Thirdly, with the same procedure as above, Charlie makes his contribution on top of Bob's:

```
$ {{#include ../../../zokrates_cli/examples/book/mpc_tutorial/test.sh:24}}

Contributing to `bob.params`...
Contribution hash: 46dc6c01ec77838293b333b2116a4bfba9aca5ddeb6945f1cbe07cda6ffb3ffdcf4e473662fe2339166d5b87db392ca6d2e87e3692cc8f0ee618298fc3f7caf1
Your contribution has been written to `charlie.params`
```

## Applying a random beacon

To finalize the ceremony, we can apply a random beacon to get the final parameters:

```
$ {{#include ../../../zokrates_cli/examples/book/mpc_tutorial/test.sh:27}}

Creating a beacon RNG
0: b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9
1: bc62d4b80d9e36da29c16c5d4d9f11731f36052c72401a76c23c0fb5a9b74423
2: 76dfcb21a877aaeba06b3269d08dc2ed1d38c62ffec132800b46e94b14f72938
...removed for brevity
1022: dd842dc43d9ac5c6dff74cca18405123761d17edd36724b092ef57c237b31291
1023: a11c8a03c22e9c31c037aa0085c061ba8dd19a3f599314570702eeef1baacd79
Final result of beacon: ef8faec4fc31faf341f368084b82d267d380992e905c923a179e0717ce39708d
Contributing to `charlie.params`...
Contribution hash: 83d67a6f935fc4d05733ebedd43f20745425b1059a32a315a790668af5a1f02166f840e2e6a5d441385931635b86df09a00f352e2ad2a88bede078862134b889
Writing parameters to `final.params`
```

The random beacon is the `2^n` iteration of `SHA256` over the hash evaluated on
some high entropy and publicly available data. Possible sources of data could be: the
closing value of the stock market on a certain date, the output of a selected set of national lotteries, the
value of a block at a particular height in one or more blockchains, etc.

## Verifying contributions

At any point in the ceremony we can verify contributions by running the following command:

```
$ {{#include ../../../zokrates_cli/examples/book/mpc_tutorial/test.sh:30}}

Verifying contributions...

Contributions:
0: 4ebf1359416fbc4231af64769173cb3332b8c2f9475d143a25634a5ce461eb675f7738b16478a0207ec9d3659170bca6154b31dfd307b78eca0c025f59c5a7fb
1: 1a4e0d17449b00ecf31d207259bc173cf30f6dbd78c81921869091a8e40f454e8c8d72e8395bf044cd777842b6ab1d889e24cf7f7d88b4732190fb0c730fb6fc
2: 46dc6c01ec77838293b333b2116a4bfba9aca5ddeb6945f1cbe07cda6ffb3ffdcf4e473662fe2339166d5b87db392ca6d2e87e3692cc8f0ee618298fc3f7caf1
3: 83d67a6f935fc4d05733ebedd43f20745425b1059a32a315a790668af5a1f02166f840e2e6a5d441385931635b86df09a00f352e2ad2a88bede078862134b889

Contributions verified
```

## Exporting keys

Once the ceremony is finalized, we can export the keys and use them to generate proofs and verify them.

```
{{#include ../../../zokrates_cli/examples/book/mpc_tutorial/test.sh:32:38}}
```

## Conclusion

The purpose of the ceremony is to generate a verifier smart contract, which can be exported using ZoKrates by using the keys we obtained through the trusted setup ceremony. At this point, we can safely deploy the contract and verify proofs on-chain.