# HERE Location Reasoning Plugin

---

Agent plugin that gives your agent HERE location awareness through the HERE Location
Reasoning MCP, plus access to its documentation.

Built as per the [Agent Plugins Specification](https://agent-plugins.org/specification),
and works across any conforming agent client.

Learn more about HERE Location Reasoning on its [product page](https://www.here.com/solutions/here-location-reasoning), or from the [documentation](https://docs.here.com/location-reasoning/docs/hlroverview).

---

## Prerequisites

- **A [HERE platform](https://platform.here.com/portal/) account.** This is a general HERE account, not specific to HERE Location Reasoning. If your organization already has one, ask your organization admin to invite you. Otherwise, [get started](https://www.here.com/platform), verify your email, and [sign in](https://platform.here.com) to the HERE platform.
- **An organization provisioned for HERE Location Reasoning.** Subscribe your organization to HERE Location Reasoning (also available as a self-service option on the [AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-5hdq5s532zfrw)), or ask the owner of an existing subscription to invite you. HERE Location Reasoning cannot be used without a provisioned account.

For enterprise evaluations, contact [HERE Sales](https://www.here.com/contact) or AWSHLRSales@here.com.

---

## Setup

- Install the plugin in your client. The following are commands for some CLI clients; check your client's own documentation for how to install plugins.

    **Kiro CLI**
    ```
    kiro-cli powers install here-location-reasoning-plugin
    ```

    **Claude Code CLI**
    ```
    claude plugin install here-location-reasoning-plugin --marketplace heremaps/here-location-reasoning-plugin
    ```

    **GitHub Copilot CLI**
    ```
    copilot plugin marketplace add heremaps/here-location-reasoning-plugin
    copilot plugin install here-location-reasoning-plugin@here-technologies
    ```

    **Codex CLI**
    ```
    codex plugin marketplace add heremaps/here-location-reasoning-plugin
    codex plugin add here-location-reasoning-plugin --marketplace here-technologies
    ```

- Find the `here-location-reasoning` MCP server installed through the plugin and authenticate it. This will take you through an interactive, browser-based sign-in, where you allow the client to access your HERE account.

---

## What you can do

**Location tasks**: geocoding, routing (including EV and public transit), place search, real-time traffic, reachable-area (isoline) calculations, and interactive maps. See the [tools reference](https://docs.here.com/location-reasoning/docs/hlrtools-reference) for the complete list of tools.

- Example prompt: "Show me Italian restaurants near Union Square in San Francisco"
- Example prompt: "Give me walking directions from the Colosseum to the Vatican in Rome"

**Get answers from HERE Location Reasoning documentation**: setup, authentication, integration, and usage.

- Example prompt: "How do I connect HERE Location Reasoning to my agent as an MCP server?"
- Example prompt: "How do I authenticate HERE Location Reasoning in my own application?"
 
---

## Support contact

For support please contact support@here.com

---

## Privacy policy

See [HERE Privacy](https://www.here.com/en-gb/privacy) for information about privacy and data handling.

---

## License

Copyright (C) 2026 HERE Europe B.V.

This project is licensed under the Apache License, Version 2.0 - see the [LICENSE](LICENSE) file for details