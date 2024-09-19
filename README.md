# remix-nodejs-compat-v2

Sample using nodejs_compat_v2 to make Cloudflare work with pg.

# Notes

`pg@8.12.0` and `pg@8.13.0` does not support nodejs_compat_v2.  
Setting the flag will completely disable the operation.

# How to run

Installing [pg-compat](https://www.npmjs.com/package/pg-compat) makes pg compatible with nodejs_compat_v2.

# How to run the sample

- initialisation

```bash
pnpm install
pnpm dev:docker
pnpm prisma:migrate
```

- develop mode

```bash
pnpm dev
```

- production mode

```bash
pnpm build
pnpm start
```

# Sample code

Now you can run pg in Node.js and Cloudflare environments without having to use different Prisma packages!

- wrangler.toml

```
compatibility_date = "2024-08-21"
compatibility_flags = ["nodejs_compat_v2"]
```

- app/routes/\_index.tsx

```tsx
import { LoaderFunctionArgs } from "@remix-run/cloudflare";
import { useLoaderData } from "@remix-run/react";
// @prisma/xxx-worker is not used
import pg from "pg";
import { PrismaPg } from "@prisma/adapter-pg";
import { PrismaClient } from "@prisma/client";

export default function Index() {
  const values = useLoaderData<string[]>();
  return (
    <div>
      {values.map((v) => (
        <div key={v}>{v}</div>
      ))}
    </div>
  );
}

export async function loader({
  context,
}: LoaderFunctionArgs): Promise<string[]> {
  const url = new URL(context.cloudflare.env.DATABASE_URL);
  const schema = url.searchParams.get("schema") ?? undefined;
  const pool = new pg.Pool({
    connectionString: context.cloudflare.env.DATABASE_URL,
  });
  const adapter = new PrismaPg(pool, { schema });
  const prisma = new PrismaClient({ adapter });
  await prisma.test.create({ data: {} });
  return prisma.test.findMany().then((r) => r.map(({ id }) => id));
}
```
