# MCP Server Template

A template for building custom MCP (Model Context Protocol) servers. This template provides a solid foundation with example tools, proper TypeScript setup, testing infrastructure, and best practices.

## 🚀 Quick Start

### 1. Clone or Copy This Template

```bash
# Copy the template to your new project
cp -r template-mcp my-new-mcp-server
cd my-new-mcp-server
```

### 2. Customize the Project

1. **Update `package.json`**:
   ```bash
   # Change the name, description, and other metadata
   vim package.json
   ```

2. **Rename the main client class**:
   ```bash
   # Rename template-client.ts to something meaningful
   mv src/template-client.ts src/my-client.ts
   ```

3. **Update imports in `src/index.ts`**:
   ```typescript
   // Change this line:
   import { TemplateClient } from './template-client.js';
   // To:
   import { MyClient } from './my-client.js';
   ```

### 3. Install and Build

```bash
# Install dependencies
npm install

# Build the project
npm run build

# Run tests
npm test
```

## 📁 Project Structure

```
my-mcp-server/
├── src/
│   ├── index.ts              # Main MCP server implementation
│   └── template-client.ts    # Business logic and external API calls
├── tests/
│   └── template-client.test.ts # Test suite
├── dist/                     # Compiled TypeScript output
├── package.json              # Project configuration
├── tsconfig.json             # TypeScript configuration
├── jest.config.js            # Testing configuration
├── .gitignore               # Git ignore rules
└── README.md                # This file
```

## 🛠️ Customizing Your MCP Server

### 1. Define Your Tools

Edit `src/index.ts` to define your tools in the `ListToolsRequestSchema` handler:

```typescript
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: 'my_custom_tool',
        description: 'Description of what this tool does',
        inputSchema: {
          type: 'object',
          properties: {
            // Define your tool's input parameters
            param1: {
              type: 'string',
              description: 'Description of param1',
            },
            param2: {
              type: 'number',
              description: 'Description of param2',
              default: 10,
            },
          },
          required: ['param1'],
        },
      },
      // Add more tools here...
    ],
  };
});
```

### 2. Implement Tool Logic

Add corresponding cases in the `CallToolRequestSchema` handler:

```typescript
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  try {
    switch (name) {
      case 'my_custom_tool': {
        const result = await myClient.myCustomMethod(
          args.param1 as string,
          args.param2 as number || 10
        );
        return {
          content: [
            {
              type: 'text',
              text: JSON.stringify(result, null, 2),
            },
          ],
        };
      }
      // Add more cases here...
    }
  } catch (error) {
    // Error handling...
  }
});
```

### 3. Implement Business Logic

Edit `src/template-client.ts` (or your renamed client) to implement your actual business logic:

```typescript
export class MyClient {
  constructor(apiKey?: string, baseUrl?: string) {
    // Initialize your client with necessary configuration
  }

  async myCustomMethod(param1: string, param2: number): Promise<any> {
    // Implement your business logic here
    // Make API calls, process data, etc.
    return {
      result: 'Your processed result',
      input: { param1, param2 },
    };
  }
}
```

## 🔧 Configuration

### Environment Variables

Create a `.env` file for sensitive configuration:

```bash
API_KEY=your_api_key_here
BASE_URL=https://api.yourservice.com
DEBUG=true
```

Load environment variables in your client:

```typescript
import dotenv from 'dotenv';
dotenv.config();

export class MyClient {
  constructor() {
    this.apiKey = process.env.API_KEY;
    this.baseUrl = process.env.BASE_URL || 'https://api.example.com';
  }
}
```

### Claude Desktop Integration

Add your server to Claude Desktop's configuration:

1. Open `~/Library/Application Support/Claude/claude_desktop_config.json`
2. Add your server:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["/path/to/your/project/dist/index.js"],
      "env": {
        "API_KEY": "your_api_key_here"
      }
    }
  }
}
```

3. Restart Claude Desktop

## 🧪 Testing

### Running Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run with coverage
npm run test:coverage
```

### Writing Tests

Add tests for your custom methods in `tests/`:

```typescript
import { MyClient } from '../src/my-client';

describe('MyClient', () => {
  let client: MyClient;

  beforeAll(() => {
    client = new MyClient();
  });

  test('should process data correctly', async () => {
    const result = await client.myCustomMethod('test', 42);
    expect(result).toBeDefined();
    expect(result.input.param1).toBe('test');
  });
});
```

## 📦 Building and Distribution

### Build for Production

```bash
# Clean previous build
npm run clean

# Build
npm run build

# The built files will be in dist/
```

### Publishing to npm

1. Update version in `package.json`
2. Build the project
3. Publish:

```bash
npm publish
```

## 🔍 Debugging

### Enable Debug Logging

```typescript
// In your client, add debug logging
console.error('Debug:', { param1, param2, result });
```

### Test Server Directly

```bash
# Run the server directly to test
npm start

# Or run in development mode with auto-reload
npm run dev
```

## 📚 Common Patterns

### 1. API Client Pattern

```typescript
export class MyApiClient {
  private httpClient: AxiosInstance;

  constructor(apiKey: string, baseUrl: string) {
    this.httpClient = axios.create({
      baseURL: baseUrl,
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json',
      },
    });
  }

  async getData(id: string): Promise<any> {
    const response = await this.httpClient.get(`/data/${id}`);
    return response.data;
  }
}
```

### 2. Data Processing Pattern

```typescript
export class DataProcessor {
  static filterData<T>(data: T[], predicate: (item: T) => boolean): T[] {
    return data.filter(predicate);
  }

  static transformData<T, U>(data: T[], transformer: (item: T) => U): U[] {
    return data.map(transformer);
  }
}
```

### 3. Error Handling Pattern

```typescript
export class CustomError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode?: number
  ) {
    super(message);
    this.name = 'CustomError';
  }
}

// Usage in tools
try {
  const result = await myClient.riskyOperation();
  return result;
} catch (error) {
  if (error instanceof CustomError) {
    throw new McpError(ErrorCode.InternalError, `Custom error: ${error.message}`);
  }
  throw new McpError(ErrorCode.InternalError, `Unexpected error: ${error}`);
}
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Make your changes
4. Add tests for your changes
5. Run tests: `npm test`
6. Commit your changes: `git commit -am 'Add some feature'`
7. Push to the branch: `git push origin feature/my-feature`
8. Submit a pull request

## 📄 License

MIT License - see LICENSE file for details.

## 🆘 Getting Help

- Check the [MCP Documentation](https://modelcontextprotocol.io/docs)
- Review the [SDK Reference](https://github.com/modelcontextprotocol/typescript-sdk)
- Look at example implementations
- Open an issue if you find bugs or need features

## 🚀 Next Steps

After customizing this template:

1. **Replace all placeholder content** with your actual implementation
2. **Add proper error handling** for your specific use cases  
3. **Implement authentication** if your service requires it
4. **Add input validation** for your tool parameters
5. **Write comprehensive tests** for your business logic
6. **Update documentation** to reflect your actual tools and usage
7. **Configure CI/CD** for automated testing and deployment

Happy building! 🎉
