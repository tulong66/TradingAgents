# System Architecture

## 🏗️ High-Level Architecture Overview

TradingAgents follows a **layered, modular architecture** designed for scalability, maintainability, and extensibility. The system is built around the concept of specialized AI agents working together in a coordinated workflow.

```mermaid
graph TB
    subgraph "Presentation Layer"
        CLI[CLI Interface]
        API[API Interface]
        WEB[Web Interface]
    end
    
    subgraph "Orchestration Layer"
        TG[TradingAgentsGraph]
        WE[Workflow Engine]
        SM[State Manager]
    end
    
    subgraph "Agent Layer"
        subgraph "Analysts"
            MA[Market Analyst]
            SA[Social Analyst]
            NA[News Analyst]
            FA[Fundamentals Analyst]
        end
        
        subgraph "Researchers"
            BR[Bull Researcher]
            BER[Bear Researcher]
            RM[Research Manager]
        end
        
        subgraph "Trading"
            TR[Trader]
        end
        
        subgraph "Risk Management"
            RA[Risky Analyst]
            CA[Conservative Analyst]
            NEA[Neutral Analyst]
            PM[Portfolio Manager]
        end
    end
    
    subgraph "Service Layer"
        LLM[LLM Services]
        MEM[Memory Services]
        TOOL[Tool Services]
    end
    
    subgraph "Data Layer"
        DS[Data Sources]
        CACHE[Data Cache]
        STORE[Persistent Storage]
    end
    
    CLI --> TG
    API --> TG
    WEB --> TG
    
    TG --> WE
    TG --> SM
    
    WE --> MA
    WE --> SA
    WE --> NA
    WE --> FA
    WE --> BR
    WE --> BER
    WE --> RM
    WE --> TR
    WE --> RA
    WE --> CA
    WE --> NEA
    WE --> PM
    
    MA --> LLM
    SA --> LLM
    NA --> LLM
    FA --> LLM
    BR --> MEM
    BER --> MEM
    TR --> TOOL
    
    LLM --> DS
    TOOL --> DS
    MEM --> STORE
    DS --> CACHE
```

## 🔧 Core Components

### 1. TradingAgentsGraph (Orchestrator)

The central orchestrator that coordinates all agents and manages the overall workflow.

```mermaid
classDiagram
    class TradingAgentsGraph {
        -config: Dict
        -llm_providers: Dict
        -memories: Dict
        -tool_nodes: Dict
        -graph: StateGraph
        +__init__(selected_analysts, debug, config)
        +propagate(company_name, trade_date)
        +reflect_and_remember(returns_losses)
        +process_signal(full_signal)
        -_create_tool_nodes()
        -_log_state(trade_date, final_state)
    }
    
    class GraphSetup {
        +setup_graph(selected_analysts)
        +create_workflow()
        +add_conditional_edges()
    }
    
    class Propagator {
        +create_initial_state()
        +get_graph_args()
    }
    
    class Reflector {
        +reflect_bull_researcher()
        +reflect_bear_researcher()
        +reflect_trader()
    }
    
    TradingAgentsGraph --> GraphSetup
    TradingAgentsGraph --> Propagator
    TradingAgentsGraph --> Reflector
```

**Key Responsibilities**:
- Initialize and configure all agents
- Manage the workflow execution
- Handle state transitions
- Coordinate inter-agent communication
- Process final decisions

### 2. Agent Architecture

Each agent follows a consistent architectural pattern:

```mermaid
graph TD
    subgraph "Agent Structure"
        A[Agent Function] --> B[Input Processing]
        B --> C[LLM Interaction]
        C --> D[Tool Usage]
        D --> E[Memory Access]
        E --> F[Output Generation]
        F --> G[State Update]
    end
    
    subgraph "Agent Components"
        H[Prompt Template]
        I[Tool Binding]
        J[Memory System]
        K[State Manager]
    end
    
    B --> H
    C --> I
    E --> J
    G --> K
```

#### Agent Implementation Pattern
```python
def create_agent(llm, toolkit, memory=None):
    def agent_node(state):
        # 1. Extract relevant state information
        context = extract_context(state)
        
        # 2. Access memory for past experiences
        if memory:
            past_memories = memory.get_memories(context)
        
        # 3. Create prompt with context and memories
        prompt = create_prompt(context, past_memories)
        
        # 4. Bind tools if needed
        chain = prompt | llm.bind_tools(tools)
        
        # 5. Execute analysis
        result = chain.invoke(state["messages"])
        
        # 6. Update state
        return update_state(result)
    
    return agent_node
```

### 3. Memory System Architecture

```mermaid
graph TB
    subgraph "Memory Architecture"
        A[FinancialSituationMemory] --> B[Vector Store]
        A --> C[Similarity Search]
        A --> D[Memory Retrieval]
        
        B --> E[ChromaDB]
        C --> F[Embedding Model]
        D --> G[Context Matching]
    end
    
    subgraph "Memory Types"
        H[Bull Memory]
        I[Bear Memory]
        J[Trader Memory]
        K[Risk Manager Memory]
    end
    
    H --> A
    I --> A
    J --> A
    K --> A
```

**Memory System Features**:
- **Semantic Search**: Find relevant past experiences
- **Context Awareness**: Match current situation to historical patterns
- **Learning Integration**: Update memories based on outcomes
- **Agent-Specific**: Each agent maintains its own memory

### 4. Data Flow Architecture

```mermaid
sequenceDiagram
    participant U as User
    participant TG as TradingGraph
    participant A as Agents
    participant T as Tools
    participant D as Data Sources
    participant M as Memory
    
    U->>TG: Request Analysis
    TG->>A: Initialize Agents
    
    loop For Each Agent
        A->>M: Retrieve Past Experiences
        M-->>A: Relevant Memories
        A->>T: Request Data
        T->>D: Fetch Data
        D-->>T: Raw Data
        T-->>A: Processed Data
        A->>A: Analyze & Reason
        A-->>TG: Analysis Result
    end
    
    TG->>TG: Synthesize Results
    TG-->>U: Final Decision
    
    Note over TG,M: Post-Analysis Learning
    TG->>M: Store Experience
```

## 🔄 Workflow Engine

### State Management

The system uses a sophisticated state management system to track the analysis progress:

```mermaid
stateDiagram-v2
    [*] --> InitialState
    
    InitialState --> AnalystPhase
    
    state AnalystPhase {
        [*] --> MarketAnalyst
        MarketAnalyst --> SocialAnalyst
        SocialAnalyst --> NewsAnalyst
        NewsAnalyst --> FundamentalsAnalyst
        FundamentalsAnalyst --> [*]
    }
    
    AnalystPhase --> ResearchPhase
    
    state ResearchPhase {
        [*] --> BullResearcher
        BullResearcher --> BearResearcher
        BearResearcher --> BullResearcher : Debate
        BullResearcher --> ResearchManager : Complete
        BearResearcher --> ResearchManager : Complete
        ResearchManager --> [*]
    }
    
    ResearchPhase --> TradingPhase
    TradingPhase --> RiskPhase
    
    state RiskPhase {
        [*] --> RiskAnalysts
        RiskAnalysts --> PortfolioManager
        PortfolioManager --> [*]
    }
    
    RiskPhase --> FinalDecision
    FinalDecision --> [*]
```

### Agent State Schema

```mermaid
erDiagram
    AgentState {
        string company_of_interest
        string trade_date
        list messages
        string market_report
        string sentiment_report
        string news_report
        string fundamentals_report
        object investment_debate_state
        string investment_plan
        string trader_investment_plan
        object risk_debate_state
        string final_trade_decision
    }
    
    InvestDebateState {
        string bull_history
        string bear_history
        string history
        string current_response
        string judge_decision
        int count
    }
    
    RiskDebateState {
        string risky_history
        string safe_history
        string neutral_history
        string history
        string judge_decision
        int count
    }
    
    AgentState ||--|| InvestDebateState : contains
    AgentState ||--|| RiskDebateState : contains
```

## 🛠️ Tool System Architecture

### Tool Organization

```mermaid
graph TB
    subgraph "Tool Categories"
        A[Market Data Tools]
        B[News & Sentiment Tools]
        C[Fundamental Analysis Tools]
        D[Technical Analysis Tools]
    end
    
    subgraph "Tool Implementations"
        A --> A1[YFinance]
        A --> A2[StockStats]
        
        B --> B1[Google News]
        B --> B2[Reddit API]
        B --> B3[Finnhub News]
        
        C --> C1[SimFin]
        C --> C2[Insider Trading]
        C --> C3[Company Metrics]
        
        D --> D1[Technical Indicators]
        D --> D2[Chart Patterns]
        D --> D3[Volume Analysis]
    end
    
    subgraph "Tool Nodes"
        E[Market Tool Node]
        F[Social Tool Node]
        G[News Tool Node]
        H[Fundamentals Tool Node]
    end
    
    A1 --> E
    A2 --> E
    B1 --> F
    B2 --> F
    B3 --> G
    C1 --> H
    C2 --> H
    C3 --> H
```

### Tool Interface Design

```python
class Toolkit:
    def __init__(self, config):
        self.config = config
        self.online_mode = config.get("online_tools", True)
    
    # Market Data Tools
    def get_YFin_data_online(self, ticker: str, period: str) -> str:
        """Fetch real-time market data"""
        pass
    
    def get_stockstats_indicators_report_online(self, ticker: str) -> str:
        """Calculate technical indicators"""
        pass
    
    # News & Sentiment Tools
    def get_google_news(self, query: str) -> str:
        """Fetch latest news"""
        pass
    
    def get_reddit_stock_info(self, ticker: str) -> str:
        """Analyze social sentiment"""
        pass
    
    # Fundamental Analysis Tools
    def get_fundamentals_openai(self, ticker: str) -> str:
        """Get fundamental analysis"""
        pass
```

## 🧠 LLM Integration Architecture

### Multi-Provider Support

```mermaid
graph TB
    subgraph "LLM Abstraction Layer"
        A[LLM Factory]
        B[Provider Interface]
        C[Model Configuration]
    end
    
    subgraph "Supported Providers"
        D[OpenAI]
        E[Anthropic]
        F[Google]
        G[Custom/Local]
    end
    
    subgraph "Model Types"
        H[Deep Thinking Models]
        I[Quick Thinking Models]
        J[Specialized Models]
    end
    
    A --> B
    B --> D
    B --> E
    B --> F
    B --> G
    
    C --> H
    C --> I
    C --> J
```

### LLM Configuration Strategy

```python
class LLMConfig:
    def __init__(self, config):
        self.provider = config["llm_provider"]
        self.deep_model = config["deep_think_llm"]
        self.quick_model = config["quick_think_llm"]
        self.backend_url = config["backend_url"]
    
    def create_llms(self):
        if self.provider == "openai":
            deep_llm = ChatOpenAI(
                model=self.deep_model,
                base_url=self.backend_url
            )
            quick_llm = ChatOpenAI(
                model=self.quick_model,
                base_url=self.backend_url
            )
        elif self.provider == "anthropic":
            deep_llm = ChatAnthropic(
                model=self.deep_model,
                base_url=self.backend_url
            )
            quick_llm = ChatAnthropic(
                model=self.quick_model,
                base_url=self.backend_url
            )
        # ... other providers
        
        return deep_llm, quick_llm
```

## 📊 Data Architecture

### Data Source Integration

```mermaid
graph LR
    subgraph "External APIs"
        A[Yahoo Finance API]
        B[Finnhub API]
        C[Reddit API]
        D[Google News API]
    end
    
    subgraph "Data Processing"
        E[Data Fetchers]
        F[Data Validators]
        G[Data Transformers]
    end
    
    subgraph "Caching Layer"
        H[Memory Cache]
        I[File Cache]
        J[Database Cache]
    end
    
    subgraph "Data Interface"
        K[Unified Data API]
        L[Tool Functions]
    end
    
    A --> E
    B --> E
    C --> E
    D --> E
    
    E --> F
    F --> G
    G --> H
    G --> I
    G --> J
    
    H --> K
    I --> K
    J --> K
    K --> L
```

### Caching Strategy

```mermaid
flowchart TD
    A[Data Request] --> B{Cache Hit?}
    B -->|Yes| C[Return Cached Data]
    B -->|No| D[Fetch from Source]
    D --> E[Validate Data]
    E --> F{Valid?}
    F -->|Yes| G[Cache Data]
    F -->|No| H[Log Error]
    G --> I[Return Data]
    H --> J[Return Error]
    
    C --> K[Update Access Time]
    I --> L[Log Success]
```

## 🔧 Configuration Architecture

### Hierarchical Configuration

```mermaid
graph TB
    A[Default Config] --> B[Environment Config]
    B --> C[User Config]
    C --> D[Runtime Config]
    
    subgraph "Config Categories"
        E[LLM Settings]
        F[Data Sources]
        G[Agent Behavior]
        H[System Settings]
    end
    
    D --> E
    D --> F
    D --> G
    D --> H
```

### Configuration Schema

```python
DEFAULT_CONFIG = {
    # System Settings
    "project_dir": "path/to/project",
    "results_dir": "./results",
    "data_cache_dir": "./cache",
    
    # LLM Settings
    "llm_provider": "openai",
    "deep_think_llm": "gpt-4",
    "quick_think_llm": "gpt-3.5-turbo",
    "backend_url": "https://api.openai.com/v1",
    
    # Agent Behavior
    "max_debate_rounds": 3,
    "max_risk_discuss_rounds": 2,
    "max_recur_limit": 100,
    
    # Tool Settings
    "online_tools": True,
}
```

## 🚀 Deployment Architecture

### Scalability Considerations

```mermaid
graph TB
    subgraph "Load Balancer"
        A[Request Router]
    end
    
    subgraph "Application Tier"
        B[TradingAgents Instance 1]
        C[TradingAgents Instance 2]
        D[TradingAgents Instance N]
    end
    
    subgraph "Service Tier"
        E[LLM Service Pool]
        F[Memory Service]
        G[Data Service]
    end
    
    subgraph "Data Tier"
        H[Cache Cluster]
        I[Database Cluster]
        J[External APIs]
    end
    
    A --> B
    A --> C
    A --> D
    
    B --> E
    B --> F
    B --> G
    C --> E
    C --> F
    C --> G
    D --> E
    D --> F
    D --> G
    
    E --> H
    F --> I
    G --> J
```

## 🔍 Monitoring and Observability

### System Monitoring

```mermaid
graph TB
    subgraph "Metrics Collection"
        A[Performance Metrics]
        B[Error Metrics]
        C[Business Metrics]
    end
    
    subgraph "Logging"
        D[Application Logs]
        E[Agent Logs]
        F[Decision Logs]
    end
    
    subgraph "Tracing"
        G[Request Tracing]
        H[Agent Workflow Tracing]
        I[LLM Call Tracing]
    end
    
    subgraph "Alerting"
        J[Performance Alerts]
        K[Error Alerts]
        L[Business Alerts]
    end
    
    A --> J
    B --> K
    C --> L
    
    D --> G
    E --> H
    F --> I
```

---

This architecture provides a solid foundation for building, scaling, and maintaining the TradingAgents system while ensuring flexibility for future enhancements.

**Next**: Explore the detailed [Workflow Design](workflow-design.md) to understand how agents collaborate.