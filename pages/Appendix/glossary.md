# Glossary of Mamoru Terms

## CLI
The CLI is a globally installed `npm` package that provides access to the `mamoru-cli` command in your terminal. It allows you to scaffold both SQL and WASM Daemons via `mamoru-cli init`.

## Validation Chain
The validation chain is the core of Mamoru. It validates incoming incident reports, manages Daemons and DaemonMetadatas, and registers them in the blockchain.

## Daemon
A daemon is an entity deployed on a specified blockchain that detects and emits incidents to the validation chain.

## SQl Daemon
SQL Daemons allow you to define incident matching logic using SQL. You can write an SQL query against a blockchain context, and if it returns something, the system emits an incident.

## Wasm Daemon
WASM Daemons allow you to define incident matching logic using any language that can be compiled to WebAssembly. Currently, only `AssemblyScript` is supported. You can write a custom code against a blockchain context, decide when to emit an incident, customize message, severity, etc.

## Daemon Metadata
Each daemon uses a "master template" that can be defined as a series of SQL queries or WASM code. This template is called "Daemon Metadata" and can be shared among many daemons if the developer allows it. Sharing a single metadata as the common source for several daemons allows each daemon to have a set of custom parameters defined during the creation of the metadata.

## The Centralized Platform
The centralized platform is responsible for listening to all events from the validation chain and sending notifications (SMS, email, webhook, Discord, etc.) to several subscribers. Users can manually or programmatically subscribe to daemon events and provide custom templates to format the data.

© Copyright 2023 Mamoru.ai - MIT License