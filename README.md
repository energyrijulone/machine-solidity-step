# Cartesi RISC-V Solidity Emulator

The Cartesi RISC-V Solidity Emulator is the on-chain host implementation of the Cartesi Machine Specification. The libraries and contracts are written in Solidity, and the testing scripts are written in Solidity (with the help of [Foundry](https://github.com/foundry-rs/foundry)). **_Do not recommend running `forge` command directly, as there are many scripts and templates working together. Please use `make` instead._**

For Cartesi's design to work, this implementation must have the exact transition function as the off-chain [Cartesi RISC-V Emulator](https://github.com/cartesi/machine-emulator), meaning that if given the same initial state (s[i]) both implementation's step functions should reach a bit by bit consistent state s[i + 1].

Since the cost of storing a full Cartesi Machine state within the blockchain is prohibitive, all machine states are represented in the blockchain as cryptographic hashes. The contents of those states and memory represented by those hashes are only known off-chain.

Cartesi uses Merkle tree operations and properties to ensure that the blockchain has the ability to correctly verify a state transition without having full state-access. However, the RISC-V Solidity emulator abstracts these operations away and acts as if it knows the full contents of a machine state - it uses the Memory Manager interface to fetch or write any necessary words to memory.

## AccessLogs

The `AccessLogs.Context` struct is consumed by the RISC-V Solidity emulator as if the entire state content was available - since the off and on-chain emulators match down to the order in which accesses are logged. When a dispute arises, Alice packs her off-chain state access log referent to the disagreement step in an struct `AccessLogs.Context`, which will guide the execution of a `step` (i.e state transition function).

The `AccessLogs` library implements the RISC-V Solidity emulator all necessary read and write operations.

It also makes sure that all accesses performed by the `step` function match the ones provided by Alice and are consistent with the Merkle proofs provided by her. If that is not the case, Alice loses the dispute.

## Step function

`step` is the previously mentioned state transition function, it is meant to take the machine from state s[i] to state[i + 1], using the `AccessLogs` as an assistant. The `step` function receives `IUArchState.State` - which should have been populated with the `AccessLogs` generated by the emulator off-chain and returns an Exit code signaling the result of its execution.

During a `step` execution, every necessary read or write (be it to memory, registers etc) is processed and verified by the `AccessLogs`.

## Execute Instruction

The `UArchExecuteInsn` contract consists of the machine instruction logic, such as decoding, executing, opcode matching and etc. The Solidity implementation is converted from the Cpp implementation directly through a translator script. This is to assure that the implementations in two languages are identical. Yet the low level differences in two languages are wrapped in the Compatibility Layer.

## Compatibility Layer

The `UArchCompat` contract is taking care of all the differences in the two implementations. Ranging from programming languages (Cpp versus Solidity) to architectural differences (RISC-V versus EVM).

## Getting Started

Run `make help` for a list of target options. Here are some of them:

```
Cleaning targets:
  clean                      - clean the cache artifacts and generated files
Generic targets:
* all                        - build solidity code. To build from a clean clone, run: make submodules all
  build                      - build solidity code
  generate-all               - generate all solidity code
  generate-step              - generate solidity-step code from cpp
  generate-mock              - generate mock library code
  generate-prod              - generate production library code
  generate-replay            - generate replay tests
  pretest                    - download necessary files for tests
  test-all                   - test all
  test-mock                  - test binary files with mock library
  test-prod                  - test production code
  test-replay                - test log files
```

### Prerequisite

-   Build [Cartesi docker image](https://github.com/cartesi/machine-emulator#getting-started)

### Requirements

-   Foundry 0.2.0
-   GNU Make >= 3.81
-   GPP >= 2.27

Different version of tools maybe working but is not guaranteed.

### Install

Install dependencies and build:

    make submodules all

### Run tests

There are two types of tests that can be run on the Solidity Emulator.

1. Load binary rv64i test programs in Solidity and verify the execution result
2. Collect step logs and proofs of a test program and replay them in Solidity, verify all accesses and proofs.

Run all tests:

    make test-all

(**_target test-replay is very time consuming_**)

## Contributing

Thank you for your interest in Cartesi! Head over to our [Contributing Guidelines](CONTRIBUTING.md) for instructions on how to sign our Contributors Agreement and get started with Cartesi!

Please note we have a [Code of Conduct](CODE_OF_CONDUCT.md), please follow it in all your interactions with the project.

## License

The machine-solidity-step repository and all contributions are licensed under
[APACHE 2.0](https://www.apache.org/licenses/LICENSE-2.0). Please review our [LICENSE](LICENSE) file.
