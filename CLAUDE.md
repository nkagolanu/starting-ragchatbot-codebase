# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start (recommended)
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install dependencies
uv sync

# Run Python commands in the environment
uv run <command>
```

### Environment Setup
- Requires `.env` file in root with `ANTHROPIC_API_KEY`
- ChromaDB data stored in `./backend/chroma_db/`
- Course documents should be placed in `./docs/` folder

## Architecture Overview

### Core RAG Flow
The system implements a tool-based RAG architecture where the AI decides when to search:

1. **Query Processing**: `rag_system.py` orchestrates the entire flow
2. **AI Decision Making**: `ai_generator.py` uses Anthropic Claude with tool calling to decide whether to search
3. **Tool-Based Search**: `search_tools.py` provides semantic search capabilities to the AI
4. **Vector Storage**: `vector_store.py` manages ChromaDB collections for course metadata and content chunks

### Key Architectural Patterns

#### Dual Collection Strategy
- `course_catalog`: Stores course metadata for semantic course name resolution
- `course_content`: Stores chunked course content for detailed search

#### Tool-Based AI Interaction
The AI has access to a `search_course_content` tool that can:
- Search across all courses or filter by specific course name
- Filter by lesson number
- Perform semantic course name matching (e.g., "MCP" matches "MCP: Build Rich-Context AI Apps")

#### Session Management
- Conversation history maintained per session
- Limited to `MAX_HISTORY` exchanges to prevent context overflow
- Session IDs generated automatically for new conversations

### Data Models (`models.py`)
- `Course`: Contains title, instructor, lessons, and metadata
- `Lesson`: Individual lesson with number, title, and optional link
- `CourseChunk`: Text chunks with course/lesson attribution for vector storage

### Configuration (`config.py`)
Key settings that affect system behavior:
- `CHUNK_SIZE: 800` - Size of text chunks for vector storage
- `CHUNK_OVERLAP: 100` - Character overlap between chunks
- `MAX_RESULTS: 5` - Maximum search results returned
- `MAX_HISTORY: 2` - Conversation exchanges to remember
- `ANTHROPIC_MODEL: "claude-sonnet-4-20250514"` - AI model used

### Document Processing Flow
1. Documents placed in `docs/` folder are auto-loaded on startup
2. `DocumentProcessor` extracts course structure and creates chunks
3. Course metadata added to catalog for semantic search
4. Content chunks stored in content collection with full attribution

### API Endpoints
- `POST /api/query`: Main query processing with session management
- `GET /api/courses`: Returns course statistics and titles
- Frontend served from `/` (static files from `frontend/`)

## Development Notes

### Adding New Document Types
Extend `DocumentProcessor.process_course_document()` to handle new file formats. Current support: TXT files with structured format:
- Line 1: `Course Title: [title]`
- Line 2: `Course Link: [url]` (optional)
- Line 3: `Course Instructor: [instructor]` (optional)
- Following lines: `Lesson N: [title]` markers followed by content

### Modifying Search Behavior
The AI's search decisions are controlled by the system prompt in `ai_generator.py`. The search tool definition in `search_tools.py` determines available search parameters.

### Vector Store Operations
ChromaDB collections are managed in `vector_store.py`. Use `clear_all_data()` for full rebuilds. Course names serve as unique identifiers.

### Frontend Integration
The frontend uses vanilla JavaScript with marked.js for markdown rendering. API calls include session management for conversation continuity.

### Key Dependencies
- `chromadb==1.0.15`: Vector database for embeddings storage
- `anthropic==0.58.2`: Claude AI API client
- `sentence-transformers==5.0.0`: Text embeddings (`all-MiniLM-L6-v2`)
- `fastapi==0.116.1`: Web framework with auto-generated API docs at `/docs`
- `uvicorn==0.35.0`: ASGI server for running the application