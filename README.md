# DevOps Interview Task (Part 1): Containerize

**This is a take-home task.** Please submit by the date given when the task was
issued.

## Overview

You are given a small web application (`app/`). A working `Dockerfile` is
provided, but it was written quickly and does **not** follow container best
practices. Your job is to refactor it into a production-quality image and push
it to a container registry.

You do **not** need to modify the application code.

## What we provide

Clone this repository into your local account.

> **Do Not** fork this repository. Clone it and re-push it into your local account.

The following is provided as part of this repository:

```text
Dockerfile            # a working but non-production Dockerfile — improve it
app/
  app.py              # a minimal Python Flask web app (do not modify)
  requirements.txt    # the app's dependencies (do not modify)
```

The application:

-   Listens on the port given by the `PORT` environment variable (default `8080`).
-   Reads a `GREETING` environment variable used in its response.
-   Exposes `GET /`, `GET /healthz`, and `GET /info`.

## Container notes

The application code in `app/` is unchanged.

Dockerfile changes:

-   Multi-stage build. Dependencies are installed into a virtualenv in a builder
    stage, so build-time artefacts cannot reach the runtime image.
-   `python:3.12-slim` instead of the full `python:3.12`, pinned by digest.
-   `build-essential`, `gcc`, `curl` and `vim` dropped. Flask and gunicorn are
    pure Python wheels, so nothing needs compiling.
-   `requirements.txt` is copied before the application code, so a code change
    does not invalidate the cached dependency layer.
-   Runs as a non-root user with a fixed numeric UID/GID of 10001. Numeric,
    because the Kubernetes `runAsNonRoot` check cannot verify a username.
-   `HEALTHCHECK` using the standard library rather than installing `curl`.
-   OCI labels, including `org.opencontainers.image.source`.
-   Exec-form `CMD`, so gunicorn is PID 1 and receives `SIGTERM` directly.

Supporting changes:

-   `.dockerignore` denies everything by default and re-includes only `app/` and
    `gunicorn.conf.py`.
-   `gunicorn.conf.py` reads the bind port from `PORT` and the worker count from
    `WEB_CONCURRENCY`, defaulting to 2 as before.
-   `.github/workflows/build-push.yml` builds a multi-arch image and publishes it
    to GHCR when a `v*` tag is pushed.

Image size drops from 1.74 GB to 230 MB, and HIGH or CRITICAL vulnerabilities
reported by Trivy from 413 to 44.

## Published image

```text
ghcr.io/vinay1999-roxzz/sample-app:v1
```
