# Apache Camel AI Agent & MCP Studio (Open Source Demo)

A clean, standalone demo illustrating **Apache Camel YAML DSL**, **AI Tool invocation**, **MCP (Model Context Protocol)**, an **AI Agent**, and a **Web UI** with zero proprietary dependencies.

All external endpoints (LLM, Prometheus, Weather, Vector Store) are mocked with synthetic sample endpoints by default.

---

## 📁 Project Architecture

```text
camel-ai-agent-demo/
├── config/
│   └── application.properties       # Standalone config & mock endpoint routes
├── routes/
│   ├── tools.camel.yaml             # AI Tool declarations (ai-tool:prometheus, ai-tool:weather)
│   ├── agent.camel.yaml             # AI Agent orchestration & prompt template
│   ├── mcp.camel.yaml               # MCP server route exposing registered tools
│   ├── web.camel.yaml               # Web UI and chat API platform-http routes
│   └── mocks.camel.yaml             # Dummy mock Prometheus, Weather, and LLM APIs
├── ui/
│   └── index.html                   # Unbranded Web Chat UI
└── README.md
```


## 🏛️ Architecture Diagram

```mermaid
flowchart TB
    subgraph Clients["Clients & Interfaces"]
        UI["Web Browser Chat UI\n(index.html / /)"]
        MCP_CLIENT["MCP Client / IDE\n(Claude, Cursor, Bruno)"]
    end

    subgraph CamelEngine["Apache Camel Runtime (YAML DSL & CLI)"]
        direction TB
        
        HTTP_IN["platform-http:/api/chat"]
        MCP_IN["platform-http:/mcp\n(MCP Server)"]
        
        AGENT["AI Agent Route\n(direct:aiChat)"]
        
        subgraph Tools["Registered AI Tools"]
            TOOL_PROM["ai-tool:prometheus\n(PromQL Diagnostics)"]
            TOOL_WEATHER["ai-tool:weather\n(Weather Lookup)"]
        end
    end

    subgraph Services["Endpoints (Mock / Live)"]
        MOCK_PROM["Mock Prometheus API\n(/mock/prometheus/...)"]
        MOCK_WEATHER["Mock Weather API\n(/mock/weather/...)"]
        LLM["LLM / OpenAI API\n(Live or Mock Chat)"]
    end

    %% Client flows
    UI -->|HTTP POST JSON| HTTP_IN
    HTTP_IN --> AGENT
    MCP_CLIENT -->|JSON-RPC 2.0| MCP_IN
    MCP_IN -.->|Exposes / Invokes| Tools

    %% Agent & Tool execution
    AGENT -->|Chat Completion| LLM
    AGENT -->|Function Calling| Tools
    TOOL_PROM -->|HTTP GET| MOCK_PROM
    TOOL_WEATHER -->|HTTP GET| MOCK_WEATHER4


### Key Highlights:
- **Clients**: Supports both standard browser HTTP chat and external MCP-compatible clients.
- **Camel Core**: Orchestrates tools, MCP protocol, and agent routing purely in YAML.
- **Endpoints**: Decoupled backend for easy swapping between local mocks and real APIs.


---

## 🚀 Running with Camel CLI (`camel-jbang`)

Ensure you have [Camel CLI](https://camel.apache.org/manual/camel-jbang.html) installed:

```bash
# Run all YAML routes and configuration
camel run routes/*.yaml --properties=config/application.properties
```

Once running:
- Open your browser at **`http://localhost:8080/`** to interact with the Chat UI.
- Use the MCP endpoint at **`http://localhost:8080/mcp`**.
- Verify mock Prometheus data at **`http://localhost:8080/mock/prometheus/api/v1/query?query=up`**.

---

## 🛠️ Switching to Live Endpoints

To use real external endpoints:
1. Edit `config/application.properties`.
2. Disable mocks:
   ```properties
   mock.prometheus.enabled = false
   prometheus.url = https://your-prometheus-server/api/v1/query
   spring.ai.openai.api-key = <your-openai-key>
   spring.ai.openai.base-url = https://api.openai.com
   ```
3. Run `camel run` with your updated configuration.

---

## 📜 License
Apache-2.0
