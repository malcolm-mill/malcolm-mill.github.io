# ChatGPT Projects and Conversations Lister - Documentation

## Overview

This Python script analyzes a ChatGPT data export (`conversations.json`) and organizes all conversations by their associated projects. It provides a clear, hierarchical view of:

- All projects in your ChatGPT account
- Every conversation within each project (sorted chronologically)
- Standalone conversations not assigned to any project
- Summary statistics about your conversation organization

## How It Works

### 1. Windows Unicode Handling (Critical Fix)

The script includes special handling for Unicode characters on Windows systems. ChatGPT conversation titles often contain:
- Emoji characters
- Accented letters (é, ñ, ü, etc.)
- Non-Latin scripts (Chinese, Arabic, Cyrillic, etc.)
- Special symbols

**The Problem:** Windows uses CP-1252 encoding by default for console output, which cannot represent most Unicode characters. When redirecting output to a file, this causes `UnicodeEncodeError`.

**The Solution:** The script reconfigures `sys.stdout` to use UTF-8 encoding on Windows:

```python
if sys.platform == 'win32':
    import io
    sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')
```

This ensures all conversation titles are properly written to the output file, regardless of which characters they contain.

### 2. File Loading

The script begins by loading the `conversations.json` file, which contains an array of all conversation objects. It uses Python's built-in `json` module to parse the file efficiently, even though it's 79 MB in size.

```python
def load_conversations(file_path):
    with open(file_path, 'r', encoding='utf-8') as f:
        conversations = json.load(f)
    return conversations
```

### 3. Project Identification

Each conversation in ChatGPT can belong to a project, identified by two possible fields:
- `gizmo_id`: Primary project identifier
- `conversation_template_id`: Alternative project identifier

The script checks both fields to determine project membership:

```python
project_id = conv.get('gizmo_id') or conv.get('conversation_template_id')
```

**Project ID Format:** `g-p-[hash]` or shorter formats like `g-[hash]`

**Examples from actual data:**
- `g-p-693486a7baf48191b65dc12869979547` (long format)
- `g-2fkFE8rbu` (short format)
- `g-admhPjWQh` (short format)

### 4. Organization Strategy

The script uses a `defaultdict` to group conversations by project:

- **Projects Dictionary:** Maps project IDs to lists of conversations
- **Standalone List:** Contains conversations without a project ID

This dual-structure approach ensures no conversation is lost and provides clear separation between organized and unorganized content.

```python
def organize_by_project(conversations):
    projects = defaultdict(list)
    standalone = []

    for conv in conversations:
        project_id = conv.get('gizmo_id') or conv.get('conversation_template_id')

        if project_id and project_id != 'null':
            projects[project_id].append(conv)
        else:
            standalone.append(conv)

    return dict(projects), standalone
```

### 5. Data Extraction

For each conversation, the script extracts:
- **Title:** The conversation's name/topic
- **Create Time:** UNIX timestamp of when it was created
- **Conversation ID:** Unique identifier
- **Update Time:** Last modification timestamp

These fields are normalized into a consistent format for display.

### 6. Chronological Sorting

Conversations within each project are sorted by creation time (oldest first), providing a timeline view of how each project evolved:

```python
sorted_convs = sorted(convs, key=lambda x: x.get('create_time', 0))
```

### 7. Timestamp Formatting

UNIX timestamps are converted to human-readable dates (YYYY-MM-DD format):

```python
def format_timestamp(timestamp):
    if timestamp:
        return datetime.fromtimestamp(timestamp).strftime('%Y-%m-%d')
    return 'unknown'
```

### 8. Output Generation

The script produces formatted output with:
- Clear section headers
- Project groupings with conversation counts
- Numbered conversation lists with dates
- Visual separators for readability
- Summary statistics at the end

### 9. Statistical Analysis

The script calculates and displays:
- Total conversation count
- Number of distinct projects
- Conversations per project (min, max, average)
- Standalone conversation count

## Usage

### Basic Usage

Run from the same directory as `conversations.json`:

```bash
python list_projects_and_conversations.py
```

### Specify Custom Path

Provide the path to `conversations.json` as an argument:

```bash
python list_projects_and_conversations.py /path/to/conversations.json
```

### Windows Example

```cmd
python list_projects_and_conversations.py conversations.json
```

### Redirect Output to File (Recommended)

Save the output to a text file for later review:

```bash
python list_projects_and_conversations.py > my_projects_list.txt
```

**Windows users must use UTF-8 capable terminal or redirect to file** (the script handles encoding automatically when redirecting).

## Output Format

### Example Output Structure

```
================================================================================
CHATGPT PROJECTS AND CONVERSATIONS
================================================================================

Found 47 projects:

PROJECT 1: g-2fkFE8rbu
  Total conversations: 9

      1. [2024-05-02] Ragazza Italiana Con Camaleonte
      2. [2024-05-02] Cubist Cat & Chameleon.
      3. [2024-05-03] Pretty PDF Mockup
      4. [2024-07-11] Python art deco illustration
      5. [2024-08-22] Cubist Engineer Portrait Request
      6. [2024-08-22] Eiffel Tower Black Hole
      7. [2024-08-28] Green Man Description Request
      8. [2024-09-10] Images for Verb "Go"
      9. [2024-09-18] Amusing Le Marche Image

--------------------------------------------------------------------------------

PROJECT 2: g-5QhhdsfDj
  Total conversations: 1

      1. [2024-05-03] Albero evolution graph

--------------------------------------------------------------------------------

STANDALONE CONVERSATIONS (not in any project): 2002

     1. [2023-08-15] Quick Python question
     2. [2023-09-03] JavaScript async/await
     ...

================================================================================

SUMMARY
================================================================================
Total conversations:        2367
Number of projects:         47
Conversations in projects:  365
Standalone conversations:   2002

Project statistics:
  Largest project:  72 conversations
  Smallest project: 1 conversations
  Average size:     7.8 conversations
================================================================================
```

### Actual Results from Sample Export

Based on a real ChatGPT export with 2,367 conversations:

- **47 projects** identified
- **365 conversations** organized into projects (15.4%)
- **2,002 standalone conversations** (84.6%)
- **Largest project:** 72 conversations
- **Smallest project:** 1 conversation
- **Average project size:** 7.8 conversations

This reveals that most conversations remain unorganized, highlighting potential for better project organization.

## Key Features

### 1. Robust Project Detection
- Checks both `gizmo_id` and `conversation_template_id`
- Handles missing or null project IDs gracefully
- Supports both long and short project ID formats
- Separates standalone conversations clearly

### 2. Unicode Support (Windows Fix)
- **Automatically detects Windows platform**
- **Reconfigures stdout to UTF-8 encoding**
- Prevents `UnicodeEncodeError` crashes
- Handles emoji, accented characters, and international text
- Works correctly when redirecting output to files

### 3. Chronological Organization
- Conversations sorted by creation date within each project
- Easy to see project evolution over time
- Standalone conversations also chronologically sorted

### 4. Readable Output
- Clear visual hierarchy
- Date prefixes for context
- Numbered lists for easy reference
- Statistical summary for overview

### 5. Error Handling
- Checks if input file exists
- Provides helpful error messages
- Gracefully handles missing fields in conversation objects

### 6. Flexible Input
- Accepts command-line argument for file path
- Defaults to current directory if no path provided
- Cross-platform compatible (Windows, Linux, macOS)

## Technical Details

### Dependencies

The script uses only Python standard library modules:
- `json`: Parse conversations.json
- `sys`: Command-line argument handling and platform detection
- `pathlib`: Cross-platform file path handling
- `collections.defaultdict`: Efficient grouping
- `datetime`: Timestamp formatting
- `io`: UTF-8 encoding wrapper (Windows only)

**No external packages required** - runs on any Python 3.6+ installation.

### Performance

- **Memory Usage:** Loads entire JSON file into memory (~79 MB)
- **Processing Time:** Typically 1-3 seconds for 2,367 conversations
- **Output Size:** ~121 KB for 2,367 conversations with 47 projects

### Compatibility

- **Python Version:** Requires Python 3.6 or higher (for f-strings)
- **Operating Systems:** Windows, macOS, Linux
- **Encoding:** UTF-8 support for international characters (automatic on all platforms)
- **Windows Console:** Works correctly when redirecting output (encoding handled automatically)

### Unicode Handling Details

**Why this matters:**
ChatGPT conversations frequently contain:
- Project names like "Ragazza Italiana Con Camaleonte" (Italian with accents)
- Technical symbols and emoji
- Multilingual content

**Without the fix:**
```
UnicodeEncodeError: 'charmap' codec can't encode characters in position 47-48:
character maps to <undefined>
```

**With the fix:**
All characters render correctly in the output file.

**Platform-specific behavior:**
- **Windows:** Uses `io.TextIOWrapper` to force UTF-8
- **Linux/macOS:** Uses system default UTF-8 (no modification needed)

## Troubleshooting

### UnicodeEncodeError

**If you see this error on older versions of the script:**
```
UnicodeEncodeError: 'charmap' codec can't encode character
```

**Solution:** The current version includes the Windows UTF-8 fix. Ensure you're using the latest version of the script.

**Manual workaround (if needed):**
```bash
# Set environment variable before running (PowerShell)
$env:PYTHONIOENCODING = "utf-8"
python list_projects_and_conversations.py > output.txt

# Or use chcp to change console code page (CMD)
chcp 65001
python list_projects_and_conversations.py > output.txt
```

### File Not Found

If the script can't find `conversations.json`:
```
Error: File not found: conversations.json
```

**Solution:** Either:
1. Run from the directory containing `conversations.json`
2. Provide the full path: `python script.py "C:\path\to\conversations.json"`

### Memory Issues

For very large exports (10,000+ conversations):
- The script loads the entire file into memory
- May require 500+ MB RAM
- Consider processing in chunks if memory is limited

## Customization Options

### Modify Sorting Order

Change from oldest-first to newest-first:

```python
sorted_convs = sorted(convs, key=lambda x: x.get('create_time', 0), reverse=True)
```

### Change Date Format

Use different date formatting:

```python
# Include time: 2024-12-05 14:30
return datetime.fromtimestamp(timestamp).strftime('%Y-%m-%d %H:%M')

# Month name: Dec 05, 2024
return datetime.fromtimestamp(timestamp).strftime('%b %d, %Y')
```

### Export to CSV

Add CSV export functionality:

```python
import csv

def export_to_csv(projects, standalone, output_file='conversations.csv'):
    with open(output_file, 'w', newline='', encoding='utf-8') as f:
        writer = csv.writer(f)
        writer.writerow(['Project ID', 'Date', 'Title', 'Conversation ID'])

        for project_id, convs in projects.items():
            for conv in convs:
                info = get_conversation_info(conv)
                writer.writerow([
                    project_id,
                    format_timestamp(info['create_time']),
                    info['title'],
                    info['conversation_id']
                ])
```

### Filter by Date Range

Add date filtering:

```python
def filter_by_date(conversations, start_date, end_date):
    filtered = []
    for conv in conversations:
        create_time = conv.get('create_time', 0)
        conv_date = datetime.fromtimestamp(create_time)
        if start_date <= conv_date <= end_date:
            filtered.append(conv)
    return filtered
```

## Use Cases

### 1. Project Audit
Quickly see which projects you have and how many conversations are in each.

**Example insight:** "Most of my conversations are standalone - I should organize them into projects."

### 2. Conversation Organization
Identify standalone conversations that should be moved into projects.

**Example insight:** "I have 2,002 unorganized conversations that could be categorized."

### 3. Data Analysis
Get statistics on your ChatGPT usage patterns and project distribution.

**Example insight:** "My largest project has 72 conversations about Screeps game development."

### 4. Export Planning
Determine which projects to export or migrate to other platforms.

**Example insight:** "I have 9 image generation projects I could archive separately."

### 5. Cleanup Planning
Find old or unused projects that can be archived or deleted.

**Example insight:** "Several projects have only 1 conversation and could be merged."

## Limitations

### 1. Project Names Not Included
The export doesn't include human-readable project names, only IDs. You'll need to cross-reference with your ChatGPT UI to identify which ID corresponds to which project name.

**Workaround:** Keep a separate mapping file or manually annotate the output.

### 2. Archived Conversations
The script doesn't distinguish between archived and active conversations (though this data is available in the `is_archived` field).

**Potential enhancement:** Add filtering for archived status.

### 3. Memory Requirements
The entire conversations.json file is loaded into memory. For very large exports (>1 GB), this might be problematic on systems with limited RAM.

**Workaround:** Process in batches if needed.

### 4. Project Name Mapping
Project IDs are cryptic (e.g., `g-2fkFE8rbu`). The export doesn't include the friendly names you see in ChatGPT's UI.

**Workaround:** Manually create a mapping based on conversation topics.

## Future Enhancements

Potential improvements to consider:

1. **JSON Output:** Export results as JSON for programmatic use
2. **HTML Report:** Generate an interactive HTML page with search and filtering
3. **Project Name Mapping:** Allow manual mapping of project IDs to friendly names via config file
4. **Date Filtering:** Add command-line options to filter by date range
5. **Search Functionality:** Find conversations containing specific keywords
6. **Export Options:** Save individual projects to separate files
7. **Markdown Output:** Generate markdown with links and formatting
8. **Statistics Dashboard:** More detailed analytics (conversations per month, model usage, etc.)

---

## Complete Script Source Code

```python
#!/usr/bin/env python3
"""
ChatGPT Project and Conversation Lister

This script reads the conversations.json file from a ChatGPT data export
and organizes conversations by project, listing all project names and
their associated conversation titles.

Usage:
    python list_projects_and_conversations.py [path_to_conversations.json]

If no path is provided, it looks for conversations.json in the current directory.
"""

import json
import sys
from pathlib import Path
from collections import defaultdict
from datetime import datetime


def load_conversations(file_path):
    """
    Load and parse the conversations.json file.

    Args:
        file_path (str): Path to conversations.json

    Returns:
        list: List of conversation objects
    """
    print(f"Loading conversations from: {file_path}")
    with open(file_path, 'r', encoding='utf-8') as f:
        conversations = json.load(f)
    print(f"Loaded {len(conversations)} conversations\n")
    return conversations


def organize_by_project(conversations):
    """
    Organize conversations into projects based on gizmo_id.

    Args:
        conversations (list): List of conversation objects

    Returns:
        tuple: (projects_dict, standalone_conversations)
            - projects_dict: Dictionary mapping project IDs to conversation lists
            - standalone_conversations: List of conversations not in any project
    """
    projects = defaultdict(list)
    standalone = []

    for conv in conversations:
        # Try to get project ID from either gizmo_id or conversation_template_id
        project_id = conv.get('gizmo_id') or conv.get('conversation_template_id')

        if project_id and project_id != 'null':
            projects[project_id].append(conv)
        else:
            standalone.append(conv)

    return dict(projects), standalone


def get_conversation_info(conv):
    """
    Extract relevant information from a conversation object.

    Args:
        conv (dict): Conversation object

    Returns:
        dict: Dictionary with title, create_time, and conversation_id
    """
    return {
        'title': conv.get('title', 'Untitled'),
        'create_time': conv.get('create_time', 0),
        'conversation_id': conv.get('conversation_id', 'unknown'),
        'update_time': conv.get('update_time', 0)
    }


def format_timestamp(timestamp):
    """
    Convert UNIX timestamp to readable date string.

    Args:
        timestamp (float): UNIX timestamp

    Returns:
        str: Formatted date string (YYYY-MM-DD)
    """
    if timestamp:
        return datetime.fromtimestamp(timestamp).strftime('%Y-%m-%d')
    return 'unknown'


def print_projects_and_conversations(projects, standalone):
    """
    Print organized project and conversation information.

    Args:
        projects (dict): Dictionary of projects and their conversations
        standalone (list): List of standalone conversations
    """
    print("=" * 80)
    print("CHATGPT PROJECTS AND CONVERSATIONS")
    print("=" * 80)
    print()

    # Print projects
    if projects:
        print(f"Found {len(projects)} projects:\n")

        for idx, (project_id, convs) in enumerate(sorted(projects.items()), 1):
            print(f"PROJECT {idx}: {project_id}")
            print(f"  Total conversations: {len(convs)}")
            print()

            # Sort conversations by creation time
            sorted_convs = sorted(convs, key=lambda x: x.get('create_time', 0))

            for i, conv in enumerate(sorted_convs, 1):
                info = get_conversation_info(conv)
                date = format_timestamp(info['create_time'])
                print(f"    {i:3d}. [{date}] {info['title']}")

            print()
            print("-" * 80)
            print()

    # Print standalone conversations
    if standalone:
        print(f"STANDALONE CONVERSATIONS (not in any project): {len(standalone)}")
        print()

        # Sort by creation time
        sorted_standalone = sorted(standalone, key=lambda x: x.get('create_time', 0))

        for i, conv in enumerate(sorted_standalone, 1):
            info = get_conversation_info(conv)
            date = format_timestamp(info['create_time'])
            print(f"  {i:4d}. [{date}] {info['title']}")

        print()
        print("=" * 80)


def generate_summary(projects, standalone, total_conversations):
    """
    Print summary statistics.

    Args:
        projects (dict): Dictionary of projects
        standalone (list): List of standalone conversations
        total_conversations (int): Total number of conversations
    """
    print("\nSUMMARY")
    print("=" * 80)
    print(f"Total conversations:        {total_conversations}")
    print(f"Number of projects:         {len(projects)}")
    print(f"Conversations in projects:  {total_conversations - len(standalone)}")
    print(f"Standalone conversations:   {len(standalone)}")

    if projects:
        project_sizes = [len(convs) for convs in projects.values()]
        print(f"\nProject statistics:")
        print(f"  Largest project:  {max(project_sizes)} conversations")
        print(f"  Smallest project: {min(project_sizes)} conversations")
        print(f"  Average size:     {sum(project_sizes) / len(project_sizes):.1f} conversations")

    print("=" * 80)


def main():
    """
    Main function to orchestrate the script execution.
    """
    # Set UTF-8 encoding for stdout to handle Unicode characters on Windows
    if sys.platform == 'win32':
        import io
        sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')

    # Determine input file path
    if len(sys.argv) > 1:
        input_file = Path(sys.argv[1])
    else:
        input_file = Path('conversations.json')

    # Check if file exists
    if not input_file.exists():
        print(f"Error: File not found: {input_file}")
        print(f"\nUsage: python {sys.argv[0]} [path_to_conversations.json]")
        sys.exit(1)

    # Load conversations
    conversations = load_conversations(input_file)

    # Organize by project
    projects, standalone = organize_by_project(conversations)

    # Print results
    print_projects_and_conversations(projects, standalone)

    # Print summary
    generate_summary(projects, standalone, len(conversations))


if __name__ == '__main__':
    main()
```

---

## Quick Reference

**Script Location:** `_OUTPUT/list_projects_and_conversations.py`

**Run Command:**
```bash
python list_projects_and_conversations.py
```

**Save Output:**
```bash
python list_projects_and_conversations.py > projects_list.txt
```

**With Custom Path:**
```bash
python list_projects_and_conversations.py "path/to/conversations.json" > output.txt
```

**Dependencies:** None (Python 3.6+ standard library only)

**Input:** `conversations.json` (ChatGPT data export)

**Output:** Formatted text listing of all projects and conversations with UTF-8 encoding

**Special Features:**
- ✅ Automatic Windows UTF-8 encoding fix
- ✅ Handles emoji and international characters
- ✅ Cross-platform compatible
- ✅ Zero external dependencies
