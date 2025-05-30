# Error Message Fix Summary

## Changes Made

### 1. Added Specific Error Prompt Types to PromptType Enum
Added three new error prompt types to `src/v2/core/prompt_manager.py`:
- `INPUT_TOO_SHORT = "validation.input.too.short"`
- `NO_BEHAVIOR_MATCH = "validation.no.behavior.match"`
- `INVALID_YES_NO = "validation.invalid.yes.no"`

### 2. Updated Flow Handlers to Use Specific Error Types
Modified `src/v2/core/flow_handlers.py`:

#### handle_symptom_input:
- Changed from `MessageType.INSTRUCTION` with `instruction_type: "describe_more"` to `MessageType.ERROR` with `error_type: "input_too_short"` when input < 10 chars
- Changed from `error_type: "no_match"` to `error_type: "no_behavior_match"` when no Weaviate match found

#### handle_confirmation:
- Changed from `MessageType.INSTRUCTION` with `instruction_type: "be_specific"` to `MessageType.ERROR` with `error_type: "invalid_yes_no"` for invalid yes/no responses

### 3. Updated Dog Agent Error Handler
Modified `src/v2/agents/dog_agent.py` `_handle_error` method to map the new error types:
- `error_type: "no_behavior_match"` → `PromptType.NO_BEHAVIOR_MATCH`
- `error_type: "input_too_short"` → `PromptType.INPUT_TOO_SHORT`
- `error_type: "invalid_yes_no"` → `PromptType.INVALID_YES_NO`

### 4. Updated Tests
- Modified test expectations in `tests/v2/core/test_flow_handlers.py` to expect the new error message types

## Result
Now the system displays specific, user-friendly error messages instead of the generic "Entschuldige, es ist ein..." message:

- **Input too short**: "Das ist etwas kurz. Kannst du mir mehr Details geben?"
- **No behavior match**: "Ich konnte dieses Verhalten nicht einordnen. Magst du es anders beschreiben oder ein anderes Verhalten nennen?"
- **Invalid yes/no**: "Ich verstehe deine Antwort nicht. Bitte antworte mit 'Ja' oder 'Nein'."

All tests are passing and the error handling is now more specific and helpful to users.