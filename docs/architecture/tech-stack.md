# Application Packaging Automation System (APAS) Technology Stack

## Technology Choices

| Category             | Technology              | Version / Details | Description / Purpose                   | Justification |
| :------------------- | :---------------------- | :---------------- | :-------------------------------------- | :----------------------- |
| **Languages**        | Python                  | 3.11+             | Primary language for backend and AI agents | Rich AI/ML ecosystem, excellent library support for AI/ML, good integration with PowerShell |
|                      | TypeScript              | 5.x               | Frontend development and type safety    | Static typing for improved code quality and developer experience |
|                      | PowerShell              | 7.x               | Integration with PSADT and WDAC         | Required for Windows application packaging and policy creation |
| **Runtime**          | Python Runtime          | 3.11+             | Backend and agent execution             | Matches primary language choice |
|                      | Node.js                 | 20.x LTS          | Frontend build and development server   | Required for React ecosystem |
| **Frameworks**       | FastAPI                 | Latest (0.110.0+) | Backend API framework                   | High performance, excellent typing support, async capabilities |
|                      | React                   | 18.x              | Frontend UI library                     | Component-based architecture ideal for visualization needs |
|                      | LangChain               | Latest            | AI agent orchestration                  | Provides ready-made components for agent coordination |
| **Databases**        | Supabase                | Latest            | Unified data platform (PostgreSQL)      | Combines relational DB, vector storage, and file storage in one platform |
|                      | PostgreSQL              | 15+               | Primary relational database (via Supabase) | Robust, reliable database with pgvector extension for embeddings |
| **UI Libraries**     | Tailwind CSS            | 3.x               | Utility-first CSS framework            | Flexible styling with minimal CSS overhead |
|                      | shadcn/ui               | Latest            | UI component library                    | High-quality, customizable components with accessibility support |
|                      | D3.js                   | Latest            | Interactive data visualizations        | Powerful library for custom decision tree visualizations |
|                      | React Flow              | Latest            | Interactive node-based visualizations  | Specialized for node/edge diagrams like decision trees |
| **AI/ML**            | Hugging Face Transformers | Latest          | Model management and inference         | Industry standard for transformer models |
|                      | Sentence Transformers   | Latest            | Text embeddings                        | High-quality embeddings for RAG implementation |
|                      | ONNX Runtime            | Latest            | Optimized model inference              | Cross-platform inference optimization |
| **State Management** | React Query             | Latest            | Server state management                | Efficient data fetching and caching |
|                      | Zustand                 | Latest            | Client state management                | Simple but powerful state management |
| **Testing**          | Pytest                  | Latest            | Backend testing framework              | Comprehensive testing capabilities with good async support |
|                      | Vitest                  | Latest            | Frontend testing framework             | Fast, modern testing for React components |
|                      | Playwright              | Latest            | End-to-end testing                     | Reliable cross-browser testing automation |
| **Dependency Management** | Poetry             | Latest            | Python dependency management           | Modern, reliable dependency management for Python |
|                      | npm/yarn                | Latest            | Node.js dependency management          | Standard tools for JavaScript ecosystem |
| **Documentation**    | MkDocs                  | Latest            | Documentation site generation          | Markdown-based documentation with good theming options |
|                      | Swagger/OpenAPI         | Latest            | API documentation                      | Automatic API documentation from FastAPI |
| **Other Tools**      | PowerShell AppDeploy Toolkit | v3.x/v4.x    | Application packaging framework        | Industry standard for enterprise application packaging |
|                      | Windows Defender Application Control | Latest | Security policy management           | Required for creating and testing application security policies |

## Library Selection Details

### Backend Libraries

1. **FastAPI**
   - Purpose: Web API framework for the backend
   - Advantages: 
     - High performance with async support
     - Automatic API documentation with OpenAPI
     - Built-in data validation and serialization
     - Strong typing and editor support

2. **Supabase SDK**
   - Purpose: Client for Supabase platform (database, auth, storage)
   - Advantages:
     - Unified API for database, storage, and auth
     - Real-time subscriptions capability
     - Native vector operations support
     - Simple setup for both local and cloud deployments

3. **LangChain**
   - Purpose: Framework for building AI agent applications
   - Advantages:
     - Ready-made components for agent orchestration
     - Built-in support for various LLM providers
     - Tools for prompt management and retrieval
     - Active development and community support

4. **Hugging Face Transformers**
   - Purpose: Access to pre-trained models and inference
   - Advantages:
     - Wide range of available models
     - Consistent API across different model architectures
     - Support for quantization and optimization
     - Active development and updates

5. **Sentence Transformers**
   - Purpose: Generate text embeddings for knowledge retrieval
   - Advantages:
     - High-quality semantic embeddings
     - Optimized for retrieval use cases
     - Easy integration with RAG systems
     - Can run locally with reasonable performance

### Frontend Libraries

1. **React**
   - Purpose: Component-based UI development
   - Advantages:
     - Component reusability
     - Virtual DOM for performance
     - Large ecosystem and community
     - Well-established patterns for complex UIs

2. **Tailwind CSS + shadcn/ui**
   - Purpose: Styling and UI components
   - Advantages:
     - Utility-first approach reduces CSS overhead
     - Consistent styling system
     - Accessible, customizable components
     - Modern, clean aesthetic

3. **D3.js**
   - Purpose: Custom data visualizations
   - Advantages:
     - Complete control over visualization details
     - Excellent for interactive visualizations
     - Can handle complex visualization requirements
     - Industry standard for custom visualizations

4. **React Flow**
   - Purpose: Node-based interactive diagrams
   - Advantages:
     - Specialized for node/edge connections
     - Interactive node manipulation
     - Built-in zooming and panning
     - Good fit for decision tree visualization

## AI Model Selection

For the single-machine deployment with 32GB RAM, we've selected the following models:

1. **General Purpose Agent (Orchestrator)**: 
   - Model: LLAMA-2-7B or Mistral-7B (4-bit quantized)
   - Memory Usage: ~4GB
   - Responsibilities: Workflow coordination, task assignment, conflict resolution

2. **Installer Analysis Agent**:
   - Model: MPT-1.3B or FLAN-T5-base
   - Memory Usage: ~1.5-2GB
   - Responsibilities: File analysis, metadata extraction, command detection

3. **PSADT Generation Agent**:
   - Model: CodeLlama-7B (4-bit quantized) or StarCoder-Plus-1B
   - Memory Usage: ~4GB or ~1GB
   - Responsibilities: Script generation, parameter configuration, testing integration

4. **WDAC Policy Agent**:
   - Model: GPT-Neo-1.3B
   - Memory Usage: ~2GB
   - Responsibilities: Policy generation, validation, testing

5. **Embedding Model**:
   - Model: all-MiniLM-L6-v2 or GTE-base
   - Memory Usage: ~100MB
   - Responsibilities: Text embedding for RAG and similarity search

These models will be managed with dynamic loading/unloading to optimize memory usage, with selective fallback to cloud APIs for complex reasoning tasks when needed.

## Local Development Setup

### Requirements
- Python 3.11+
- Node.js 20.x LTS
- Git
- Visual Studio Code (recommended)
- Docker (for local Supabase)
- Windows 10/11 Pro/Enterprise
- Admin rights (for WDAC policy creation/testing)

### Setup Steps
1. Clone repository
2. Install Python dependencies using Poetry
3. Install Node.js dependencies using npm/yarn
4. Start local Supabase instance
5. Initialize database schema
6. Start FastAPI development server
7. Start React development server

## Change Log

| Change        | Date       | Version | Description                   | Author         |
| ------------- | ---------- | ------- | ----------------------------- | -------------- |
| Initial draft | 2025-05-08 | 0.1     | Initial tech stack definition | Architect Agent |
