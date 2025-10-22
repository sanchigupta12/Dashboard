# CSV to Dashboard AI - Multi-Agent Data Analytics Platform

## Overview

CSV to Dashboard AI is an intelligent data analytics application that transforms CSV files into interactive dashboards using AI-powered analysis. The system employs a multi-agent architecture powered by Google's Gemini API to analyze data, detect business domains, suggest visualizations, and enable natural language interaction with data. The application provides both a Streamlit-based web interface and a Flask REST API backend, offering flexibility in deployment and integration.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Multi-Agent Architecture Pattern

The application implements a specialized agent-based system where each agent has a distinct responsibility in the data analytics pipeline:

**Agent Design Philosophy:**
- **Separation of Concerns**: Each agent handles a specific domain (analysis, planning, visualization, execution, memory)
- **AI Integration**: Strategic use of Google Gemini API for agents requiring natural language understanding and generation
- **Stateless Agents**: Agents don't maintain state; context is passed through method calls and managed by MemoryAgent
- **Composability**: Agents can be orchestrated in different workflows depending on the frontend (Streamlit vs Flask)

**Core Agents:**

1. **DataIntelligenceAgent** (`agents/data_intelligence.py`)
   - Purpose: Analyzes CSV structure, detects business domain, assesses data quality
   - AI-Powered: Uses Gemini API for domain detection and intelligent analysis
   - Outputs: Column classifications (numeric, categorical, datetime), domain identification with confidence scores, data quality metrics

2. **PlannerAgent** (`agents/planner.py`)
   - Purpose: Implements ReAct (Reasoning + Acting) methodology to determine available tasks
   - Non-AI: Uses rule-based logic for task recommendations
   - Logic: Reasons about data state and recommends appropriate next actions (chat, visualize)

3. **VisualizationAgent** (`agents/visualization.py`)
   - Purpose: Generates domain-specific visualization recommendations
   - AI-Powered: Uses Gemini API to suggest 6-7 contextually relevant charts
   - Strategy: Considers domain type, column types, and data characteristics for intelligent suggestions

4. **ExecutorAgent** (`agents/executor.py`)
   - Purpose: Handles chat interactions and dashboard generation
   - AI-Powered: Uses Gemini API for natural language Q&A about data
   - Features: Business-focused responses with domain-specific terminology

5. **MemoryAgent** (`agents/memory.py`)
   - Purpose: Maintains conversation context and user interaction history
   - Non-AI: Stores interactions, preferences, and session metadata
   - Tracking: Records analysis results, user preferences, and interaction patterns

### Frontend Architecture

**Dual Frontend Strategy:**

1. **Streamlit Application** (`app.py`)
   - Primary UI: Step-by-step workflow interface
   - State Management: Streamlit session state for maintaining context across interactions
   - Layout: Wide responsive layout with custom CSS (Tailwind integration)
   - Workflow: Upload → Analyze → Interact (chat/visualize)

2. **HTML/JavaScript Interface** (`index.html`)
   - Alternative UI: Browser-based interface for Flask backend
   - Visualization: Plotly.js for interactive charts
   - Styling: Tailwind CSS for responsive design
   - Features: Workflow step indicators, real-time status updates

**Design Decisions:**
- Chose Streamlit for rapid prototyping and built-in state management
- Maintained HTML/JS option for more customizable deployments
- Both frontends consume the same agent architecture

### Backend Architecture

**Flask REST API** (`backend.py`)
- Pattern: RESTful API endpoints for agent orchestration
- CORS: Enabled for cross-origin requests (browser-based frontend)
- Session Management: In-memory session_data dictionary (not production-ready)
- Endpoints: `/api/upload` for CSV processing, additional endpoints implied for agent interactions

**Processing Pipeline:**

1. CSV Upload → CSVProcessor (encoding detection, cleaning)
2. Data Analysis → DataIntelligenceAgent (structure + domain detection)
3. Task Planning → PlannerAgent (determine available actions)
4. User Interaction → ExecutorAgent (chat) or VisualizationAgent (charts)
5. Context Storage → MemoryAgent (maintain session history)

### Data Processing Layer

**CSVProcessor** (`utils/csv_processor.py`)
- Multi-encoding support: Tries utf-8, latin1, cp1252, iso-8859-1
- Robust parsing: Multiple separator detection (comma, semicolon)
- Data cleaning: Dataframe sanitization and type inference
- Error handling: Graceful fallback through encoding attempts

**ChartGenerator** (`utils/chart_generator.py`)
- Visualization Library: Plotly for interactive charts
- Supported Charts: Bar, line, scatter, pie, histogram, box plot, heatmap
- Configuration-driven: Accepts chart config objects from VisualizationAgent
- Error resilience: Column validation and fallback to default chart types

### Key Architectural Decisions

**1. Agent-Based Architecture**
- Problem: Complex data analytics workflow requiring multiple specialized capabilities
- Solution: Decompose into specialized agents with single responsibilities
- Rationale: Enables independent development, testing, and AI integration per agent
- Trade-off: Added orchestration complexity vs. modularity and maintainability

**2. Google Gemini API Integration**
- Problem: Need for natural language understanding and domain intelligence
- Solution: Integrate Gemini API for specific agents (DataIntelligence, Visualization, Executor)
- Rationale: Leverage LLM capabilities for semantic understanding without building custom NLP
- Cost Consideration: API calls required for analysis, suggestions, and chat (pay-per-use model)

**3. Dual Frontend Approach**
- Problem: Different deployment scenarios (embedded vs. standalone)
- Solution: Maintain both Streamlit and HTML/JS frontends
- Trade-off: Code duplication vs. deployment flexibility
- Note: Both share the same agent backend logic

**4. In-Memory Session State**
- Current Implementation: session_data dictionary in Flask backend
- Limitation: Not suitable for production (no persistence, single-process only)
- Future Consideration: Would need Redis, database, or persistent session store for scale

**5. Plotly for Visualizations**
- Problem: Need interactive, exportable charts
- Solution: Plotly Express and Graph Objects
- Alternatives Considered: Matplotlib (static), D3.js (complex), Altair (limited interactivity)
- Advantages: Interactive, web-native, supports multiple chart types, JSON serializable

## External Dependencies

### AI Services
- **Google Gemini API**: Core LLM for natural language processing, domain detection, and intelligent suggestions
  - Used by: DataIntelligenceAgent, VisualizationAgent, ExecutorAgent
  - API Key: Configured via environment variable `GEMINI_API_KEY`
  - Library: `google-genai` (version >=1.27.0)

### Python Frameworks
- **Streamlit** (>=1.47.1): Web application framework for primary UI
- **Flask** (>=3.1.1): REST API server for alternative deployment
- **Flask-CORS** (>=6.0.1): Cross-origin resource sharing for browser access

### Data Processing
- **Pandas** (>=2.3.1): Core data manipulation and CSV processing
- **NumPy**: Numerical operations (implicit dependency via pandas)

### Visualization
- **Plotly** (>=6.2.0): Interactive chart generation
  - Frontend: Plotly.js (CDN version 2.26.0)
  - Backend: Plotly Python library

### Frontend Resources (CDN)
- **Tailwind CSS** (2.2.19): UI styling framework
- **Font Awesome** (6.0.0): Icon library
- **Plotly.js** (2.26.0): Client-side chart rendering

### Storage Considerations
- **Current**: No database (in-memory session state only)
- **Future Integration Point**: Application is designed to add persistent storage for:
  - Session management
  - User interaction history
  - Analysis results caching
  - Dashboard configurations
- **Note**: Architecture supports future database integration via MemoryAgent refactoring