# Typescript shananigans

## Set up NodeJS/ExpressJS project with Typescript

```bash
npm init -y
npm i -D typescript ts-node nodemon
npm i express
npm i -D @types/node @types/express
npx tsc --init
```

`tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "rootDir": "./src",
    "outDir": "./dist",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true
  }
}
```

`package.json`:

```json

"main": "index.js",

...,

"scripts": {
  "start": "node dist/index.js",
  "build": "rm -rf dist && tsc",
  "dev": "nodemon src/index.ts"
},
```

`index.ts`:

```typescript
import express from "express";
import http from "http";
import os from "os";

const internalIp = Object.values(os.networkInterfaces())
  .flat()
  .filter((item) => item?.family === "IPv4" && !item?.internal)[0]?.address;

const PORT = process.env.PORT ?? 3000;

const app = express();
const server = http.createServer(app);

// CAPTURE APP TERMINATION / RESTART EVENTS
// To be called when process is restarted or terminated
const gracefulShutdown = function (msg: string, callback: () => void) {
  console.log(msg);
  callback();
};
// For nodemon restarts
process.once("SIGUSR2", function () {
  gracefulShutdown("nodemon restart", function () {
    process.kill(process.pid, "SIGUSR2");
  });
});
// For app termination
process.on("SIGINT", function () {
  gracefulShutdown("\napp terminated via Ctrl+C", function () {
    server.close(() => {
      process.exit(0);
    });
  });
});
// For deployed  app termination
process.on("SIGTERM", function () {
  gracefulShutdown("deployed app termination", function () {
    server.close(() => {
      process.exit(0);
    });
  });
});

function main() {
  app.get("/", (req, res) => {
    res.send("Hello World!");
  });

  server.listen(PORT, () =>
    console.log(
      `Server successfully started!
Local:            http://localhost:${PORT}
On your network:  http://${internalIp}:${PORT}`
    )
  );
}

main();
```

## Setting up path alias (for ExpressJS apps without custom bundlers)

Dependencies:

- module-alias
- tsc-alias

```bash
npm i module-alias tsc-alias
```

Dev-dependencies:

- @types/module-alias

```bash
npm i -D @types/module-alias
```

`package.json`:

```json
"scripts": {
    "build": "rm -rf dist && tsc && tsc-alias -p tsconfig.json",
    "start": "node dist/index.js",
    "dev": "nodemon src/index.ts"
  },

  ...,

"_moduleAliases": {
"@src": "src",
"@service": "src/service",
"@controller": "src/controller"
}
```

`tsconfig.json`:

```json
{
  "compilerOptions": {

    ...,

    "baseUrl": "src",
    "paths": {
      "@src/*": ["*"],
      "@service/*": ["service/*"],
      "@controller/*": ["controller/*"]
    }
  }
}
```

`index.ts` or `server.js` (entrypoint of application):

Include at top of file:

```typescript
import "module-alias/register";
```

By default, VS Code only imports shortest path. If want to maintain full path alias imports, change settings as below:

`"typescript.preferences.importModuleSpecifier": "non-relative"`

### Short explanation

Paths in `tsconfig.json` is for TS server to perform autocomplete.  
`_moduleAliases` at `package.json` is for compiler to replace paths for running output.  
`tsc-alias` will replace path aliases with relative paths in compiled JavacScript files.
