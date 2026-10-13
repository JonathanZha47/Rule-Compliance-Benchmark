# vendor/pi — the pi coding agent, pinned

The benchmark's pi strategies (`pi-native-cut`, `pi-native-policy-preserving`) run pi's own
`AgentSession.compact()`. These seven npm tarballs are the pi packages every run of the paper used;
`npm ci` installs them from here, so no pi checkout is needed.

| tarball | package |
|---|---|
| `pi-coding-agent-0.84.3.tgz` | the agent session and its compaction |
| `pi-ai-0.84.3.tgz` | model providers (the OpenRouter / OpenAI-completions path is the one used) |
| `pi-agent-core-0.84.3.tgz`, `pi-client-`, `pi-protocol-`, `pi-tui-`, `pi-telemetry-` | internal dependencies of the above |

`package.json` depends on the coding-agent tarball and uses `overrides` to point every internal pi
dependency at its tarball here, so npm never mixes in a registry version. Checksums are in
[`SHA256SUMS`](SHA256SUMS); `./arcbench preflight` verifies them.

## Where they come from

- **Source:** the open-source pi coding agent (cited in the paper) at commit
  `97fa14e39cfce78c273a36b2d9e8509cd5bc6b72` (`v0.84.3-3-g97fa14e39`, "fix(coding-agent): reject
  truncated compaction summaries"). The package versions still read `0.84.3`.
- **One local edit**, type-only: [`patches/0001-cloudflare-ai-gateway-type-annotation.patch`](patches/0001-cloudflare-ai-gateway-type-annotation.patch)
  adds a generic type argument so the commit compiles. Types are erased at compile time; it changes
  no runtime behaviour, and it touches a provider the benchmark does not call.
- **Generated model catalogs** (`packages/ai/src/providers/*.models.ts`, `data/`) are produced by pi's
  build from live provider listings and are included as they were built. The benchmark does not use
  them: pi's compactor is given an explicit OpenRouter model definition (`src/compaction/pi-native.ts`).
- **Packed** from that build with `npm pack --ignore-scripts`.

Why not the npm registry: `0.84.3` on npm predates the pinned commit's compaction fix, and `0.84.4`
changes pi's compaction requests. Either would make the pi strategies a different configuration
from the one the paper measured.

## Checking that nothing drifted

`tests/golden-replay.test.ts` replays sampled episodes — pi's streamed compaction calls included —
and requires byte-identical model requests to the recorded ones. With these tarballs installed,
the full set (`npm run test:golden-full`, 1,819 episodes) passes.
