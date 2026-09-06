# Architecture overview

* MCP follows a client-server architecture.
* The key participants in the MCP architecture are:
    * **MCP Host:** The AI application that coordinates and manages one or multiple MCP clients.
    * **MCP Client:** A component that maintains a connection to an MCP server and obtains context from an MCP server for the MCP host to use.
    * **MCP Server**: A program that provides context to MCP clients.

<img src="../assets/01_mcp_architecture.png" width="1000" height="500">

* For example: Visual Studio Code acts as an MCP host. 
* When Visual Studio Code establishes a connection to an MCP server, such as the Database MCP server, the Visual Studio Code runtime instantiates an MCP client object that maintains the connection to the Database MCP server. 
* When Visual Studio Code subsequently connects to another MCP server, such as the local filesystem server, the Visual Studio Code runtime instantiates an additional MCP client object to maintain this connection.

---

## Layers
MCP consists of two layers:

1. **Data layer:** Defines the JSON-RPC based protocol for client-server communication, including version discovery and core primitives, such as `tools`, `resources`, `prompts` and `notifications`.
2. **Transport layer:** Defines the communication mechanisms and channels that enable data exchange between clients and servers.

### Data layer
Data layer implements a **JSON-RPC 2.0 (Remote procedure call)** based exchange protocol that defines the message structure. This includes:
1. **Lifecycle management**: Handles connection initialization, capability negotiation, and connection termination between clients and servers.
2. **Server features**: Enables servers to provide core functionality like tools for AI, resources for context, and prompts for templates from and to the client.
3. **Client features**: Enables servers to ask the client to sample from the host LLM, elicit input from the user, and log messages to the client.
4. **Utility features**: Supports additional capabilities like notifications for real-time updates and progress tracking for long-running operations.

### Transport layer
The transport layer manages communication channels and authentication between clients and servers, It supports two transport mechanisms:
1. **Stdio transport**: Uses standard input/output streams for direct process communication between local processes on the same machine, providing optimal performance with no network overhead.
    * The client launches the MCP server as a subprocess.
    * The server reads JSON-RPC messages from its standard input (stdin) and sends messages to its standard output (stdout).
    * Messages are individual JSON-RPC requests, notifications, or responses.
    * Messages are delimited by newlines, and MUST NOT contain embedded newlines.
    * The server MUST NOT write anything to its stdout that is not a valid MCP message.
    * The client MUST NOT write anything to the server’s stdin that is not a valid MCP message.
2. **Streamable HTTP transport**: 
    * Uses **HTTP POST** for client-to-server messages with optional **Server-Sent Events** for streaming capabilities. 
    * This transport supports HTTP authentication methods including bearer tokens, API keys, and custom headers. 
    * MCP recommends using OAuth to obtain authentication tokens.
    * The server MUST provide a single HTTP endpoint path that supports both POST and GET methods, this could be a URL like https://example.com/mcp.
    * The client MUST use HTTP POST to send JSON-RPC messages to the MCP endpoint.
    * The client MUST include an Accept header, listing both application/json and text/event-stream as supported content types.

--- 

## Data Layer Protocol

**Primitives**

<img src="../assets/06_server_promitives.png" width="600" height="300">

MCP primitives define what clients and servers can offer each other.
1. **Server Exposed Primitives:**
    1. **Tools**: Executable functions that AI applications can invoke to perform actions (e.g., API calls, database queries)
    2. **Resources**: Data sources that provide contextual information to AI applications (e.g., file contents, database records)
    3. **Prompts**: Reusable templates that help structure interactions with language models (e.g., system prompts, few-shot examples)

    > Each primitive type has associated methods for discovery `(*/list)`, retrieval `(*/get)`, and in some cases, execution (tools/call). <br>MCP clients will use the `*/list` methods to discover available primitives. <br>For example, a client can first list all available tools (tools/list) and then execute them.
2. **Client Exposed Primitives:**
    * **Sampling**: 
        * Allows servers to request language model completions from the client’s AI application. 
        * This is useful when server authors want access to a language model, using the `sampling/createMessage` method to request a LLM from the client’s AI application
    * **Elicitation**: 
        * Allows servers to request additional information from users.
        * This is useful when server authors want to get more information from the user, or ask for confirmation of an action.
        * They can use the `elicitation/create` method to request additional information from the user.
    * **Logging**: Enables servers to send log messages to clients for debugging and monitoring purposes.


**Lifecycle**

<img src="../assets/07_mcp_lifecycle.png" width="600" height="300">

1. **Initialization**
* MCP begins with lifecycle management through a capability negotiation handshake.
* The client sends an initialize request to establish the connection and negotiate supported features.
* The initialization process serves several critical purposes:
    1. **Protocol Version Negotiation**: The `protocolVersion` field (e.g., “2025-11-25”) ensures both client and server are using compatible protocol versions, If a mutually compatible version is not negotiated, the connection should be terminated.
    2. **Capability Discovery**: The `capabilities` object allows to declare what features they support, including which primitives they can handle (tools, resources, prompts).
    3. **Identity Exchange**: The `clientInfo` and `serverInfo` objects provide identification and versioning information for debugging and compatibility purposes.

<details>
  <summary><b>Initialize Request</b></summary>
    <img src="../assets/08_request.png" width="400" height="400">
</details>

<details>
  <summary><b>Initialize Response</b></summary>
        <img src="../assets/08_response.png" width="400" height="400">
</details>

2. **Tool Discovery (Primitives)**
* The client can discover available tools by sending a `tools/list` request.
<details>
  <summary><b>Tool List Request</b></summary>
    <img src="../assets/09_tool_list_request.png" width="400" height="200">
</details>

<details>
  <summary><b>Tool List Response</b></summary>
        <img src="../assets/09_tool_list_response.png" width="400" height="400">
</details>


