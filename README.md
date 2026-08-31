# my-snippets-cli

A command-line companion for the [My Snippets](https://my-snippets-omega.vercel.app)
component registry.

> **Status: stub.** `src/index.ts` is six lines that print the CLI name and echo back its
> arguments. No commands are implemented. This README describes what is actually here today.

---

## What exists

```ts
#!/usr/bin/env node

const args = process.argv.slice(2);

console.log("my-snippets CLI");
console.log("args:", args);
```

That's the whole program. The surrounding packaging, however, is real and working:

| Piece | State |
| --- | --- |
| Build | `tsc` → `dist/`, CommonJS |
| Binary | `bin: { "my-snippets": "./dist/index.js" }` with a shebang |
| Publishing | `files: ["dist"]`, `prepare` script builds on install |
| Dependencies | None — TypeScript and `@types/node` are dev-only |
| Global link | Currently linked globally, so `my-snippets` resolves to this folder |

So the distribution mechanics work end to end — running `my-snippets foo bar` anywhere prints
`args: [ 'foo', 'bar' ]`. Only the functionality is missing.

---

## Architecture

Deliberately dependency-free. A CLI that installs components should start fast and not drag a
tree of packages behind it, so the intent appears to be Node built-ins only (`fetch`, `fs`,
`path`, `util.parseArgs`) — the same approach taken by
`my-snippets/scripts/generate-top-langs.mjs`.

**What it would talk to:** the registry publishes static JSON that already fits this purpose:

```text
https://my-snippets-omega.vercel.app/registry.json    ← index of all 76 components
https://my-snippets-omega.vercel.app/r/<slug>.json    ← one component + its file closure
https://my-snippets-omega.vercel.app/r/theme.json     ← the required CSS variants
```

Each `r/<slug>.json` follows the shadcn registry-item schema and carries every file the
component needs, so a fetch-and-write CLI needs no server of its own.

**Worth knowing:** `npx shadcn@latest add <url>` already does this. See
[IMPROVEMENTS.md](./IMPROVEMENTS.md) §1 for whether this CLI should exist at all, and what
would make it worth building.

---

## Development

```bash
npm install
npm run dev      # tsc --watch
npm run build    # tsc
node dist/index.js hello
```

---

## Layout

```text
my-snippets-cli/
├── src/index.ts     ← the stub
├── dist/            ← build output
├── tsconfig.json
└── package.json
```
