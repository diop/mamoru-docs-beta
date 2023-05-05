
# The Mamoru CLI

## Overview

Mamoru CLI is a system designed to rapidly develop Mamoru Daemon applications. It provides:

- Project scaffolding
- A build system
- A publishing system for validation chains

Mamoru CLI aims to be the standard tool for creating daemons within the [Mamoru project](https://mamoru.ai). We work continuously to ensure that each piece works smoothly and is easy to use.

## Components

### CLI

The CLI is a globally installed **[npm](https://www.npmjs.com/)** package that provides access to the `mamoru-cli` command in your terminal. It allows you to scaffold both SQL and [WASM](https://webassembly.org/) Daemons via `mamoru-cli init`.

### ValidationChain

The validation chain is the core of Mamoru. It validates incoming incident reports, manages Daemons and DaemonMetadatas, and registers them in the blockchain.

### Daemon

A daemon is an entity deployed on a specified blockchain that detects and emits incidents to the validation chain.

### Daemon Metadata

Each daemon uses a "master template" that can be defined as a series of SQL queries or WASM code. This template is called "Daemon Metadata" and can be shared among many daemons if the developer allows it. Sharing a single metadata as the common source for several daemons allows each daemon to have a set of custom parameters defined during the creation of the metadata.

### The centralized platform

The centralized platform is responsible for listening to all events from the validation chain and sending notifications (SMS, email, webhook, Discord, etc.) to several subscribers. Users can manually or programmatically subscribe to daemon events and provide custom templates to format the data.
