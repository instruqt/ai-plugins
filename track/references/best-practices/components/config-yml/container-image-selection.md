# Container Image Selection

Evaluates whether the chosen container or VM image is appropriate for the track's workload, balancing pre-installed tooling against image size and startup time.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Image missing critical tools the track requires, or using a massive general-purpose image for a task that needs one CLI tool |
| 2 | Below Standard | Image mostly works but requires substantial setup-script patching to install missing tools, or uses cloud-client when no cloud CLIs are needed |
| 3 | Adequate | Image is functional but not ideal -- e.g., using cloud-client for a pure Kubernetes exercise, or shell image when cloud CLIs are referenced in assignments |
| 4 | Good | Correct image for the use case; aware of environment variable differences between images; setup scripts only add track-specific tools (production baseline) |
| 5 | Excellent | Minimal image that includes exactly what is needed -- no unnecessary tools, fast startup, setup scripts are lean |

## Guidance

Instruqt provides two primary container images:

- **gcr.io/instruqt/shell** -- Lightweight shell environment. Includes common CLI tools (curl, jq, git, vim, etc.) but no cloud provider CLIs. Use for exercises focused on general Linux, Docker, Kubernetes, or vendor-specific tools you install in setup.
- **gcr.io/instruqt/cloud-client** -- Heavier image with AWS CLI, Azure CLI, and Google Cloud SDK pre-installed. Use when the track exercises require interacting with cloud provider APIs.

Key differences between these images:

- Environment variables and shell profiles differ -- scripts that source `.bashrc` may behave differently
- cloud-client is significantly larger, increasing sandbox startup time
- cloud-client includes Python 3 and pip; shell does not always

When to use upstream images:

- **node:22, python:3.11, golang:1.22** -- when the track is about that language/runtime and you need a clean, version-specific environment
- **hashicorp/terraform, hashicorp/vault** -- when you want the official tool image and will add nothing else
- **Private registry images** -- for vendor-specific pre-built environments; ensure the registry credentials are configured in the org settings

Good -- lightweight image for a Kubernetes exercise:

```yaml
containers:
- name: shell
  image: gcr.io/instruqt/shell
  shell: /bin/bash
```

Good -- cloud-client for an AWS exercise:

```yaml
containers:
- name: shell
  image: gcr.io/instruqt/cloud-client
  shell: /bin/bash
```

Good -- upstream image for a language-specific exercise:

```yaml
containers:
- name: dev
  image: node:22
  shell: /bin/bash
```

Bad -- cloud-client for a track that never touches cloud APIs:

```yaml
# Track only teaches git branching -- no cloud CLIs needed
containers:
- name: shell
  image: gcr.io/instruqt/cloud-client
  shell: /bin/bash
```

Bad -- shell image when the track requires AWS CLI:

```yaml
# Track exercises use 'aws s3 ls' but image lacks AWS CLI
containers:
- name: shell
  image: gcr.io/instruqt/shell
  shell: /bin/bash
```

Bad -- setup script reinstalling tools the image already has:

```yaml
# Using cloud-client but setup.sh runs 'pip install awscli'
containers:
- name: shell
  image: gcr.io/instruqt/cloud-client
  shell: /bin/bash
```

## What to Watch For

- Setup scripts that install large packages (cloud CLIs, language runtimes) that could be avoided by choosing the right base image
- Using cloud-client when the track never references AWS, Azure, or GCP -- adds startup time for no benefit
- Using shell when assignment code blocks reference `aws`, `az`, or `gcloud` commands
- Upstream images that lack basic tools (curl, jq, vim) the learner needs for the exercise -- plan setup script additions
- Private registry images without verifying org-level registry credentials are configured
- Version-pinned upstream images (good: `node:22`) vs floating tags (risky: `node:latest`) -- prefer pinned versions for reproducibility
