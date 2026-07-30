# mcalc

A tiny big-number CLI in pure [Mere](https://merelang.org/), and a live
demonstration of Mere's package manager resolving a **cross-repo transitive
dependency graph**.

```sh
mere install                       # fetch deps into .mere_modules/, write mere.lock
mere mcalc.mere fact 100           # 93,326,215,443,...,000,000
mere -c mcalc.mere > c.c && clang -O2 c.c -o mcalc && ./mcalc fib 200
```

## The dependency graph

```
mcalc  ──▶  github.com/284km/mbigfmt  ──▶  github.com/284km/mbignum
(app)        (thousands grouping)           (arbitrary precision)
```

`mcalc` depends only on **mbigfmt**. It never names **mbignum** — that is a
*transitive* dependency, pulled in because mbigfmt declares it. `mere install`
follows the cross-repo `[dependencies]` chain, installs each package under its
full module path (`.mere_modules/github.com/284km/…/`), and writes a
`mere.lock` that pins **both** packages — including the transitive mbignum —
with their resolved commit and a content hash.

That is exactly what a lockfile is for: `mcalc`'s manifest names one
dependency, but its build is reproducible only if *every* package in the
resolved graph is pinned. The committed `mere.lock` records the whole graph,
and a later `mere install` re-verifies each hash (go.sum-style), failing loudly
if a pinned revision's content ever changes underneath it.
