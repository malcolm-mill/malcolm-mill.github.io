# Structure of conversations.json

## Overview

The `conversations.json` file contains an **array of conversation objects**. Each conversation object represents an individual chat/conversation.

## Top-level Conversation Properties

Each conversation object has the following key properties:

- **`title`**: The conversation title (e.g., "Exporting Chats to Markdown", "Creep creation code")
- **`create_time`**: Unix timestamp when created
- **`update_time`**: Unix timestamp when last updated
- **`conversation_id`**: Unique identifier for the conversation
- **`conversation_template_id`** and **`gizmo_id`**: These appear to be **project identifiers**
  - Conversations with the same `gizmo_id`/`conversation_template_id` belong to the same project
  - Format: `"g-p-{hash}"` (e.g., `"g-p-693486a7baf48191b65dc12869979547"`)

## Message Structure

Each conversation contains a **`mapping`** object that stores all messages:

- **`mapping`**: A tree-like structure containing all messages in the conversation
  - Each node has an `id`, `message` object, `parent`, and `children` references
  - This creates a tree structure that can represent branching conversations

### Message Contents

Messages within the mapping contain:

- **`author.role`**: The role of the message author
  - `"user"`: User messages
  - `"assistant"`: AI responses
  - `"system"`: System messages
- **`content.parts`**: Array of message content (text, images, attachments, etc.)
- **`create_time`**: Unix timestamp when the message was created
- **`metadata`**: Additional information including:
  - Model used (e.g., `"gpt-5-2"`, `"gpt-5-1"`)
  - Attachments
  - Request IDs
  - Turn exchange information

## Relationship Between Projects and Chats

**Projects are identified by the `gizmo_id` or `conversation_template_id` field.**

- All conversations with the same `gizmo_id`/`conversation_template_id` belong to the same project
- Individual chats are separate conversation objects within the main array
- Chats are grouped together by having matching project IDs

### Organizing by Projects

To organize conversations by projects:

1. **Group conversations** by their `gizmo_id` or `conversation_template_id`
2. Each conversation object is an **individual chat** within that project
3. Conversations without these fields (or with null/empty values) are **standalone chats** not in any project

### Example Structure

```
conversations.json = [
  {
    "title": "Chat 1",
    "conversation_id": "abc-123",
    "gizmo_id": "g-p-project-A",  // ← Project A
    "mapping": { ... }
  },
  {
    "title": "Chat 2",
    "conversation_id": "def-456",
    "gizmo_id": "g-p-project-A",  // ← Project A (same project as Chat 1)
    "mapping": { ... }
  },
  {
    "title": "Chat 3",
    "conversation_id": "ghi-789",
    "gizmo_id": "g-p-project-B",  // ← Project B (different project)
    "mapping": { ... }
  }
]
```

## Key Fields Summary

| Field | Purpose |
|-------|---------|
| `title` | Conversation name/title |
| `conversation_id` | Unique ID for this specific chat |
| `gizmo_id` / `conversation_template_id` | Project identifier (groups related chats) |
| `create_time` / `update_time` | Timestamps for the conversation |
| `mapping` | Tree structure containing all messages |
| `mapping.*.message.author.role` | Who sent the message (user/assistant/system) |
| `mapping.*.message.content.parts` | Actual message content |
