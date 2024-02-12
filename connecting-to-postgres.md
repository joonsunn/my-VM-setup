# On Connecting to Postgres

Base setup to create postgres+pgadmin container hosted on Docker: <https://gist.github.com/Da9el00/ff1d36bf914dc1c6c71aa9d70ac68136> and <https://www.youtube.com/watch?v=qECVC6t_2mU>

docker-compose.yml:

```bash
version: "3.8"
services:
  db:
    container_name: postgres_container
    image: postgres
    restart: always
    environment:
      POSTGRES_USER: [USERNAME]
      POSTGRES_PASSWORD: [PASSWORD]
      POSTGRES_DB: [DB_NAME]
    ports:
      - "5432:5432"
  pgadmin:
    container_name: pgadmin4_container
    image: dpage/pgadmin4
    restart: always
    environment:
      PGADMIN_DEFAULT_EMAIL: [USER_EMAIL]
      PGADMIN_DEFAULT_PASSWORD: [PASSWORD_FOR_PGADMIN]
    ports:
      - "5050:80"
```

Credentials for pgadmin and postgres are independent.

```bash
docker compose up
```

Database url environment variable will be:

```bash
DATABASE_URL="postgresql://[username]:[password]@[hostname or IP address]:5432/[db_name]?schema=public"
```

## Method 1: Using separate Express server

More info: <https://blog.logrocket.com/crud-rest-api-node-js-express-postgresql/>

```javascript
const { SQLStatement } = require("sql-template-strings");

const Pool = require("pg").Pool;
const pool = new Pool({
  user: "[USERNAME]",
  host: "[IP ADDRESS]",
  database: "[DB_NAME]",
  password: "[PASSWORD]",
  port: 5432,
});

// helper function for template string literal function: https://www.reddit.com/r/nextjs/comments/17jglcd/dropin_replacement_code_for_nextjs_magic_sql_tag/?rdt=45248
async function sql(strings, ...values) {
  const statement = new SQLStatement(strings, values);
  return statement;
}

const getNotes = async (request, response) => {
  pool.query("SELECT * FROM notes", (error, results) => {
    if (error) {
      throw error;
    }
    response.status(200).json(results.rows);
  });
};

const getNoteById = async (request, response) => {
  const { id } = request.params;
  console.log(request.query);
  console.log(request.params);
  try {
    pool.query(
      await sql`SELECT * FROM notes WHERE id = ${id}`,
      (error, results) => {
        if (error) {
          console.log(error);
          response.status(500).json({ error: "error" });
        } else {
          //   console.log(results);
          response.status(200).json(results.rows);
        }
      }
    );
  } catch (error) {
    console.log(error);
  }
};

module.exports = {
  getNotes,
  getNoteById,
};
```

Then in `index.js` just make the appropriate routes call the appropriate middlewares.

Front end app to use `axios` to make a HTTP request to the appropriate end point as usual. Might need to use CORS.

## Method 2: Using integrated backend of NextJS via Prisma

More info: <https://medium.com/@2018.itsuki/postgresql-with-next-js-and-prisma-44f66a05378a>

```bash
npm install prisma @prisma/client
npx prisma init
```

After `init`, `prisma.schema` will be created.

If already have db set up and just waiting to be connected, run:

```bash
npx prisma db pull
```

The run the following to create client:

```bash
npx prisma generate
```

Make a file in `/lib/prisma.ts`:

```javascript
import { PrismaClient } from "@prisma/client";

const prismaClientSingleton = () => {
    return new PrismaClient()
}

type PrismaClientSingleton = ReturnType<typeof prismaClientSingleton>

const globalForPrisma = globalThis as unknown as {
    prisma: PrismaClientSingleton | undefined
}

const prisma = globalForPrisma.prisma ?? prismaClientSingleton()

export default prisma

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

Then, create a file under `/src/app/api/[route_name]/route.ts`:

```javascript
import prisma from "@/lib/prisma";
import type { NextApiRequest, NextApiResponse } from "next";


export async function GET(request: NextApiRequest, response: NextApiResponse) {
        try {
        if(request.method === 'GET'){
            const data = await prisma.notes.findMany()
            return Response.json(data)
        }
    } catch (error) {
        console.log(error)
        return response.status(500).json(error)
    }
}
```

Then call it as you would from whatever component:

```javascript
  const getPrismaNotes = async () => {
    const response = await axios.get("/api/fetchNotes");
    return response.data;
  };
```

### Set up `react-query`

To make life easier, use `react-query` for querying:

```bash
npm install @tanstack/react-query
```

Then make a file under `/src/components/ReactQueryClientProvider.ts`:

```javascript
"use client";

import { QueryClient, QueryClientProvider } from "react-query";
import { useState } from "react";

export const ReactQueryClientProvider = ({
  children,
}: {
  children: React.ReactNode;
}) => {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            // With SSR, we usually want to set some default staleTime
            // above 0 to avoid refetching immediately on the client
            staleTime: 60 * 1000,
          },
        },
      })
  );
  return (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
};
```

Then wrap the whole root layout with the client provider component:

`/src/layout.tsx`

```javascript
...
import { ReactQueryClientProvider } from "@/components/ReactQueryClientProvider";
...

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <ReactQueryClientProvider>
      <html lang="en">
        <body className={inter.className}>{children}</body>
      </html>
    </ReactQueryClientProvider>
  );
}
```

Then can call `useQuery` in all components within the project.

