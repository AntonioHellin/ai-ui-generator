# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in a chat interface, and the AI generates React components that are previewed in real-time using a virtual file system (no files written to disk). The app supports both authenticated users (with Prisma/SQLite persistence) and anonymous users.

## Commands

### Development
```bash
npm run dev              # Start dev server with Turbopack
npm run dev:daemon       # Start dev server in background with logs
```

### Build & Start
```bash
npm run build            # Build for production
npm run start            # Start production server
```

### Testing
```bash
npm test                 # Run tests with Vitest
```

### Database
```bash
npm run setup            # Install deps, generate Prisma client, run migrations
npm run db:reset         # Reset database (force)
npx prisma migrate dev   # Run migrations in development
npx prisma generate      # Generate Prisma client
```

### Linting
```bash
npm run lint             # Run ESLint
```

## Architecture

### Virtual File System
The core abstraction is `VirtualFileSystem` (src/lib/file-system.ts), which maintains an in-memory file tree. It provides:
- File/directory creation, reading, updating, deletion
- Path normalization (ensures consistent `/` prefix)
- Rename operations (used for moving files)
- Serialization/deserialization for persistence
- Text editor-like operations (view, create, str_replace, insert)

The file system is instantiated per-request in the chat API and reconstructed from the serialized state stored in the database.

### AI Tools
The AI uses two main tools defined in src/lib/tools/:
1. **str_replace_editor** (str-replace.ts): View files, create files, replace strings, insert lines. Used for generating and editing component code.
2. **file_manager** (file-manager.ts): Rename and delete files/folders. Supports moving files via rename.

Both tools operate on the VirtualFileSystem instance.

### JSX Transformation & Preview
The preview system (src/components/preview/PreviewFrame.tsx + src/lib/transform/jsx-transformer.ts):
1. Takes the virtual file system's files
2. Transforms JSX/TSX to JavaScript using Babel standalone
3. Creates blob URLs for each transformed file
4. Builds an import map mapping module specifiers to blob URLs
5. Generates an HTML document with the import map and entry point
6. Renders in a sandboxed iframe

**Key features:**
- Supports `@/` import alias (maps to root `/`)
- Handles missing imports by creating placeholder modules
- Automatically detects entry point (App.jsx, index.jsx, etc.)
- Collects CSS files and injects as `<style>` tags
- Third-party packages resolved via esm.sh

### Authentication & Persistence
- JWT-based auth (src/lib/auth.ts) using jose library
- Passwords hashed with bcrypt
- Projects stored in SQLite via Prisma (src/lib/prisma.ts)
- Prisma schema in prisma/schema.prisma with User and Project models
- Projects store messages (JSON) and file system data (JSON)
- Anonymous users tracked via src/lib/anon-work-tracker.ts

### AI Provider Abstraction
src/lib/provider.ts provides `getLanguageModel()`:
- Uses Anthropic Claude (claude-haiku-4-5) if ANTHROPIC_API_KEY is set
- Falls back to MockLanguageModel for static responses when no API key
- Mock provider generates basic Counter/Form/Card components through multiple tool calls

### System Prompt
src/lib/prompts/generation.tsx contains the system prompt instructing the AI to:
- Create React components with Tailwind CSS
- Use `/App.jsx` as the entry point
- Use `@/` import alias for local files
- Keep responses brief unless asked
- Operate on root route `/` of virtual FS

### Data Flow
1. User sends message in ChatInterface (src/components/chat/ChatInterface.tsx)
2. Message sent to /api/chat/route.ts with file system state
3. API reconstructs VirtualFileSystem, streams AI response with tools
4. Tools modify file system, results returned to AI
5. File system state saved to database (for authenticated users)
6. Updated files sent to client, preview auto-updates via PreviewFrame

### Testing
- Vitest configured with jsdom environment (vitest.config.mts)
- Tests for file system, contexts, components in __tests__ directories
- Use `@testing-library/react` for component tests

## Important Notes

- All file paths in the virtual FS must start with `/`
- The preview only works with JavaScript/TypeScript/JSX/TSX files
- Entry point defaults to `/App.jsx` but auto-detects other common patterns
- Import map generation handles both local modules and third-party packages
- Database migrations auto-run on `npm run setup`
- Prisma client output is in src/generated/prisma (not default node_modules)
- Cross-env is used for Windows compatibility with NODE_OPTIONS
- Use comments sparingly. Only comment complex code.
- The database schema is defined in the @prisma/schema.prisma file. Reference it anytime you need to understand the structure of data stored in the database.