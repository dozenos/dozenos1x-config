# AGENTS.md

## Project purpose
OCaml library that parses, manipulates, and serializes DozenOS 1.x and EdgeOS configuration files. The data layer that the rest of the DozenOS config stack is built on.

## Tech stack
- OCaml; `dune` 2.0 with `menhir` (parser); `ppx_deriving_yojson`, `yojson` for JSON I/O.
- `opam`: `dozenos1x-config.opam` declares version `0.3`, MIT license, deps `ocamlfind`, `menhir`, `dune >= 1.4.0`, `ppx_deriving_yojson <= 3.9.1`, `yojson`.
- License: LGPL-2.1 in tree (`LICENSE`); opam declares `MIT` (note the discrepancy if redistributing).

## Build / test / run
- Local: `opam install . --deps-only` then `dune build -p dozenos1x-config`.
- Install: `dune install` (or via opam-pin).
- No `dune runtest` suite in tree; coverage comes from `libdozenosconfig` and `dozenos-1x` smoketests downstream.

## Repository layout
- `src/` — OCaml sources (parser via menhir, AST, serializers) plus `lexical_numeric_compare.c` (small C helper).
- `dune-project`, `dozenos1x-config.opam` — build/package metadata.
- `.github/workflows/` — `check-pr-conflicts.yml`, `cla-check.yml`, `pr-mirror-repo-sync.yml` — all delegate to `dozenos/.github` reusables.

## Cross-repo context
- Wrapped by an internal repository (OCaml→C shim producing `libdozenosconfig0.so`) which is also vendored inside `dozenos/dozenos-1x/libdozenosconfig/` as the canonical build source for the Debian package.
- Consumed (via `libdozenosconfig0`) by `dozenos/dozenos-1x`'s `python/dozenos/configtree.py` (ctypes binding) and by the legacy C++ Vyatta layer.
- `dozenos/vyconf` (config-session daemon, OCaml) depends on this library directly.
- Live consumer of the generation-1 mirror pipeline (`pr-mirror-repo-sync.yml@current`).

## Conventions
- Commit / PR title: `component: T12345: description` (Phorge ID mandatory).
- Default branch `current`; backports via `@Mergifyio backport <branch>`.
- Reusable workflows pinned `@current` — changes in `dozenos/.github` ship immediately.

## Notes for future contributors
- This is a load-bearing library — every `commit` operation against DozenOS goes through it. Add tests at the `libdozenosconfig`/`dozenos-1x` layers, not just locally.
- Keep parser changes backwards compatible; existing configs and migration scripts depend on round-trip stability.
- LICENSE (LGPL-2.1) vs opam (MIT) mismatch is worth resolving before any non-trivial relicensing scenario.
- `vyconf` (still early-stage per its README) is a fellow OCaml consumer; coordinate API changes with that repo.
