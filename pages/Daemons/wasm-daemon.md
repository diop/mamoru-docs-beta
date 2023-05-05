# WASM Daemon Development

WASM Daemons allow you to define incident matching logic using any language that can be compiled to WebAssembly.
Currently, only [AssemblyScript](https://www.assemblyscript.org/) is supported.
You can write a custom code against a blockchain context, decide when to emit an incident, customize message, severity, etc.

1. [Prerequisite](#prerequisite)
    1. [Setup environment](#setup-environment)
    2. [Define contract for monitoring](#define-contract-for-monitoring)
2. [Creating a Daemon](#creating-a-daemon)
3. [Testing](#testing)
4. [Deploying](#deploying)

## Prerequisite

### Setup environment

#### Install `validatoin-chaind`

You need the validation-chaind CLI to interact with the Mamoru Validation Chain.
It is available in the `mamorufoundation/validation-chain:latest` docker image.

Usage:

```shell
docker run -it --rm mamorufoundation/validation-chain:latest {subcommand}
```

Consider creating a command alias for convenience:

```shell
alias validation-chaind="docker run -it --rm -v $HOME/.validation-chain:/root/.validation-chain mamorufoundation/validation-chain:latest"
```

#### Create and fund an account

Execute the following command:

```shell
validation-chaind keys add wasm-daemon-guide
```

This commands adds an account named `wasm-daemon-guide` to your keychain.
Take the newly created account address (look like `cosmos1qaar0r0tyyy6ufslam0hv9zvhvy0z0cv5x8xev`) and fund it with some tokens.

// TODO: make public faucet for a testnet

### Define contract for monitoring

In this guide, we will work with Gnosis Safe contract in Ethereum Goerli network.
The contract source code can be found [here](https://github.com/safe-global/safe-contracts/tree/main/contracts).

## Creating a Daemon

### Init AssemblyScript project

First, we need to init the project template.
Create a new directory, `cd` into it and run the following commands:
```shell
npm init
npm install --save-dev assemblyscript@0.26.3
npx asinit .
```

### Write the daemon code

Let's emit an incident if someone withdraws money from the Safe.
The function we need is called `execTransaction`, the code is located [here](https://github.com/safe-global/safe-contracts/blob/96a4e280876c33c53a09b5ef6ee78201a101ff58/contracts/Safe.sol#L119-L146).

As we can see from the code, the function emits `ExecutionSuccess(bytes32,uint256)` event on success.
Keccak-256 hash of the event is `0x442e715f626346e8c54381002da614f62bee8d27386535b2521ec8540898556e`.
We will be tracking this event in the network transactions and emit an incident if it's found.

This how AssemblyScript Mamoru Daemon code may look like:
```typescript
import { query, report } from "@mamoru-ai/mamoru-sdk-as/assembly";

// `ExecutionSuccess(bytes32,uint256)` event
const SAFE_EXECUTION_SUCCESS_HASH: string = "0x442e715f626346e8c54381002da614f62bee8d27386535b2521ec8540898556e";

// the address of your Gnosis Safe
const MY_SAFE_ADDRESS: string = "0x785a205084ac256cad3133326abee31b5e53931a";

export function main(): void {
    let rows = query(`SELECT bytes_to_hex(e.topic0) AS topic0 FROM events e WHERE e.address = '${MY_SAFE_ADDRESS}'`);

    rows.forEach(row => {
        if (row.getString("topic0")!.valueOf() == SAFE_EXECUTION_SUCCESS_HASH) {
            report();
        }
    })
}
```
Replace `MY_SAFE_ADDRESS` to yours and put the resulting code into `./assembly/index.ts`.
Then, compile the code into WebAssembly binary:
```shell
npx asc ./assembly/index.ts --target release --exportRuntime
```
You'll see a file `./build/release.wasm` that is the compiled daemon in WebAssembly format.

### Here's another example of a WASM daemon
Here's an example of a Mamoru WASM daemon that listens for `ExecutionSuccess` events in a Gnosis Safe contract and reports the transaction hash for each detected event:

```typescript copy
// File: mamoru_daemon.ts

import { query, report } from "@mamoru-ai/mamoru-sdk-as/assembly";

const SAFE_EXECUTION_SUCCESS_HASH: string = "0x442e715f626346e8c54381002da614f62bee8d27386535b2521ec8540898556e";
const MY_SAFE_ADDRESS: string = "0x785a205084ac256cad3133326abee31b5e53931a";
const POLL_INTERVAL: u32 = 60000; // 60 seconds

export function main(): void {
    let lastProcessedBlock: u64 = 0;

    while (true) {
        lastProcessedBlock = checkExecutionSuccessEvents(lastProcessedBlock);
        sleep(POLL_INTERVAL);
    }
}

function checkExecutionSuccessEvents(lastBlock: u64): u64 {
    const latestBlockNumber = getLatestBlockNumber();
    const rows = queryEventsByAddressAndBlockRange(MY_SAFE_ADDRESS, lastBlock + 1, latestBlockNumber);

    rows.forEach(row => {
        if (isExecutionSuccessEvent(row)) {
            const txHash = row.getString("transaction_hash")!;
            report(txHash);
        }
    });

    return latestBlockNumber;
}

function getLatestBlockNumber(): u64 {
    const sqlQuery = "SELECT max(b.block_number) as latest_block FROM blocks b";
    const result = query(sqlQuery);
    const latestBlockNumber = result.getRow(0).getU64("latest_block");
    return latestBlockNumber;
}

function queryEventsByAddressAndBlockRange(address: string, startBlock: u64, endBlock: u64): ArrayBuffer {
    const sqlQuery = `
        SELECT bytes_to_hex(e.topic0) AS topic0, e.transaction_hash
        FROM events e
        WHERE e.address = '${address}' AND e.block_number BETWEEN ${startBlock} AND ${endBlock}`;

    return query(sqlQuery);
}

function isExecutionSuccessEvent(row: ArrayBuffer): boolean {
    const eventHash = row.getString("topic0")!.valueOf();
    return eventHash == SAFE_EXECUTION_SUCCESS_HASH;
}

function sleep(ms: u32): void {
    // Mamoru does not currently support blocking sleep operations in AssemblyScript.
    // You might need to implement this function using JavaScript and Web Workers or other non-blocking techniques.
}
```

## Testing

Testing are not implemented for daemons yet.
However, WASM daemons are validated during deploying phase.
You will receive an error message if your WASM module is invalid, some types are incompatible etc.

## Deploying

To deploy the daemon to Mamoru Validation Chain Testnet, you must submit a `register-daemon` transaction:

```shell
export DAEMON_BINARY=$(cat ./build/release.wasm | base64)

# TODO: the command is not valid yet, as per changes in `validation-chaind`
validation-chaind tx \
  validationchain register-daemon "{\"content\": \"${DAEMON_BINARY}\", \"chain\": {\"chain_type\": 4}}" \
  --from wasm-daemon-guide \
  --node https://validation-chain.testnet.mamoru.foundation:26657 \
  --chain-id validationchaintestnet
```

After a few minutes, Mamoru Sniffers will fetch your daemon and start running your daemon against blockchain transactions.
