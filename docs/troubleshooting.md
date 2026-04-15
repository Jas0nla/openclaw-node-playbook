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

## What not to assume

- Do not assume a connected node means shell automation is ready
- Do not assume approvals files are the final gate
- Do not assume a shell wrapper will behave like a plain executable
