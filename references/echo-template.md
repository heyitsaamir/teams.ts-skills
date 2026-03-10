# Echo Bot Template

Create the following files in the project directory. Replace `<project-name>` with the user's chosen project name (kebab-case).

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
    "@microsoft/teams.api": "latest",
    "@microsoft/teams.apps": "latest",
    "@microsoft/teams.cards": "latest",
    "@microsoft/teams.common": "latest",
    "@microsoft/teams.graph": "latest",
    "@microsoft/teams.dev": "latest"
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

This is the main bot logic — a simple echo bot that repeats back what the user says.

```typescript
import { App } from '@microsoft/teams.apps';
import { DevtoolsPlugin } from '@microsoft/teams.dev';

const app = new App({
  plugins: [new DevtoolsPlugin()],
});

app.on('message', async ({ send, activity }) => {
  await send({ type: 'typing' });
  await send(`you said "${activity.text}"`);
});

app.start(process.env.PORT || 3978).catch(console.error);
```

## `.env`

This file should already exist from `teams2 app create --env .env`. It contains:

```
CLIENT_ID=<from teams2>
CLIENT_SECRET=<from teams2>
TENANT_ID=<from teams2>
```

No additional environment variables are needed for the echo bot.
