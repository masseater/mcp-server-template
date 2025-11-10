# MCP Server Template

A production-ready template for building Model Context Protocol (MCP) servers with TypeScript. This template provides a solid foundation with support for both stdio and HTTP transports, extensible tool architecture, and best practices.

## Features

- ✅ **Dual Transport Support**: stdio (default) and HTTP transports
- ✅ **Type-Safe**: Full TypeScript support with strict mode enabled
- ✅ **Extensible Architecture**: Base classes for easy tool creation
- ✅ **Production-Ready**: Logging, error handling, and configuration management
- ✅ **Developer-Friendly**: Hot reload, ESLint, and comprehensive tooling
- ✅ **Sample Tools**: Echo and Ping tools as examples

## Prerequisites

- Node.js >= 22.10.0
- pnpm >= 10.19.0

## Quick Start

### 1. Use this template

Click "Use this template" on GitHub or clone this repository:

```bash
git clone https://github.com/yourusername/mcp-server-template.git
cd mcp-server-template
```

### 2. Install dependencies

```bash
pnpm install
```

### 3. Run in development mode

```bash
# stdio transport (default)
pnpm run dev

# HTTP transport (with hot reload)
pnpm run dev:http
```

### 4. Build for production

```bash
pnpm run build
```

### 5. Run in production

```bash
# stdio transport
pnpm run start:stdio

# HTTP transport
pnpm run start:http
```

## Project Structure

```
mcp-server-template/
├── src/
│   ├── server/
│   │   ├── mcp-server.ts       # Main MCP server implementation
│   │   └── tool-registry.ts    # Tool registration and management
│   ├── tools/
│   │   ├── base/
│   │   │   ├── base-tool.ts    # Base class for all tools
│   │   │   └── raw-tool.ts     # Base class for tools with raw option
│   │   ├── implementations/
│   │   │   ├── echo-tool.ts    # Example: Echo tool
│   │   │   └── ping-tool.ts    # Example: Ping tool
│   │   └── base-tool.ts        # Tool utilities
│   ├── types/
│   │   └── index.ts            # Type definitions
│   ├── config/
│   │   └── index.ts            # Configuration management
│   ├── utils/
│   │   └── logger.ts           # Logging utilities
│   └── index.ts                # Entry point
├── dist/                       # Compiled output (generated)
├── logs/                       # Log files (generated)
├── package.json
├── tsconfig.json
├── tsup.config.ts
└── README.md
```

## Creating Your Own Tools

### Basic Tool Example

Create a new file in `src/tools/implementations/`:

```typescript
import { z } from "zod";
import { BaseTool } from "../base/base-tool.js";
import type { MCPToolResponse } from "../../types/index.js";

type MyToolArgs = {
  input: string;
};

export class MyTool extends BaseTool<MyToolArgs> {
  readonly name = "my_tool";
  readonly description = "Description of what my tool does";

  getInputSchema() {
    return z.object({
      input: z.string().describe("Input parameter description"),
    });
  }

  async execute(args: MyToolArgs): Promise<MCPToolResponse<YourDataType>> {
    // Your tool logic here
    const result = await someOperation(args.input);

    return {
      success: true,
      message: "Operation completed successfully",
      data: result,
    };
  }
}
```

### Register Your Tool

Add your tool to `src/server/tool-registry.ts`:

```typescript
import { MyTool } from "../tools/implementations/my-tool.js";

// Add to Tool type
type Tool = EchoTool | PingTool | MyTool;

// Add to tools array in initialize()
const tools: Tool[] = [
  new EchoTool(context),
  new PingTool(context),
  new MyTool(context), // Add your tool
];
```

## Available Scripts

```bash
# Development
pnpm run dev          # Start with stdio transport (hot reload)
pnpm run dev:http     # Start with HTTP transport (hot reload)

# Production
pnpm run build        # Build for production
pnpm run start:stdio  # Run built server with stdio
pnpm run start:http   # Run built server with HTTP

# Code Quality
pnpm run type-check   # Run TypeScript type checking
pnpm run lint         # Run ESLint
pnpm run lint:fix     # Fix ESLint errors automatically

# Testing
pnpm run test         # Run tests once
pnpm run test:watch   # Run tests in watch mode
pnpm run test:coverage # Run tests with coverage

# Maintenance
pnpm run clean        # Remove dist directory
pnpm run knip         # Detect unused files/dependencies
pnpm run knip:fix     # Auto-fix knip issues
```

## Configuration

Configuration is managed through environment variables and can be extended in `src/config/index.ts`:

```typescript
export type ServerConfig = {
  server: {
    name: string;
    version: string;
  };
  mcp: {
    maxResponseSize: number;
    defaultPageSize: number;
  };
  logging: {
    level: string;
    enableDebugConsole: boolean;
  };
}
```

### Environment Variables

- `LOG_LEVEL`: Logging level (error|warn|info|debug, default: info)
- `DEBUG`: Enable debug console output (true|false, default: false)
- `PORT`: HTTP server port (default: 3000)

## Transport Modes

### stdio Transport (Default)

Used for direct integration with MCP clients like Claude Desktop:

```bash
pnpm run dev          # Development
pnpm run start:stdio  # Production
```

### HTTP Transport

Used for HTTP-based clients or debugging:

```bash
pnpm run dev:http     # Development (port 3000)
pnpm run start:http   # Production (port 3000)
```

Endpoints:
- `GET /health` - Health check endpoint
- `POST /mcp` - MCP Streamable HTTP endpoint

## Architecture

### Tool System

The template uses a **Template Method Pattern** for tools:

1. **BaseTool**: Abstract base class for all tools
   - Handles error catching and response formatting
   - Provides consistent tool definition interface

2. **RawTool**: Extended base for tools with `raw` option
   - Supports minimal vs full response modes
   - Automatically handles `raw` parameter extraction

### Error Handling

All tools automatically:
- Catch and log errors
- Return user-friendly error messages
- Set `isError: true` flag for error responses
- Never expose sensitive information

## Example Tools

### Echo Tool

Simple tool that echoes back the input message:

```bash
# Test with HTTP transport
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "method": "tools/call",
    "params": {
      "name": "echo",
      "arguments": {
        "message": "Hello, MCP!"
      }
    }
  }'
```

### Ping Tool

Health check tool that returns server status:

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "method": "tools/call",
    "params": {
      "name": "ping",
      "arguments": {}
    }
  }'
```

## Logging

Logs are written to:
- `logs/mcp-server-{timestamp}.log` (always)
- Console output (HTTP transport only)

Log levels: `error`, `warn`, `info`, `debug`

## Best Practices

1. **Tool Naming**: Use snake_case for tool names
2. **Descriptions**: Write clear, concise tool descriptions
3. **Input Validation**: Always use Zod schemas for input validation
4. **Error Messages**: Return user-friendly error messages
5. **Response Format**: Use `MCPToolResponse<T>` for consistent responses
6. **Context**: Add shared resources to `ToolContext` type

## Extending the Template

### Adding API Clients

1. Create client in `src/clients/`
2. Add to `ToolContext` type in `src/tools/base-tool.ts`
3. Initialize in `src/server/tool-registry.ts`
4. Use in your tools via `this.context`

### Adding Response Formatters

1. Create formatter in `src/formatters/`
2. Add to context or import directly in tools
3. Use RawTool base class for tools needing optimization

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT License - see LICENSE file for details

## Resources

- [Model Context Protocol Documentation](https://modelcontextprotocol.io)
- [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [Zod Documentation](https://zod.dev)

## Support

For issues and questions:
- GitHub Issues: https://github.com/yourusername/mcp-server-template/issues
- MCP Discord: https://discord.gg/modelcontextprotocol

---

**Built with ❤️ using TypeScript and the Model Context Protocol**
