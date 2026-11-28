# Synology MCP — source, image and deployment

This repository maintains the Synology MCP source. Container images are built and published by the central workflow in the private [elconcept/ai-homelab-vault](https://github.com/elconcept/ai-homelab-vault) repository:

- Workflow: `.github/workflows/images.yml`
- Deployment image: `concept5400/patron:homelab-mcp-synology-latest`
- Source-specific images: `concept5400/patron:homelab-mcp-synology-sha-<source-SHA>`
- Build input: this repository's `Dockerfile`, with the repository root as context.

The central workflow owns Docker Hub authentication and publication. This fork no longer has a Docker Hub publication job or requires Docker Hub credentials.

## Source synchronization and tests

`.github/workflows/homelab-dockerhub.yml` retains its existing filename and now runs only `prepare` and `verify`. It runs after pushes to `main`, on manual dispatch, and every six hours at `17 */6 * * *` UTC.

Scheduled and manual runs merge `atom2ueki/mcp-server-synology:main` into this fork without force pushes. The selected source SHA is then tested on a GitHub-hosted runner. A conflict stops synchronization. Unit tests exclude live NAS and destructive tests.

Only `UPSTREAM_SYNC_SSH_KEY` is needed here for the existing upstream synchronization. Docker Hub configuration belongs to the private vault.

## NAS layout and deployment

- Source checkout: `/volume1/workspace/mcp-server-synology`.
- Compose, runtime configuration and logs: `/volume2/docker/mcp-synology`.
- The NAS pulls the published image and runs the service through its Compose stack.

Keep the HTTP deployment's port `8765`, `/mcp` path, external network, allowed hosts/origins and read-only configuration mount. Runtime DSM credentials stay in the deployment configuration.

After updating the stack, verify MCP initialization, portal tool discovery and a read-only DSM operation.
