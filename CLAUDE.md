# CLAUDE.md (for meta-repo)

This file provides guidance to Claude Code when working across the DogBot multi-repo project.

## Naming Convention

**IMPORTANT**: This project uses two names:
- **WuffChat** - The public-facing brand name. Use this in:
  - User-facing documentation
  - Marketing materials
  - UI text
  - Public communications
  - README files when describing the product
  
- **DogBot** - The internal technical name. Use this in:
  - Repository names
  - Code variables/functions
  - Technical documentation
  - File paths
  - Internal references

## Repository Structure

- `dogbot-agent/` - Backend API service (FastAPI, V2 FSM architecture)
  - Main branch: V2 architecture in production
  - Port: 8000
  - Dependencies: OpenAI, Weaviate, Redis (optional)
  
- `dogbot-ui/` - Frontend application
  - [Add your UI framework/port/key details]
  
- `dogbot-ops/` - Deployment and operations
  - Platform: Scalingo
  - Auto-deploy enabled from GitHub

## Current Architecture Status

- **dogbot-agent**: V2 deployed with clean FSM architecture
  - ValidationService for input validation
  - FlowEngine for state management
  - Separate handlers, agents, and services

## Cross-Repo Development Commands

```bash
# Run all services locally
cd dogbot-agent && python -m uvicorn src.v2.main:app --port 8000 &
cd dogbot-ui && [your UI start command] &

# Run tests across repos
cd dogbot-agent && pytest
cd dogbot-ui && [your UI test command]
```

## Important Context

1. **V2 Architecture**: The agent service just completed a major refactoring from V1 to V2
2. **Production Status**: V2 is live on Scalingo
3. **Recent Issues Fixed**: 
   - Scalingo health check timeouts
   - Error message handling
   - Validation service implementation

## Development Priorities

[Add your current priorities across repos]

## Known Issues

[Add any cross-repo issues or integration points]