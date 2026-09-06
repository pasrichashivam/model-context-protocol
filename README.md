# [Model Context Protocol](https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro)
> MCP is a standardized protocol that gives AI applications a consistent way to discover and use external tools, resources, and prompts, instead of requiring every AI application to build a separate custom integration for every system

**Non-Techincal Explaination**

>MCP is just a shared toolbox a standard collection of tools and APIs that any AI model can reach into, instead of every developer building their own private toolbox from scratch.

---

## Before vs. After MCP
<img src="./assets/02_mcp_before_after.png" width="1200" height="500">

| Aspect | Traditional Integration | MCP-Based Integration |
|---|---|---|
| **Integration setup** | Straightforward for individual APIs, with each LLM integrating directly through its own wrapper | Simplifies connectivity by allowing the host to connect directly to an MCP server |
| **Ownership & maintenance** | Each engineering team builds, manages, and updates its own integration, leading to duplicated effort at scale | Integration logic is maintained by the server owner, typically the API or service provider |
| **Handling API changes** | Any change to the underlying provider API requires each individual integration to be modified | The MCP server handles provider-side changes, allowing connected clients to remain unchanged |
| **Authentication & security** | Authentication is implemented and managed separately for each tool or integration | Authentication is still required, but responsibilities can be consolidated and managed centrally at the server layer |


<details>
  <summary><b>Before MCP</b></summary>
    <img src="./assets/10_before_mcp.png" width="800" height="300">
</details>

<details>
  <summary><b>After MCP</b></summary>
    <img src="./assets/11_after_mcp.png" width="800" height="300">
</details>

--- 

## The problem without MCP
* Imagine you have an AI assistant that needs to work with:
    * GitHub
    * Google Drive
    * Slack
    * A database
    * Company's internal APIs

* Without MCP, the AI application has to build a special integration for every service.
    ```text
                AI
                ├── Custom GitHub integration
                ├── Custom Slack integration
                ├── Custom Jira integration
                ├── Custom Database integration
                └── Custom Google Drive integration
    ```
* And every integration may work differently,The developer has to figure out:
    * How do I tell GitHub what I want?
    * How do I authenticate with Slack?
    * How do I retrieve data from this database?

## What MCP does
**MCP introduces a common protocol for AI applications to communicate with tools and data.**

```mermaid
flowchart LR
    U[👤 User] --> AI[🤖 AI Model]
    AI --> MCP[MCP Client]

    MCP --> JR[Jira]
    MCP --> SL[Slack]
    MCP --> DB[Database]
    AI --> U
```
* The AI doesn't need to understand every tool's unique integration mechanism.
* The MCP server exposes capabilities in a standardized way.
* MCP is like a standard USB interface for AI applications to connect with external tools and data.

### What actually happens
There are typically three important pieces:

```mermaid
flowchart LR
    A["AI Application<br/>/ MCP Host"]
    B["MCP Server<br/><br/>Tools<br/>Resources<br/>Prompts"]
    C["External System<br/><br/>GitHub / DB /<br/>Slack / etc."]

    A -->|MCP| B
    B --> C
```
* The MCP server acts as the standardized bridge.
* The important idea is that the AI application doesn't have to implement the entire Tools specific interaction itself.

### Example without MCP
* Suppose you're building an AI coding assistant.
* User asks: "Check GitHub issues and create a Jira ticket for the important ones"
* Without MCP:
    ```text
                    AI Application
                ┌─────────┴─────────┐
                ↓                   ↓
        GitHub integration   Jira integration
                ↓                   ↓
            GitHub API           Jira API
    ```
* Now imagine adding:
    * Slack
    * Database
    * Google Drive and many more.
* You end up with a growing collection of custom integrations.

### Example with MCP
* The AI application has a standard way of interacting with MCP servers.
* Each MCP server handles the specifics of its underlying system.

```text
                    AI Application
                           │
                           ↓ MCP
                 ┌──────────────────┐
                 │   MCP Servers    │
                 └────────┬─────────┘
             ┌────────────┼────────────┐
             ↓            ↓            ↓
        GitHub MCP    Jira MCP     Slack MCP
             ↓            ↓            ↓
          GitHub        Jira         Slack

```

## Takeaway 
**MCP is a standard protocol for exposing tools, resources, and prompts to AI applications**
```mermaid
flowchart LR
    A["AI<br/><br/>What should I do?"]
    B["MCP Server<br/><br/>Here are the capabilities<br/>you can use."]
    C["External System<br/><br/>GitHub / Jira / DB / Slack"]

A -->|MCP| B
B -->|API / SDK / DB driver| C
```

---

## [MCP Architecture & LifeCycle](./notes/01_mcp_architecture.md)

## Useful Resources for Learning MCP
* [**MCP Lifecycle**](https://mcp-lifecycle.netlify.app/)
* [**MCP Lifecycle — Conversation Simulator**](https://mcp-lifecycle-simulator.netlify.app/)
* [**MCP Explained**](https://ai-automation-with-mayank.netlify.app/#mcp)
