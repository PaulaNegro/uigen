# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in natural language, and Claude generates them with real-time preview and code editing. Built with Next.js 15 (App Router), React 19, TypeScript, and Tailwind CSS v4.

## Commands

- **Development:** `npm run dev` (uses Turbopack)
- **Build:** `npm run build`
- **Test:** `npm run test` (Vitest)
- **Lint:** `npm run lint`
- **Setup:** `npm run setup` (install deps, generate Prisma client, run migrations)
- **DB Reset:** `npm run db:reset`

## Architecture

### Core Flow
1. User sends message via chat interface
2. `/api/chat` endpoint streams response from Claude (claude-haiku-4-5 via Vercel AI SDK)
3. Claude uses tools (`str_replace_editor`, `file_manager`) to create/modify files in virtual file system
4. Preview frame transforms JSX with Babel standalone and renders in sandboxed iframe

### Key Directories
- `/src/app/api/chat` - AI chat endpoint with streaming
- `/src/lib/tools` - AI tool definitions (str-replace, file-manager)
- `/src/lib/contexts` - FileSystemContext (virtual FS), ChatContext (AI chat state)
- `/src/lib/prompts` - System prompts for Claude
- `/src/lib/transform` - JSX transformation and import map generation
- `/src/lib/file-system.ts` - Virtual file system (in-memory, serializable)
- `/src/components/preview` - Iframe-based live preview with Babel transformation

### Virtual File System
All generated code lives in memory, not on disk. The `FileSystem` class in `file-system.ts` provides CRUD operations and serializes to JSON for database persistence. Every project must have `/App.jsx` as the entry point.

### Module Resolution
- Local imports use `@/` alias mapped to blob URLs of transformed code
- External packages (React, lucide-react, etc.) resolve via esm.sh CDN
- Import map generated dynamically in `import-map.ts`

### Authentication
JWT-based sessions stored in httpOnly cookies. Users table in SQLite via Prisma. Projects can have null userId for anonymous users.

### Database
SQLite with Prisma ORM. Schema in `/prisma/schema.prisma`. Projects store messages and file system state as JSON blobs.

## Testing
Tests use Vitest with jsdom and React Testing Library. Test files in `__tests__` directories alongside source files.
