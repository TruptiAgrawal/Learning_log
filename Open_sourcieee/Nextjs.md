● Learning Log — Contributing to Next.js (Open Source)

  Date: May 28, 2026

  ---
  What I Did Today

  I explored the Next.js open-source repository by Vercel with the goal of finding a real issue to contribute to as a beginner.

  ---
  1. What is Next.js?

  Next.js is a web framework built on top of React. It was created by Vercel (originally called "Zeit") in 2016 by an engineer named Guillermo Rauch. It is open-source, meaning anyone can use it, contribute to it, and deploy it anywhere — not just on Vercel's platform.

  What React gives you:
  - A way to build UI components

  What Next.js adds on top of React:
  - File-based routing (your folder structure = your URLs)
  - Server-side rendering
  - API routes
  - Image optimisation
  - A full build system

  The relationship:
  React (made by Meta)
    └── Next.js builds on React (made by Vercel, open-sourced)
          └── Your app builds on Next.js

  Who uses Next.js: TikTok, Twitch, Notion, GitHub, and thousands of companies worldwide — all without being dependent on
  Vercel.

  ---
  2. Why is Next.js a Separate Repo from Vercel?

  Vercel builds two separate things:

  ┌────────────────┬───────────────────────┬───────────────────────┐
  │                │        Next.js        │    Vercel Platform    │
  ├────────────────┼───────────────────────┼───────────────────────┤
  │ What it is     │ Open-source framework │ Commercial hosting    │
  ├────────────────┼───────────────────────┼───────────────────────┤
  │ Who can use it │ Anyone, anywhere      │ Vercel customers only │
  ├────────────────┼───────────────────────┼───────────────────────┤
  │ License        │ MIT (fully free)      │ Proprietary           │
  └────────────────┴───────────────────────┴───────────────────────┘

  Next.js being a separate public repository means:
  - Anyone in the world can contribute to it
  - It has no lock-in to Vercel's hosting
  - The community trusts it because it is truly open

  ▎ Think of it like Toyota building a road — anyone can drive on it, but Toyota cars happen to work really well on it.

  ---
  3. Understanding the Repository Structure

  The Next.js repository is a monorepo — one big repository that contains many packages. It uses pnpm as its package manager
   and Turborepo for managing builds across packages.

  next.js/
  ├── packages/next/        ← The main Next.js framework (what gets published to npm)
  │   └── src/
  │       ├── server/       ← Server-side code (rendering, routing, config)
  │       ├── client/       ← Browser-side code (Link, Image, Router)
  │       ├── build/        ← Build pipeline (webpack plugins, Turbopack config)
  │       └── shared/       ← Utilities shared between server and client
  ├── turbopack/            ← Turbopack bundler (written in Rust)
  ├── crates/               ← Rust code for SWC (JavaScript transforms)
  ├── test/                 ← All tests (e2e, development, production, unit)
  ├── docs/                 ← Documentation
  └── examples/             ← Example Next.js apps

  Key rule: You always edit files inside packages/next/src/. The packages/next/dist/ folder is auto-generated — never edit
  it.

  ---
  4. Two Routers in Next.js

  Next.js has two separate routing systems. Understanding which one you are working in is important.

  ┌───────────────┬─────────────────────────┬─────────────────────────┐
  │               │       App Router        │      Pages Router       │
  ├───────────────┼─────────────────────────┼─────────────────────────┤
  │ Folder        │ app/                    │ pages/                  │
  ├───────────────┼─────────────────────────┼─────────────────────────┤
  │ Components    │ React Server Components │ Client components       │
  ├───────────────┼─────────────────────────┼─────────────────────────┤
  │ Data fetching │ async components        │ getServerSideProps etc. │
  ├───────────────┼─────────────────────────┼─────────────────────────┤
  │ Status        │ Modern (v13+)           │ Legacy, still supported │
  └───────────────┴─────────────────────────┴─────────────────────────┘

  Most new work and bug fixes happen in the App Router.

  ---
  5. Key Things to Know Before Contributing

  Tooling

  - Always use pnpm, never npm or yarn
  - After editing source files, you must rebuild: pnpm --filter=next build
  - Use watch mode during development: pnpm --filter=next dev

  Branch

  - All pull requests go to the canary branch (not main or master)
  - Always create your branch off of canary

  Commits

  - Next.js requires signed commits — set up GPG signing before contributing
  - Never add "Generated with Claude Code" or AI footers to commits

  Tests

  pnpm test-dev-turbo   test/path/to/test.ts   # Turbopack (default)
  pnpm test-dev-webpack test/path/to/test.ts   # Webpack
  pnpm test-start-turbo test/path/to/test.ts   # Production mode

  New Features vs Bug Fixes

  - Bug fixes → just open a PR
  - New features → must open a GitHub Discussion first and get community approval before writing any code

  ---
  6. The Issue I Found — #73380

  What is the bug?

  In Next.js App Router, you can create a custom 404 page by creating a file called not-found.tsx. You can also organise
  your routes using route groups — folders with names in parentheses like (app). Route group folders are invisible to URLs —
   they only exist to help you organise your code.

  The bug: When not-found.tsx is placed inside a route group folder, Next.js does not recognise it as the root 404 page and
  shows the default ugly 404 instead.

  app/
  └── not-found.tsx             ✅ Works — custom 404 shows

  app/
  └── (app)/                    ← route group folder
      └── not-found.tsx         ❌ Broken — default 404 shows instead

  Why does the bug exist?

  Deep inside the codebase, in this file:
  packages/next/src/server/lib/find-page-file.ts

  There is a function called isRootNotFound (around line 154) that checks whether a given file is the root not-found page.
  It works by:

  1. Taking the full file path
  2. Removing the app/ directory prefix
  3. Checking if what remains matches the pattern not-found.<extension>

  The problem: When the file lives inside (app)/, after removing the app/ prefix, what remains is (app)/not-found.tsx — not
  just not-found.tsx. The pattern check fails because it doesn't expect anything before the filename.

  function isRootNotFound(filePath: string) {
    const rest = filePath.slice(appDirPath.length + 1)
    return rootNotFoundFileRegex.test(rest)
    // regex is: ^not-found\.<ext>$
    // "(app)/not-found.tsx" does NOT match this — BUG!
  }

  The fix direction

  The codebase already has a helper function called isGroupSegment() in:
  packages/next/src/shared/lib/segment.ts

  export function isGroupSegment(segment: string) {
    return segment[0] === '(' && segment.endsWith(')')
  }

  This function identifies route group folders. The fix is:
  - Split the remaining path into folder segments
  - Skip any segments that are route groups (those matching isGroupSegment)
  - Then check if what remains is just not-found.<extension>

  So (app)/not-found.tsx → strip (app) → not-found.tsx → matches correctly ✅

  Why this issue is a good pick

  - No competing pull requests (no one has claimed it)
  - Affects a very common Next.js pattern (route groups are used everywhere)
  - The fix is contained to one small function (~10 lines)
  - Only requires TypeScript knowledge — no Rust, no bundler internals
  - Real, visible, user-facing impact

  ---
  7. How to Reproduce the Bug Locally

  The test app was created inside the monorepo at:
  apps/notfound-test/
  ├── package.json
  └── app/
      ├── layout.tsx          ← root layout (required)
      └── (app)/
          ├── layout.tsx
          ├── page.tsx
          └── not-found.tsx   ← custom 404 (currently ignored by Next.js)

  Run the dev server:
  node packages/next/dist/bin/next dev apps/notfound-test --port 3333

  Visit http://localhost:3333/this-does-not-exist → you see the default Next.js 404 instead of the custom one. That confirms
   the bug.

  ---
  8. Next Steps

  - [ ] Confirm bug is reproducible locally
  - [ ] Write the fix in packages/next/src/server/lib/find-page-file.ts
  - [ ] Write a test using pnpm new-test
  - [ ] Run pnpm --filter=next build to rebuild
  - [ ] Run the test to confirm the fix works
  - [ ] Open a pull request against the canary branch

  ---
  Key Files to Remember

  ┌────────────────────────────────────────────────┬─────────────────────────────────────────┐
  │                      File                      │              What it does               │
  ├────────────────────────────────────────────────┼─────────────────────────────────────────┤
  │ packages/next/src/server/lib/find-page-file.ts │ Where the bug lives                     │
  ├────────────────────────────────────────────────┼─────────────────────────────────────────┤
  │ packages/next/src/shared/lib/segment.ts        │ Has isGroupSegment() helper for the fix │
  ├────────────────────────────────────────────────┼─────────────────────────────────────────┤
  │ apps/notfound-test/                            │ Local test app to reproduce the bug     │
  └────────────────────────────────────────────────┴─────────────────────────────────────────┘

  ---
  Vocabulary I Learned Today

  ┌───────────────┬─────────────────────────────────────────────────────────────────────────┐
  │     Term      │                                 Meaning                                 │
  ├───────────────┼─────────────────────────────────────────────────────────────────────────┤
  │ Monorepo      │ One repository containing many packages                                 │
  ├───────────────┼─────────────────────────────────────────────────────────────────────────┤
  │ pnpm          │ Package manager used by this project                                    │
  ├───────────────┼─────────────────────────────────────────────────────────────────────────┤
  │ Turbopack     │ Next.js's new Rust-based bundler (replaces webpack)                     │
  ├───────────────┼─────────────────────────────────────────────────────────────────────────┤
  │ Route group   │ A folder named (like-this) that organises routes without affecting URLs │
  ├───────────────┼─────────────────────────────────────────────────────────────────────────┤
  │ App Router    │ The modern Next.js routing system using the app/ folder                 │
  ├───────────────┼─────────────────────────────────────────────────────────────────────────┤
  │ canary        │ The development branch of Next.js where all PRs go                      │
  ├───────────────┼─────────────────────────────────────────────────────────────────────────┤
  │ not-found.tsx │ Special Next.js file that becomes your custom 404 page                  │
  ├───────────────┼─────────────────────────────────────────────────────────────────────────┤
  │ Open-source   │ Code that is public, free to use, and open for contributions            │
  └───────────────┴─────────────────────────────────────────────────────────────────────────┘