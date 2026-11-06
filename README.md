# HERE Location Reasoning Plugin

---

Agent plugin that provides your agent with location awareness through the HERE Location
Reasoning MCP, and access to HERE documentation through the HERE Documentation MCP.

Built as per the [Agent Plugins Specification](https://agent-plugins.org/specification),
and works across any conforming agent client.

> Learn more about HERE Location Reasoning in the [documentation](https://docs.here.com/location-reasoning/docs/hlroverview).

---

## Prerequisites

A HERE platform account with HERE Location Reasoning access.

---

## Setup

- Install the plugin in your client

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
    copilot plugin install heremaps/here-location-reasoning-plugin
    ```

    **Codex CLI**
    ```
    codex plugin marketplace add heremaps/here-location-reasoning-plugin
    codex plugin add here-location-reasoning-plugin --marketplace here-technologies
    ```

- Authenticate the HERE Location Reasoning MCP

---

## What you can do

**Location tasks**

- Example prompt: "Plan an EV route from Berlin to Munich and show the charging stops on the way"

**Get answers from HERE documentation**

- Example prompt: "How can I programmatically access the HERE Location Reasoning MCP?"
 
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