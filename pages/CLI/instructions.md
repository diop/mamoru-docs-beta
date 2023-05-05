# Mamoru CLI Quick Start

## Installation

<aside style="background-color: #e8feea; padding:10px">
✅  <b>Node version requirements</b>

Mamoru-CLI requires [Node.js](https://nodejs.org/en/) 16.x.x, you can manage multiple version of Node on the same machine using [nvm (MacOs, Linux)](https://github.com/nvm-sh/nvm) or [nvm for windows](https://github.com/coreybutler/nvm-windows)

</aside>

To install the new package, use one of the following commands. You will need administrator privileges to execute these commands, unless npm was installed on your system through a Node.js version manager (such as n or nvm).

### Using NPM

```jsx
npm install -g mamoru-cli
```

### Using Yarn

```jsx
yarn global add mamoru-cli
```

## Upgrading

To upgrade the global Mamoru-CLI package, run:

### Using NPM

```jsx
npm update -g mamoru-cli
```

### Using Yarn

```jsx
yarn global upgrade --latest mamoru-cli
```

# Creating a new project

To create a new project, run:

```jsx
mamoru-cli init my-new-daemon --type sql --chain sui
```

This command will create a SQL project targeting *[sui blockchain](https://sui.io/)* in the "my-new-daemon" folder. To see all available parameters for the command, please check the reference

Once the project is created, you will see a prompt similar to this:

```bash

```

To get started, move to your directory and install dependencies using either `npm install` or `yarn`.

You should see four files for SQL daemons:

- `Readme.md`: This is a placeholder readme file for your project.
- `manifest.yml`: This file defines basic information about the daemon, such as its type, name, a custom logo URL, the target chain, and custom tags for discoverability.
- `package.json`: This is a minimal standard `package.json` file from NPM. It allows you to install custom AssemblyScript packages from the npm registry, and it's especially useful for WASM daemons. It also contains the build scripts used for WASM daemons.
- `queries.yml`: This file defines the queries that the daemon will run. At the time of deployment, these queries will be sent in the form of daemon metadata.

# Building the project.

The building step is only required for WASM daemons. During this step, your [AssemblyScript](https://www.assemblyscript.org/) source code will be transformed into a `.wasm` file that will later be sent to the validation chain as a daemon metadata. The dependencies required for compiling the source code are specified in the `package.json` file of each project, so each project can evolve at a different pace.

To run the build step use

```bash
mamoru-cli build
```

Upon successful execution of the command, a `build` directory will appear in your project root.

# Publishing the daemon.

Once your project is completed, you can deploy your first daemon using the following command:

```bash
mamoru-cli deploy --private-key <YOUR_PRIVATE_KEY>
```

The deploy command works in slightly different ways depending on the characteristics of your Daemon. Here, we provide a basic overview of these differences.

## For SQL Daemons

In this case, the deploy command verifies the `manifest.yml` and `queries.yml` files, transforms and signs the transaction using your private key, and sends it to the validation chain as a *Daemon Metadata*.

## For WASM Daemons

The `deploy` command verifies the existence of the `build` folder and performs a small validation of the sources. It then converts the binary to base64 format, creates and signs the transaction using the private key, and finally uses it to create a *Daemon Metadata*.

Keep in mind that the base64 string of the binary may require extra gas to be processed. Depending on your case, you may need to add some extra gas. To do this, use the `--gas` flag.

```bash
mamoru-cli deploy --private-key <YOUR_PRIVATE_KEY> --gas 1000000
```

## For SOLE or SUBSCRIBABLE Daemons

At this point, you may have noticed a `subscribable` field in your **manifest.yml** file. Daemons can have two types: **SOLE** or **SUBSCRIBABLE**.

Creating a **SOLE** daemon means that only you can spawn new instances of the daemon metadata. In this case, running the `deploy` command will first create the *Daemon Metadata*, and then the *Daemon* itself in a subsequent call.

Creating a **SUBSCRIBABLE** daemon means that the `deploy` command will only create a *Daemon Metadata*, and the *Daemon* instance can be spawned later by you or any other person interacting with the validation chain.