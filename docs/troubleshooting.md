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

## Symptom: SSH and UI both become slow after UI maintenance work

Do not assume the gateway is the only culprit.

In this environment, a more likely cause is leftover build or test workers in the source tree on the Pi.

Check:

```bash
ps -ef | egrep 'vitest|vite' | grep -v grep
```

Common offenders:

- `vitest/dist/workers/forks.js`
- `node ./node_modules/.bin/vitest run ...`
- `vite build`

If these remain after a maintenance run, kill them before continuing and then re-check:

1. SSH responsiveness
2. gateway `/healthz`
3. control UI page load

## Symptom: UI patching made the frontend inconsistent

Do not hot-edit `index-*.js` files in the live container as the normal fix path.

Use this sequence instead:

1. edit source
2. rebuild `dist`
3. back up `/app/dist/control-ui`
4. replace the deployed `control-ui` directory
5. restart the gateway
6. validate `/healthz` and `/`

If the UI becomes blank after deployment, restore the previous `control-ui` backup and restart the gateway.

## Symptom: Docker memory limits appear configured but do not apply

If `docker compose up -d` prints warnings like:

- `Your kernel does not support memory limit capabilities`
- `Limitation discarded`

then the Pi kernel/cgroup setup is not enforcing container memory caps.

In that case:

1. keep `mem_limit` and `mem_reservation` in compose as documentation of the intended budget
2. do not assume Docker will actually enforce them
3. rely on `NODE_OPTIONS=--max-old-space-size=1024` and operational discipline to avoid runaway memory usage
4. treat stray `vitest` or `vite` workers as a higher-priority risk than the steady-state gateway

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
