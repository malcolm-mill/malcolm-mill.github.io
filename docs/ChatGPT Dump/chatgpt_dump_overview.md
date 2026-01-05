# ChatGPT Data Export - Complete Overview

## Summary

This is a complete data export from ChatGPT containing **2,367 conversations** spanning your entire ChatGPT usage history. The export includes all conversation data, uploaded files, DALL-E generated images, and metadata.

**Export Date:** January 5, 2026
**Total Size:** ~208 MB
**Primary Data Format:** JSON

---

## Directory Structure

```
chatgpt-export/
├── conversations.json          (79 MB) - All conversation data
├── chat.html                   (81 MB) - Human-readable HTML view
├── user.json                   - User account information
├── message_feedback.json       - Message rating history
├── shopping.json               - Shopping-related data (empty)
├── group_chats.json           - Group chat data (empty)
├── dalle-generations/         - AI-generated images (159 files)
├── user-[REDACTED]/           - User-uploaded files (31 files)
└── [305 asset files]          - Screenshots, images, and attachments
```

---

## Core Data Files

### 1. conversations.json (79 MB)
**Primary conversation archive** - Contains all 2,367 conversations in structured JSON format.

**Key characteristics:**
- Single massive JSON array of conversation objects
- Each conversation is a complete, self-contained record
- No line breaks (single-line file for efficiency)
- Size: 82,085,205 bytes

**Structure:** See [conversations_structure.md](conversations_structure.md) for detailed breakdown.

**Each conversation contains:**
- Conversation metadata (title, timestamps, IDs)
- Project association (via `gizmo_id` / `conversation_template_id`)
- Complete message tree with branching support
- User and assistant messages with full content
- System messages and metadata
- Attachments and file references
- Model information (GPT-4, GPT-5, etc.)

### 2. chat.html (81 MB)
**Human-readable archive** - Self-contained HTML file with all conversations.

**Purpose:**
- Browsable interface for viewing conversations
- Includes embedded CSS and JavaScript
- References asset files for images/attachments
- Useful for quick searching and reading without programming

### 3. user.json
**Account information** - Your ChatGPT user profile.

**Contains:**
- User ID: `user-[REDACTED]`
- Email address
- ChatGPT Plus subscription status
- Birth year and phone number

### 4. message_feedback.json
**Rating history** - Records of thumbs up/down feedback given to messages.

**Contains:**
- 1 feedback entry
- Conversation references
- Rating type (thumbs_up/thumbs_down)
- Timestamps
- Evaluation metadata

### 5. shopping.json
**Shopping data** - Currently empty array `[]`.

**Likely purpose:**
- Future feature for ChatGPT shopping integrations
- Placeholder for commerce-related interactions

### 6. group_chats.json
**Group conversations** - Currently contains `{"chats": []}`.

**Likely purpose:**
- Multi-user chat sessions
- Collaborative conversations
- Team workspace chats

---

## Asset Files

### Root Directory Assets (305 files)
**Naming pattern:** `file-[hash]-[uuid].[ext]` or `file_[hash]-sanitized.[ext]`

**File types:**
- PNG images (majority)
- JPG/JPEG images
- WebP images
- One JPEG file

**Purpose:**
- Screenshots uploaded during conversations
- Diagrams and charts
- Code snippets as images
- Reference materials

**Sanitization:** Files with `-sanitized` suffix have been processed for privacy/security.

### dalle-generations/ (159 files)
**AI-generated images** created using DALL-E during conversations.

**Naming pattern:** `file-[OpenAI-ID]-[UUID].webp`

**Characteristics:**
- All WebP format (efficient compression)
- Associated with specific conversations
- Generated artwork, diagrams, visualizations
- Referenced in conversation message metadata

### user-[REDACTED]/ (31 files)
**User-uploaded files** with version tracking.

**Naming pattern:** `file_[hash]-[UUID].png`

**Purpose:**
- Files you uploaded across multiple conversations
- Likely duplicates or versions of images also in root
- Organized by user ID for multi-user scenarios

---

## Logical Organization

### Project-Based Grouping

Conversations are organized into **projects** using these identifiers:
- `gizmo_id`
- `conversation_template_id`

**Project ID format:** `g-p-[hash]`

**Examples found:**
- `g-p-693486a7baf48191b65dc12869979547` (appears frequently)
- `g-p-6950233eed8881919261474e2e0866f9`
- `g-p-68f73a90168c81918d53773c1323ed0c`

**How it works:**
- All conversations with the same project ID belong to the same ChatGPT Project
- Projects group related conversations by topic/purpose
- Conversations without project IDs are standalone chats
- This allows organizing 2,367 conversations into manageable topical groups

### Conversation Chronology

**Date range:** Conversations span multiple years (timestamps in UNIX epoch format)

**Ordering:**
- Conversations in the JSON array have no guaranteed order
- Each conversation has `create_time` and `update_time` timestamps
- Use timestamps to reconstruct chronological history

### Message Threading

**Branching structure:**
- Messages form a tree, not a linear list
- Each message node has `parent` and `children` references
- Supports conversation branching (when you edit a prompt and regenerate)
- Root nodes start with `client-created-root`

**Navigation:**
- `current_node` field indicates the active conversation path
- System messages are often visually hidden
- Assistant messages may include thinking/reasoning steps

---

## Data Characteristics

### File Sizes

| File | Size | Percentage |
|------|------|------------|
| conversations.json | 79 MB | 38% |
| chat.html | 81 MB | 39% |
| Asset files (305) | ~40 MB | 19% |
| DALL-E images (159) | ~6 MB | 3% |
| User files (31) | ~2 MB | 1% |
| **Total** | **~208 MB** | **100%** |

### Conversation Statistics

- **Total conversations:** 2,367
- **Format:** JSON (machine-readable) + HTML (human-readable)
- **Message types:** User, Assistant, System
- **Models used:** GPT-4, GPT-5-1, GPT-5-2, GPT-4.5 (various versions)
- **Special features:** Thinking/reasoning steps, code execution, file analysis, DALL-E generation

### Asset Statistics

- **Total asset files:** 495
  - Root directory: 305 images
  - DALL-E generated: 159 images
  - User-uploaded: 31 files
- **Formats:** PNG, JPG, WebP, JPEG
- **Purpose:** Visual context for conversations

---

## Data Relationships

### How Conversations Reference Assets

**In conversations.json:**
- Messages contain `content.parts` with `content_type: "image_asset_pointer"`
- Asset pointers use format: `sediment://file_[hash]`
- Hash matches the filename in root directory (without `-sanitized` suffix)
- DALL-E images referenced similarly with their unique IDs

**Example flow:**
1. User uploads image `screenshot.png`
2. ChatGPT stores as `file_00000000abc123-sanitized.png`
3. Conversation message references `sediment://file_00000000abc123`
4. HTML file embeds image using relative path

### How Projects Organize Conversations

**Grouping logic:**
1. Each conversation has a `gizmo_id` or `conversation_template_id`
2. Conversations with matching IDs belong to same project
3. Projects have implicit names (visible in ChatGPT UI, not in export)
4. You can reconstruct project structure by grouping on these IDs

**Example reconstruction:**
```python
projects = {}
for conv in conversations:
    project_id = conv.get('gizmo_id')
    if project_id:
        projects.setdefault(project_id, []).append(conv)
```

### Timeline Reconstruction

**Chronological ordering:**
- Sort conversations by `create_time` for creation order
- Sort conversations by `update_time` for last-modified order
- Within each conversation, traverse message tree by `create_time`
- Message threading allows multiple timeline branches per conversation

---

## Use Cases

### 1. Search and Analysis
- Full-text search across all conversations
- Identify patterns in questions asked
- Extract code snippets or solutions
- Build personal knowledge base

### 2. Project-Based Organization
- Group conversations by `gizmo_id` to reconstruct projects
- Export specific project conversations to dedicated folders
- Create project indexes or documentation

### 3. Conversion and Export
- Convert conversations to Markdown files
- Generate documentation from technical discussions
- Create training datasets
- Build Obsidian/Notion/MkDocs knowledge bases

### 4. Data Migration
- Import into other platforms
- Archive for long-term storage
- Backup before deleting old conversations
- Transfer knowledge between accounts

### 5. Analytics
- Conversation length distribution
- Most active time periods
- Model usage statistics
- Topic clustering

---

## Technical Notes

### JSON Structure Peculiarities

**Single-line format:**
- No line breaks in `conversations.json` despite 79 MB size
- Standard `wc -l` returns 0
- Optimized for parsing, not human reading
- Use `jq` or JSON parsers, not text tools

**Character encoding:**
- UTF-8 with Unicode support
- Emoji and special characters preserved
- Code blocks maintain formatting

**File identification:**
- Detected as "Nim source code" by `file` command (false positive)
- Actually valid JSON despite detection quirk

### Asset File Naming

**Two naming schemes:**

1. **Legacy format:** `file_[hash]-sanitized.ext`
   - Older upload system
   - Always includes `-sanitized` suffix
   - Hash is hexadecimal

2. **Modern format:** `file-[OpenAI-ID]-[UUID].ext`
   - Newer upload system
   - OpenAI's internal ID format
   - UUID for uniqueness

**Sanitization:**
- Removes metadata (EXIF, GPS, etc.)
- Strips potentially sensitive information
- Maintains visual content

### HTML Rendering

**chat.html features:**
- Self-contained (CSS/JS embedded)
- Loads assets via relative paths
- Searchable interface
- Copy-paste functionality
- Dark mode support (likely)

**Limitations:**
- Large file size (81 MB) may slow browsers
- No server required (static HTML)
- Asset references may break if files moved

---

## Privacy and Security

### Personal Information

**Included in export:**
- Email address (in user.json)
- Phone number (in user.json)
- Birth year (in user.json)
- All conversation history (including private discussions)
- File uploads (potentially containing sensitive data)

**Recommendations:**
- Treat this export as highly confidential
- Sanitize before sharing or publishing
- Review uploaded files for sensitive content
- Check conversations for private information

### File Sanitization

**What's sanitized:**
- Image metadata (EXIF, location data)
- Embedded information in uploads

**What's NOT sanitized:**
- Conversation text content
- User-written code or data
- File contents (only metadata removed)
- Personal details in messages

---

## Working with This Export

### Reading Conversations

**Option 1: HTML (easiest)**
- Open `chat.html` in a web browser
- Use browser search (Ctrl+F / Cmd+F)
- Navigate by clicking conversations

**Option 2: JSON (programmable)**
```python
import json

with open('conversations.json', 'r', encoding='utf-8') as f:
    conversations = json.load(f)

# Example: Find conversations about a topic
for conv in conversations:
    if 'Python' in conv['title']:
        print(conv['title'], conv['create_time'])
```

**Option 3: Command-line**
```bash
# Count conversations
jq 'length' conversations.json

# Find by title
jq '.[] | select(.title | contains("Python"))' conversations.json

# Extract all titles
jq '.[].title' conversations.json
```

### Extracting Specific Data

**Get all conversation titles:**
```bash
jq -r '.[].title' conversations.json > titles.txt
```

**List all unique project IDs:**
```bash
jq -r '.[].gizmo_id' conversations.json | sort -u
```

**Find conversations in a specific project:**
```bash
jq '.[] | select(.gizmo_id == "g-p-693486a7baf48191b65dc12869979547")' conversations.json
```

**Extract creation dates:**
```python
import json
from datetime import datetime

with open('conversations.json', 'r') as f:
    data = json.load(f)

for conv in data:
    timestamp = conv['create_time']
    date = datetime.fromtimestamp(timestamp)
    print(f"{date.strftime('%Y-%m-%d')}: {conv['title']}")
```

### Converting to Other Formats

See accompanying documentation or scripts for:
- Markdown conversion (individual files per conversation)
- Project-based folder organization
- Obsidian vault creation
- MkDocs documentation site generation
- CSV export of conversation metadata

---

## Appendix: Field Reference

### Conversation Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `title` | string | Conversation title |
| `conversation_id` | string | Unique conversation identifier |
| `gizmo_id` | string | Project ID (if in a project) |
| `conversation_template_id` | string | Alternative project identifier |
| `create_time` | float | UNIX timestamp of creation |
| `update_time` | float | UNIX timestamp of last update |
| `mapping` | object | Message tree structure |
| `current_node` | string | Active message path ID |
| `is_archived` | boolean | Archive status |
| `default_model_slug` | string | Model used (e.g., "gpt-5-2") |
| `safe_urls` | array | Whitelisted URLs |
| `blocked_urls` | array | Blocked URLs |

### Message Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Message unique identifier |
| `author.role` | string | "user", "assistant", or "system" |
| `content.content_type` | string | "text", "multimodal_text", etc. |
| `content.parts` | array | Message content segments |
| `create_time` | float | UNIX timestamp |
| `status` | string | "finished_successfully", etc. |
| `metadata` | object | Additional information |
| `parent` | string | Parent message ID |
| `children` | array | Child message IDs |

### Asset Pointer Format

```json
{
  "content_type": "image_asset_pointer",
  "asset_pointer": "sediment://file_00000000abc123",
  "size_bytes": 25524,
  "width": 632,
  "height": 258,
  "metadata": {
    "sanitized": true
  }
}
```

---

## Summary

This ChatGPT data export is a comprehensive archive containing:

- **2,367 conversations** with complete message history
- **495 asset files** (images, uploads, DALL-E generations)
- **Project organization** via gizmo IDs
- **Metadata** including user info and feedback
- **Multiple formats** (JSON for processing, HTML for browsing)

The data is **well-structured** for programmatic access while remaining **human-readable** through the HTML interface. The export provides complete fidelity to your ChatGPT history, enabling archival, analysis, migration, and knowledge base creation.
