# Chapter 12 — Deployment and containerisation

Grounding for **Area 7**. Source: *Building Generative AI Services with FastAPI*, Ch 12. This covers the containerisation detail. The earlier-chapter comparison of deployment targets (VMs, cloud functions, managed app services, containers) is not in the provided text, so grade that choice lightly until it is grounded.

## Container security (the real fails)

- **Run as a non-root user.** Docker runs containers as root by default, giving the container full read/write to mounted host directories and writing root-owned files to the host. More importantly it is a security risk: a compromised container or malicious image then has root access to the host. Add a non-root `USER`.
- **No secrets baked into image layers.** Secrets copied or set via `ENV`/`ARG`, or pulled in by `COPY . .`, persist in layers. Inject secrets at runtime instead.
- **`.dockerignore` is mandatory.** Without it, `COPY . .` drags in caches, virtualenvs, `.env`, and `.git`, bloating the image, caching unnecessary files, and leaking secrets. A typical ignore list: `**/.DS_Store`, `**/__pycache__`, `**/.mypy_cache`, `**/.venv`, `**/.env`, `**/.git`.

## Image size and build optimisation (efficiency, not security)

Large images are slower to build, run, and test, and cost more. The book shows a typical image dropping from 1.42 GB to 34 MB through these steps. Flag bloat as a weakness, not a security failure.

- **Minimal base image.** Use `python:3.x-slim` (Debian-based, better for installing packages, ~186 MB) or `python:3.x-alpine` (smallest, ~71 MB, but limited Python package support). Rule of thumb: slim if you care about build time, Alpine if you care about size. Avoid full distributions and avoid `latest` in production.
- **Avoid GPU inference runtimes where possible.** GPU libraries are huge (around 3 GB of NVIDIA packages plus 1.6 GB for torch). If CPU inference is acceptable, the ONNX runtime with INT8 quantization (see Ch 10) can cut image size up to roughly 10x (5-10 GB down to ~0.5 GB). Only relevant when self-hosting models.
- **Externalise application data and models.** Copying large models into the image inflates build time and size. Use volumes in development and external storage or Kubernetes persistent volumes in production, loading models at startup. Caveat: slow startup downloads can trip health checks and get the container killed, so lengthen the probe or, as a last resort, bake the model in.
- **Layer ordering and caching.** Each `ENV`/`COPY`/`ADD`/`RUN` adds a layer; `WORKDIR`/`ENTRYPOINT`/`LABEL`/`CMD` only touch metadata. A changed layer invalidates it and all later layers. Order from most stable to most volatile: copy `requirements.txt` and `pip install` **before** `COPY . .`, so a code change does not trigger a full dependency reinstall.
- **Minimise layers.** Combine `RUN apt-get update && apt-get install -y ...` into one instruction.
- **Keep the build context small** with `.dockerignore` (also reduces cache invalidation).
- **Cache and bind mounts.** Bind mounts include files for a single `RUN` without persisting as layers; cache mounts persist across builds (for example a Hugging Face model cache at `/root/.cache/huggingface`). External cache (`--cache-from`/`--cache-to` with `buildx`) speeds up ephemeral CI/CD builders.
- **Multi-stage builds.** Split into stages and copy only the needed artifacts forward, leaving build tooling out of the production image. A common pattern is a shared base, then a slim production stage, then a development stage that adds tooling and hot reload, selected with `--target`. This is what takes the image down to tens of megabytes.

## Storage and persistence

The container's writable layer is **ephemeral**: data written to it is lost on stop, restart, or removal. Persist runtime data and logs with volumes, bind mounts, or a database. Note that bind mounts replace the container's permissions with the host's, which is a common source of permission bugs.

## Grading calibration (Area 7)

- Treat non-root user, no baked-in secrets, and a `.dockerignore` as security checks that should pass for any production or external service.
- Treat base-image choice, multi-stage builds, layer ordering, and image size as efficiency: flag bloat and missed optimisations, but do not fail the area on size alone.
- Model externalisation, ONNX, and quantization apply only when the service self-hosts models; ignore them for hosted-API services.
- Persistence (volumes vs ephemeral layer) matters only if the service writes data or logs it needs to keep.
