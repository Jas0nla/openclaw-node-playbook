# Troubleshooting

## Symptom: node is paired and connected, but exec still fails

Check these separately:

1. Gateway and node pairing
2. `tools.exec.*` policy
3. Host approvals on the gateway
4. Host approvals on the node
5. Runtime safe-binding shape of the command

## Symptom: UI `Update now` does nothing

If the gateway is running in Docker with a pinned image tag:

- UI update is not authoritative
- the host `.env` image tag wins

Fix from the host deployment:

1. update image tag in `.env`
2. `docker compose pull`
3. `docker compose up -d`

## Symptom: `approval cannot safely bind this interpreter/runtime command`

This is not the same as ordinary approval prompting.

It means OpenClaw runtime refused to bind approval to the command form itself.

Typical triggers:

- `sh -lc ...`
- `bash -lc ...`
- inline interpreter/runtime wrappers
- ambiguous loader paths
- multi-file runtime forms

## What to try first

1. test a very small direct command like `id`
2. avoid shell wrappers
3. prefer node-native capabilities like `system.which`
4. move browser work to `browser.proxy`

## Symptom: `browser control disabled`

Split this into two branches:

1. gateway-side browser config
2. node-side browser plugin availability

In this environment, the failure persisted even after:

- `browser.enabled=true`
- gateway browser routing pointed at `jason-mac`

The real blocker was on the node machine:

- local config had `plugins.allow=["telegram"]`
- that excluded the bundled `browser` plugin

Fix on the node machine:

1. ensure `browser.enabled=true`
2. ensure `plugins.allow` includes `browser` or remove the restrictive allowlist
3. ensure `plugins.entries.browser.enabled=true`
4. restart the node host

Then re-test from the gateway with:

1. `openclaw browser status`
2. `openclaw browser open https://example.com`

## What not to assume

- Do not assume a connected node means shell automation is ready
- Do not assume approvals files are the final gate
- Do not assume a shell wrapper will behave like a plain executable
