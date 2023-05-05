# SQL Daemon Development

SQL Daemons allow you to define incident matching logic using SQL.
You can write an SQL query against a blockchain context, and if it returns something, the system emits an incident.

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
validation-chaind keys add sql-daemon-guide
```

This commands adds an account named `sql-daemon-guide` to your keychain.
Take the newly created account address (look like `cosmos1qaar0r0tyyy6ufslam0hv9zvhvy0z0cv5x8xev`) and fund it with some tokens.

// TODO: make public faucet for a testnet

### Define contract for monitoring

In this guide, we will work with Sui Capy in Sui Devnet network.
The contract source code can be found [here](https://github.com/MystenLabs/sui/tree/main/sui_programmability/examples/capy).

## Creating a Daemon

Let's say we want to emit an incident if a specific Capy is listed on the marketplace.
The function we need is called `list` and the code is
located [here](https://github.com/MystenLabs/sui/blob/5478f7b4ba3de99c38bfcb4e6afa14496dba908e/sui_programmability/examples/capy/sources/capy_market.move#L126-L147).

With Mamoru, there is two ways to track an item listing:

#### Direct lookup for a function call

Mamoru traces every function call in the supported blockchains.
This way, we can write an SQL query that reacts to an arbitrary function call with arbitrary arguments.

Follow the blockchain schema for available fields.
// TODO: publish schemas for blockchains

Given that the published smart-contract module address is `0xA::capy_market` and the Capy id is `0xB`,
the query looks like this:

```
SELECT * FROM transactions t
INNER JOIN call_traces ct ON t.seq = ct.tx_seq
INNER JOIN call_trace_args cta ON ct.seq = cta.call_trace_seq
WHERE 
	ct.transaction_module = "0xA::capy_market"
	AND ct.function = "list"
	AND as_string(cta.arg) = "0xB"
```

That query means: Mamoru will emit an incident if `0xA::capy_market::list` is called with `0xB` in arguments.

// TODO: add argument number to `call_trace_args` to query like `arg number N is X`

#### Track for `ItemListed` event

Mamoru traces all events that are emitted in the supported blockchains.
As per the contract code, `list` function emits `ItemListed` event which we can also track.

Given that the published smart-contract module address is `0xA::capy_market` and the Capy id is `0xB`,
the query looks like this:

```
// TODO: support field parsing in custom Move events ¯\_(ツ)_/¯
```

## Testing

Testing are not implemented for daemons yet.
However, queries are validated during deploying phase.
You will receive an error message if your query has syntax issues, wrong tables/fields usage, types incompatibility etc.

## Deploying

To deploy the SQL daemon to Mamoru Validation Chain Testnet, you must submit a `register-daemon` transaction:

```shell
validation-chaind tx \
  validationchain register-daemon $'{"content": "YOUR_SQL_QUERY", "chain": {"chain_type": 0}, "activateSince": "2022-12-13T15:35:58+00:00", "inactivateSince": "2032-12-13T15:35:58+00:00"}' \
  --from sql-daemon-guide \
  --node https://validation-chain.testnet.mamoru.foundation:26657 \
  --chain-id validationchaintestnet
```

After a few minutes, Mamoru Sniffers will fetch your daemon and start querying new transaction against your query.
