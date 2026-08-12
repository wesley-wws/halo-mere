# JS/TS Calibration Notes (frontend and Node backend)

When to read this file: after detecting a `package.json` with web framework deps (react, vue,
angular, svelte, next, nuxt) or server framework deps (express, nestjs, fastify, koa, hono).

These are **not** a pattern catalog. Feature-Sliced Design, Atomic Design, monorepo tooling,
and the rest are already in the model's training data. What follows are guard rails against
the specific directions the model over-flags, or misses, on this stack.

## Module boundaries are usually enforced by nothing

This is the stack where "module boundary" most often means *no enforcement at all*. Without
TypeScript project references or `workspaces`-declared package boundaries, dependency
direction is convention-only.

Flag this gap explicitly rather than concluding Lens 2 is satisfied because imports happen to
point the right way today. An unenforced boundary holds until the first contributor in a
hurry, and nothing in the build will tell them they crossed it.

## ORM-generated types as domain entities

Prisma or Drizzle types used directly as domain entities is the JS/TS equivalent of EF Core in
a Domain project: it couples domain logic to schema shape and migration cadence. Apply Lens 2
the same way.

## Mixed framework conventions

Next.js `pages/` and `app/` routers coexisting, or multiple state-management libraries with no
stated strategy, are common drift findings. Call them out under Lens 1 even when each pattern
is locally fine, for the same reason as the .NET mixed-pattern case: the missing decision is
the finding, not either pattern.
