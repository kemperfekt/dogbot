# 🐕 WuffChat (DogBot) - Understanding Your Dog's Perspective

<div align="center">
  <img src="https://github.com/kemperfekt/dogbot-ui/blob/main/src/assets/hund_icon.png" alt="WuffChat Logo" width="200"/>
  
  [![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)](https://fastapi.tiangolo.com)
  [![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://reactjs.org/)
  [![OpenAI](https://img.shields.io/badge/OpenAI-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com)
  [![Weaviate](https://img.shields.io/badge/Weaviate-2C2C2C?style=for-the-badge&logo=weaviate)](https://weaviate.io/)
  
  **[🌐 Website](https://wuffchat.de) | [🎯 App](https://app.wuffchat.de) | [🔧 API](https://api.wuffchat.de)**
</div>

---

## 🎯 What is WuffChat?

**WuffChat** is an AI-powered conversational assistant that helps dog owners understand their furry friends by **explaining behavior from the dog's perspective**. Using advanced AI and a comprehensive knowledge base about canine instincts, WuffChat provides empathetic, instinct-based behavioral analysis.

> **Technical Note**: This project is internally referred to as "DogBot" in repositories and code. WuffChat is the public-facing brand name.

```mermaid
graph LR
    A[🧑 Dog Owner] -->|Describes Behavior| B[💬 WuffChat]
    B -->|Analyzes| C[🧠 AI Engine]
    C -->|Searches| D[📚 Knowledge Base]
    D -->|Returns| E[🐕 Dog's Perspective]
    E -->|Provides| F[💡 Training Tips]
    F -->|Helps| A
```

## 🏗️ Architecture Overview

```mermaid
graph TB
    subgraph "Frontend"
        UI[dogbot-ui<br/>React App]
    end
    
    subgraph "Backend"
        API[dogbot-agent<br/>FastAPI]
        FSM[FSM Engine<br/>11 States]
        AI[GPT-4<br/>Analysis]
    end
    
    subgraph "Data Layer"
        VDB[(Weaviate<br/>Vector DB)]
        CACHE[(Redis<br/>Cache)]
    end
    
    subgraph "Infrastructure"
        OPS[dogbot-ops<br/>Data Management]
        WWW[dogbot-www<br/>Landing Page]
    end
    
    UI <-->|REST API| API
    API --> FSM
    FSM --> AI
    API <--> VDB
    API <--> CACHE
    OPS -->|Manages| VDB
    
    style UI fill:#61DAFB,color:#000
    style API fill:#009688,color:#fff
    style AI fill:#412991,color:#fff
    style VDB fill:#2C2C2C,color:#fff
```

## 🚀 Quick Start

### Prerequisites
- Python 3.11+
- Node.js 18+
- Docker (optional, for Weaviate)
- OpenAI API key

### 🏃‍♂️ Run Everything Locally

```bash
# Clone the meta repository
git clone https://github.com/kemperfekt/dogbot.git
cd dogbot

# Start the backend
cd dogbot-agent
pip install -r requirements.txt
export OPENAI_API_KEY=your_key_here
uvicorn src.v2.main:app --port 8000

# In a new terminal, start the frontend
cd ../dogbot-ui
npm install
npm start

# Optional: Setup local Weaviate with data
cd ../dogbot-ops
python setup_dogbot_weaviate.py
```

Visit `http://localhost:3000` to start chatting! 🎉

## 📦 Repository Structure

| Repository | Purpose | Tech Stack | Status |
|------------|---------|------------|--------|
| **[dogbot-agent](https://github.com/kemperfekt/dogbot-agent)** | Backend API & AI Logic | FastAPI, GPT-4, Weaviate | ✅ Production |
| **[dogbot-ui](https://github.com/kemperfekt/dogbot-ui)** | Chat Interface | React, Tailwind CSS | ✅ Production |
| **[dogbot-ops](https://github.com/kemperfekt/dogbot-ops)** | Data & Schema Management | Python, Content-as-Code | ✅ Active |
| **[dogbot-www](https://github.com/kemperfekt/dogbot-www)** | Landing Page | Static HTML, Tailwind | ✅ Live |

## 🧠 How It Works

### Conversation Flow

```mermaid
stateDiagram-v2
    [*] --> START
    START --> GREETING: User starts
    GREETING --> CONTEXT_GATHERING: Collects info
    CONTEXT_GATHERING --> INITIAL_ASSESSMENT: Analyzes behavior
    INITIAL_ASSESSMENT --> INSTINCT_DIAGNOSIS: Identifies instincts
    INSTINCT_DIAGNOSIS --> DETAILED_SOLUTION: Provides tips
    DETAILED_SOLUTION --> FEEDBACK: Asks for feedback
    FEEDBACK --> END: Completes
    
    note right of INITIAL_ASSESSMENT
        AI provides response from
        the dog's perspective
    end note
    
    note right of INSTINCT_DIAGNOSIS
        Maps behavior to core
        canine instincts
    end note
```

### Core Instincts Model

WuffChat analyzes behavior through four fundamental canine instincts:

```mermaid
mindmap
  root((Dog Behavior))
    Jagdinstinkt
      Prey Drive
      Chase Behavior
      Resource Guarding
    Territorialinstinkt
      Space Protection
      Boundary Setting
      Alert Behavior
    Rudelinstinkt
      Pack Dynamics
      Social Hierarchy
      Cooperation
    Sexualinstinkt
      Mating Behavior
      Competition
      Hormonal Changes
```

## 🔧 Development

### Local Development Setup

Each repository has its own development environment:

```bash
# Backend development
cd dogbot-agent
python -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows
pip install -r requirements.txt
pytest  # Run tests

# Frontend development
cd dogbot-ui
npm install
npm test
npm run build

# Data management
cd dogbot-ops
python setup_dogbot_weaviate.py --help
```

### Environment Variables

Create `.env` files in each repository:

**dogbot-agent/.env**
```env
OPENAI_API_KEY=sk-...
WEAVIATE_URL=http://localhost:8080
REDIS_URL=redis://localhost:6379
ENVIRONMENT=development
```

**dogbot-ui/.env**
```env
REACT_APP_API_URL=http://localhost:8000
```

## 🚢 Deployment

WuffChat is deployed on [Scalingo](https://scalingo.com) with automatic deployments from GitHub:

```mermaid
graph LR
    A[GitHub Push] -->|Webhook| B[Scalingo Build]
    B -->|Deploy| C[Production]
    C --> D[app.wuffchat.de]
    C --> E[api.wuffchat.de]
    C --> F[wuffchat.de]
```

### Manual Deployment

```bash
# Deploy backend
cd dogbot-agent
git push scalingo main

# Deploy frontend
cd dogbot-ui
git push scalingo main
```

## 📊 API Documentation

The API is fully documented with OpenAPI/Swagger:

- **Local**: http://localhost:8000/docs
- **Production**: https://api.wuffchat.de/docs

### Key Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Health check |
| `/start-conversation` | POST | Begin new chat |
| `/send-message` | POST | Send message |
| `/feedback` | POST | Submit feedback |

## 🧪 Testing

```bash
# Run all tests
cd dogbot-agent && pytest
cd ../dogbot-ui && npm test

# Run specific test suites
pytest tests/test_flow_engine.py
npm test -- --coverage
```

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](./CONTRIBUTING.md) for details.

### Development Workflow

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 🛡️ Security & Privacy

- All conversations are anonymous
- No personal data is stored without consent
- OpenAI API calls use anonymized prompts
- See our [Privacy Policy](https://wuffchat.de/datenschutz.html)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.


## 📝 Naming Convention

- **WuffChat**: Public-facing product name (used in marketing, UI, and customer communication)
- **DogBot**: Internal technical name (used in code, repositories, and technical documentation)

This dual naming allows us to maintain technical consistency while presenting a friendly brand to users.

---

<div align="center">
  
  [Report Bug](https://github.com/kemperfekt/dogbot/issues) · [Request Feature](https://github.com/kemperfekt/dogbot/issues)
  
</div>