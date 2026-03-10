# AI Bot Template

Create the following files in the project directory. Replace `<project-name>` with the user's chosen project name (kebab-case).

This template creates an AI-powered bot with OpenAI integration, streaming responses, and conversation memory.

## `package.json`

```json
{
  "name": "<project-name>",
  "version": "0.0.1",
  "license": "MIT",
  "private": true,
  "main": "dist/index",
  "types": "dist/index",
  "files": [
    "dist",
    "README.md"
  ],
  "scripts": {
    "clean": "npx rimraf ./dist",
    "build": "npx tsup",
    "start": "node .",
    "dev": "tsx watch -r dotenv/config src/index.ts"
  },
  "dependencies": {
    "@microsoft/teams.ai": "latest",
    "@microsoft/teams.api": "latest",
    "@microsoft/teams.apps": "latest",
    "@microsoft/teams.cards": "latest",
    "@microsoft/teams.common": "latest",
    "@microsoft/teams.dev": "latest",
    "@microsoft/teams.graph": "latest",
    "@microsoft/teams.openai": "latest"
  },
  "devDependencies": {
    "@types/node": "^22.5.4",
    "dotenv": "^16.4.5",
    "rimraf": "^6.0.1",
    "tsx": "^4.20.6",
    "tsup": "^8.4.0",
    "typescript": "^5.4.5"
  }
}
```

## `tsconfig.json`

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "module": "NodeNext",
    "target": "ESNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noImplicitAny": true,
    "declaration": true,
    "inlineSourceMap": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": false,
    "resolveJsonModule": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "pretty": true,
    "outDir": "dist",
    "rootDir": "src",
    "types": ["node"]
  }
}
```

## `tsup.config.js`

```js
/** @type {import('tsup').Options} */
module.exports = {
  dts: true,
  minify: false,
  bundle: false,
  sourcemap: true,
  treeshake: true,
  splitting: true,
  clean: true,
  outDir: 'dist',
  entry: ['src/index.ts'],
  format: ['cjs'],
};
```

## `src/index.ts`

This is the main bot logic — an AI-powered bot with streaming and conversation memory.

```typescript
import { App } from '@microsoft/teams.apps';
import { ChatPrompt, Message } from '@microsoft/teams.ai';
import { LocalStorage } from '@microsoft/teams.common/storage';
import { DevtoolsPlugin } from '@microsoft/teams.dev';
import { OpenAIChatModel } from '@microsoft/teams.openai';

const storage = new LocalStorage<Array<Message>>();
const app = new App({
  storage,
  plugins: [new DevtoolsPlugin()],
});

app.on('message', async ({ stream, activity }) => {
  const prompt = new ChatPrompt({
    messages: storage.get(`${activity.conversation.id}/${activity.from.id}`),
    model: new OpenAIChatModel({
      model: 'gpt-4o',
      apiKey: process.env.OPENAI_API_KEY,
    }),
  });

  await prompt.send(activity.text, {
    onChunk: (chunk) => stream.emit(chunk),
  });
});

app.start(process.env.PORT || 3978).catch(console.error);
```

### How the AI template works

- **`LocalStorage`** — Persists conversation history per user per conversation. The key `${conversationId}/${userId}` ensures each user has their own memory.
- **`ChatPrompt`** — Manages the LLM conversation. Pass `messages` to give it conversation history so the bot remembers prior messages.
- **`OpenAIChatModel`** — Connects to OpenAI (or Azure OpenAI). Configured via environment variables.
- **Streaming** — The `onChunk` callback streams tokens to the user in real-time via `stream.emit()`, so they see the response as it's generated rather than waiting for the full response.

### Using Azure OpenAI instead

If the user wants Azure OpenAI instead of regular OpenAI, the model initialization changes to:

```typescript
const model = new OpenAIChatModel({
  model: process.env.AZURE_OPENAI_MODEL_DEPLOYMENT_NAME!,
  apiKey: process.env.AZURE_OPENAI_API_KEY,
  endpoint: process.env.AZURE_OPENAI_ENDPOINT,
  apiVersion: process.env.AZURE_OPENAI_API_VERSION,
});
```

And the ChatPrompt becomes:

```typescript
const prompt = new ChatPrompt({
  instructions: 'You are a helpful assistant.',
  messages: storage.get(`${activity.conversation.id}/${activity.from.id}`),
  model,
});
```

## `.env`

This file should already exist from `teams2 app create --env .env`. The AI template needs additional variables:

```
CLIENT_ID=<from teams2>
CLIENT_SECRET=<from teams2>

# For OpenAI
OPENAI_API_KEY=<user provides>

# OR for Azure OpenAI (use one or the other)
AZURE_OPENAI_API_KEY=<user provides>
AZURE_OPENAI_ENDPOINT=<user provides>
AZURE_OPENAI_API_VERSION=2024-12-01-preview
AZURE_OPENAI_MODEL_DEPLOYMENT_NAME=<user provides>
```

Ask the user whether they want to use OpenAI or Azure OpenAI, and adjust the `src/index.ts` and `.env` accordingly.

## Extending the AI Bot

Once the basic AI bot is running, here are common next steps developers might want:

### Adding system instructions
```typescript
const prompt = new ChatPrompt({
  instructions: 'You are a helpful assistant that specializes in...',
  messages: storage.get(`${activity.conversation.id}/${activity.from.id}`),
  model,
});
```

### Adding function calling (tools)
```typescript
const prompt = new ChatPrompt({
  instructions: 'You are a helpful assistant.',
  model,
  messages: existingMessages,
}).function(
  'search',
  'search for information',
  {
    type: 'object',
    properties: {
      query: { type: 'string', description: 'The search query' },
    },
    required: ['query'],
  },
  async ({ query }: { query: string }) => {
    // Implement your search logic
    return { results: [] };
  }
);
```

### Marking responses as AI-generated
```typescript
import { MessageActivity } from '@microsoft/teams.api';

const response = await prompt.send(activity.text);
if (response.content) {
  const message = new MessageActivity(response.content).addAiGenerated();
  await send(message);
}
```

### Adding feedback collection
```typescript
import { MessageActivity } from '@microsoft/teams.api';

// In the message handler:
const message = new MessageActivity(response.content)
  .addAiGenerated()
  .addFeedback();
await send(message);

// Handle feedback:
app.on('message.submit.feedback', async ({ activity, log }) => {
  const { reaction, feedback } = activity.value.actionValue;
  log.info(`Feedback: ${reaction}`, feedback);
});
```
