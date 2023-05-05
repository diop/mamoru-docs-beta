# The Mamoru Network Architecture

The Mamoru Network is a robust and scalable architecture designed to facilitate seamless incident monitoring, detection, and reporting within blockchain ecosystems. The architecture consists of several interconnected components, working together to provide developers with an efficient and reliable solution for managing incidents.

## Key Components of the Mamoru Network Architecture

1. **Validation Chain**: The Validation Chain is the backbone of the Mamoru Network, responsible for validating incoming incident reports, managing Daemons, and handling Daemon Metadata. It ensures the integrity and reliability of the network by registering all essential data on the blockchain.

2. **Daemons**: Daemons are specialized entities deployed on specified blockchains to detect and emit incidents to the Validation Chain. They are designed to work with both SQL and WASM code, allowing developers to define and implement custom incident detection logic.

3. **Daemon Metadata**: Daemon Metadata serves as a "master template" consisting of SQL queries or WASM code. This template can be shared among multiple Daemons, enabling each Daemon to have a set of custom parameters defined during its creation. Daemon Metadata allows developers to create and manage a more modular and reusable codebase.

4. **Centralized Platform**: The centralized platform is responsible for listening to all events from the Validation Chain and dispatching notifications (SMS, email, webhook, Discord, etc.) to subscribers. Users can manually or programmatically subscribe to Daemon events and provide custom templates to format the data, ensuring a tailored incident reporting experience.

5. **CLI**: The Command Line Interface (CLI) is a globally installed npm package that offers access to the mamoru-cli command in your terminal. It enables developers to scaffold SQL and WASM Daemons with ease using the mamoru-cli init command.

The Mamoru Network Architecture is built to ensure seamless integration with various blockchain environments, providing developers with a comprehensive solution for incident detection, reporting, and management. Its modular and flexible design makes it an ideal choice for developers looking to enhance their blockchain ecosystem's security and performance.
