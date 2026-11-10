# Provenance: audited ARTEX snapshot

This repository is a **pinned, unmodified snapshot** of [BaaRai/ARTEX](https://github.com/BaaRai/ARTEX)
(itself a fork of [Autumn-27/ARTEX](https://github.com/Autumn-27/ARTEX)), published so it can be shared and
deployed from a known-good point. It is licensed under the AGPL-3.0, like upstream (see `LICENSE`).

## What exactly was reviewed

| Item | Value |
|---|---|
| ARTEX source | `BaaRai/ARTEX` commit **`b3717f7`** ("release: v0.4.0"), tagged here as **`audited-v0.4.0`** |
| Agent engine | `github.com/BaaRai/Norma v0.4.4` (via the `replace` in `go.mod`) |
| Engine checksum | `h1:x5UAeaTuhREI1GiIYuwIEybRxr/pLCeCUvXmUbwv5Is=` (from `go.sum`) |

The only file added on top of the audited commit is this one. Everything else is byte-identical to upstream.

## Review summary

A static review (the code was not executed during review) checked for these deal-breakers:

- hardcoded callback or command-and-control hosts
- exfiltration of API keys, scan results or evidence anywhere other than the configured LLM endpoint and the target
- obfuscated or encoded executable blobs
- malicious install, build or CI hooks (phone-home, remote code fetched and run)
- changes in the fork that alter egress, authentication or key handling

**Result: none found** in ARTEX or in the Norma engine, which was confirmed identical to its upstream. The
in-app automatic update was removed upstream in commit `2d5f967`, so the binary never replaces itself. The
`skills/` tree is plain-text methodology prompts only, with no binaries or payloads.

## Important: it is an offensive tool by design

ARTEX gives its agent an unsandboxed shell and has no built-in egress control. That is intended behavior, not
a backdoor, but it means **you must contain it yourself**. Recommended baseline:

- run it in an isolated container or VM, never directly on a workstation or a shared server
- restrict outbound traffic to the LLM API and the authorized target scope (for example a proxy allowlist
  plus firewall rules that block private ranges)
- put the dashboard behind your own authentication
- use a dedicated, spend-capped LLM API key
- only point it at systems you are authorized to test

## Verify it yourself

```bash
git clone https://github.com/BaaRai/ARTEX upstream && cd upstream
git remote add audited https://github.com/dalr0p/artex-audited && git fetch audited --tags
git rev-parse audited-v0.4.0^{commit}      # expect: b3717f7... (same commit as upstream)
git diff b3717f7 audited/main --stat      # expect: only PROVENANCE.md added
go mod download && go mod verify         # checks the engine against go.sum
```
