# Docker Hub automation for this fork

This fork keeps application source in GitHub and publishes deployment images to Docker Hub. NAS configuration and credentials are runtime files, never part of the image.

## Source and remotes

- `origin`: `https://github.com/elconcept/mcp-server-synology.git`
- `upstream`: `https://github.com/atom2ueki/mcp-server-synology.git`
- Branch: `main`.
- The working checkout belongs in `/workspace/mcp-server-synology` (the NAS share is `/volume1/workspace`).
- Keep deployment Compose, configuration and logs in `/volume2/docker/mcp-synology`. The deployment uses `image:` only, with no `build:` or source bind mount.

## Automation

`.github/workflows/homelab-dockerhub.yml` runs on pushes to main, manual dispatch and `17 */6 * * *` UTC. Scheduled and manual runs merge upstream main without force pushes. A conflict stops the update without replacing local changes.

The selected post-merge SHA is passed to isolated test and image jobs. Unit tests exclude live NAS and destructive tests. Publishing does not depend on a second push event from GitHub's token.

Images support `linux/amd64` and `linux/arm64`. Tags are `sha-<full-commit-SHA>` and `latest`. An existing commit image is reused; a missing image is rebuilt on the next successful run even if upstream has not changed. Latest is promoted only while the selected SHA is still main.

Docker Hub tag immutability, if enabled, should cover only `^sha-[0-9a-f]{40}$`, leaving latest mutable. A digest identifies exact image bytes regardless of tag settings.

## Repository configuration

| Setting | Kind | Purpose |
|---|---|---|
| DOCKERHUB_USERNAME | Variable | Authenticated Docker Hub account |
| DOCKERHUB_IMAGE | Variable | Docker Hub namespace/repository |
| DOCKERHUB_TOKEN | Secret | Docker Hub token with push access |
| UPSTREAM_SYNC_SSH_KEY | Secret | Dedicated write-enabled SSH deploy key for this fork only |

The synchronization job uses a dedicated SSH deploy key restricted to this fork, including workflow updates. It does not store a personal GitHub PAT in the fork. Only the prepare job receives this key; test and publication jobs have read-only GitHub access. SSH host verification remains enabled.

The upstream GHCR and PyPI publication workflows are retained for merge compatibility and disabled in this fork. Upstream tests remain available. Do not copy upstream tags merely to update application code.

GitHub can delay scheduled runs, and public-repository schedules can be disabled after prolonged repository inactivity. Check the Actions page if scheduled runs stop; workflow_dispatch is also available. No NAS credentials are needed by these jobs.

## Deployment

Pull the published image and update the existing HTTP deployment. Preserve its port 8765, /mcp path, external Docker network, allowed hosts/origins and read-only configuration mount. Do not substitute the upstream stdio Compose for the existing HTTP deployment.

After redeployment verify container state, HTTP MCP initialization, portal tool discovery, and a read-only DSM operation. Keep the previous image reference until verification succeeds.
