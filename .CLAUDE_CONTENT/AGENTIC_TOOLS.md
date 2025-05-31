# Agentic Tools Design Document

## Context
This document captures the design discussion for making DogBot more conversational and dynamic through the introduction of "agentic tools" - reusable conversation techniques that agents can use to achieve specific communication goals.

## Problem Statement
Current flow is too static and linear:
- After diagnosis → immediate exercise offer → restart/feedback
- No opportunity for follow-up questions
- Binary yes/no choices throughout
- Users can't explore the diagnosis or ask clarifying questions

## Core Concept: Agentic Tools

### Definition
Agentic tools are goal-driven conversation components that:
- Have a specific communication goal (e.g., "ensure user understands diagnosis")
- Use GPT/RAG to dynamically decide what to ask/say
- Are agent-agnostic (return structured data, not formatted messages)
- Can be injected into agents by the orchestrator
- Return control when their goal is achieved

### Key Design Principles
1. **No hardcoded conversation flows** - Tools use GPT to decide next questions dynamically
2. **Agent-agnostic** - Tools return structured data; agents handle persona/formatting
3. **Goal-driven** - Tools work until a natural language goal is achieved
4. **Reusable** - Same tool can be used across different flow states and agents

## Architecture

### Layered Architecture
```
Orchestrator
    ↓ (coordinates)
Flow Engine
    ↓ (manages state)
Agents (format messages, maintain persona)
    ↓ (use tools)
Tools (pure logic, no persona)
    ↓ (query services)
Services (GPT, Weaviate, Redis)
```

### Communication Flow
1. Orchestrator injects tool into agent based on context
2. Agent passes user input to tool
3. Tool returns structured result (no formatting)
4. Agent formats result with appropriate persona
5. When tool achieves goal, control returns to orchestrator

## Tool Design

### Base Tool Structure
```python
class AgenticTool:
    """
    Pure logic tool - returns structured data, not formatted messages.
    """
    
    def __init__(self, goal: str, strategy: str, gpt_service: GPTService):
        self.goal = goal          # Natural language goal
        self.strategy = strategy  # HOW to achieve goal, not persona
        self.gpt_service = gpt_service
        self.state = {}          # Tool's working memory
    
    async def execute(self, user_input: str, context: UserContext) -> ToolResult:
        """
        Returns structured data, not messages.
        """
        # Use GPT to decide next action based on goal and strategy
        
@dataclass
class ToolResult:
    """Structured output from tools."""
    goal_achieved: bool
    action_type: str      # e.g., "probe_understanding", "request_clarification"
    data: Dict[str, Any]  # Tool-specific data
```

### Example Tool: Understanding Confirmation
```python
class UnderstandingConfirmationTool(AgenticTool):
    """
    Agent-agnostic tool for confirming understanding.
    Returns what to verify, not how to say it.
    """
    
    def __init__(self, content: str, key_concepts: List[str], gpt_service: GPTService):
        goal = f"Verify user understands these concepts: {key_concepts}"
        
        strategy = """
        1. Check if user can paraphrase the concept
        2. Verify they can identify triggers/causes
        3. Ensure they see the connection to their specific situation
        4. Confirm readiness to proceed
        """
        
        super().__init__(goal, strategy, gpt_service)
```

## User Context Management

### Current Issue
User information is scattered across session state. For tools to work effectively, we need centralized context.

### Proposed Solution
```python
class UserContext:
    """Centralized user information across conversation."""
    
    def __init__(self, session_id: str):
        self.session_id = session_id
        self.shared_information = {}  # What user tells us
        self.behavior_history = []    # Behaviors discussed
        self.understanding_level = {} # Per-concept tracking
        self.preferences = {}         # Communication style, etc.
    
    def add_information(self, key: str, value: Any, source: str):
        """Track who learned what and when."""
```

## Integration with Current Architecture

### Minimal Changes Required
1. Add `active_tool` field to context passed between components
2. Modify flow handlers to check for active tools before normal processing
3. Agents check for tools and format their output

### Example Integration
```python
# In FlowHandlers
async def handle_context_input(self, session, user_input, context):
    # Check if we have an active tool
    if tool := context.get("active_tool"):
        result = await tool.execute(user_input, context.user_context)
        
        if result.goal_achieved:
            context.pop("active_tool")
            # Proceed to next state
        else:
            # Stay in current state, let tool continue
            
    # Normal flow if no active tool...
```

## Implementation Strategy

### Phase 1: MVP
1. Create base `AgenticTool` class
2. Implement `UnderstandingConfirmationTool` 
3. Modify one flow handler (`handle_context_input`) to test
4. Update `DogAgent` to format tool results

### Phase 2: Expansion
1. Add more tool types (elaboration, examples, quiz)
2. Implement `UserContext` properly
3. Add tool injection points throughout flow

### Phase 3: Advanced
1. Tools that use RAG for context
2. Multi-tool coordination
3. Agent-initiated tool activation

## Open Questions for Future Discussion

1. **Tool Persistence**: Should tools maintain state across multiple conversation turns?
2. **Tool Library**: How to organize and discover available tools?
3. **Goal Specification**: Natural language vs formal verification?
4. **Error Handling**: What if a tool can't achieve its goal?
5. **User Override**: How to handle users who want to skip tool interactions?

## Next Steps

1. Implement base `AgenticTool` class
2. Create `UnderstandingConfirmationTool` as proof of concept
3. Test with single integration point (post-diagnosis)
4. Iterate based on user feedback

## Success Criteria

- Conversations feel more natural and exploratory
- Users can ask follow-up questions after diagnosis
- No hardcoded question sequences
- Tools are truly reusable across different contexts
- Architecture remains clean and testable