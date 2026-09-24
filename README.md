# ai-ui-generator

An AI-powered React component generator featuring real-time preview in a sandboxed virtual file system.

## Project Overview

`ai-ui-generator` is an intelligent web application that turns conversational natural language descriptions into live React components with Tailwind CSS styling. Operating on an in-memory `VirtualFileSystem`, generated components are dynamically compiled via Babel Standalone and rendered in an isolated sandbox iframe with zero filesystem disk pollution.

## Features

- **Conversational Component Authoring**: Describe UI components naturally through a chat interface.
- **In-Memory Virtual File System**: Multi-file component trees constructed and edited without writes to disk.
- **Live Sandboxed Preview**: Hot compilation of JSX/TSX and CSS injection in real time.
- **Anthropic Claude Integration**: Powered by Claude models with graceful fallback to mock component generators when no API key is supplied.
- **User Authentication & Project Persistence**: JWT auth with SQLite persistence via Prisma.

## Prerequisites

- [Node.js](https://nodejs.org/) (version 18.x or later)
- [npm](https://www.npmjs.com/) (version 9.x or later)

## Installation/Build

1. Clone the repository and navigate to the project root:
   ```bash
   git clone https://github.com/AntonioHellin/ai-ui-generator.git
   cd ai-ui-generator
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Configure environment variables:
   ```bash
   cp .env.example .env
   ```
   Add your `ANTHROPIC_API_KEY` and set a secure `JWT_SECRET`.

4. Run database setup and Prisma client generation:
   ```bash
   npm run setup
   ```

5. Build for production:
   ```bash
   npm run build
   ```

## Usage

Start the development server with Turbopack:
```bash
npm run dev
```
Open `http://localhost:3000` to interact with the application.

To run tests with Vitest:
```bash
npm test
```

To run lint checks:
```bash
npm run lint
```

## Environment Variables

| Variable | Description | Default |
|---|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key for Claude AI generation | *(Optional, falls back to Mock)* |
| `JWT_SECRET` | Secret key for JWT session tokens | *(Required for authentication)* |
| `DATABASE_URL` | SQLite database connection string | `file:./dev.db` |
| `PORT` | Local web server port | `3000` |

## License

This project is licensed under the MIT License.
