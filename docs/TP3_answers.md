# TP3 answers: Containers

## Exercise 1
(a) scoreboard:local is 181 MB. python:3.12-slim is 119 MB.
(b) Two things in the build stage but not in the runtime image: the source files (src/, pyproject.toml, requirements.txt) and the leftovers of the build (build and egg-info folders).
(c) Fewer files means less for an attacker to read or abuse. Without the build leftovers, the image is smaller and has a smaller attack surface.
(d) In the second build, the steps up to pip install -r requirements.txt were CACHED. Only COPY src/ and the steps after it ran again. The requirements rarely change, so Docker reuses the slow install. If src/ came first, every code change would reinstall everything. The rebuild took 16.7 s instead of 70.5 s.

## Exercise 2
(a) The named volume -v sbdata:/app/data. The database is written there.
(b) Docker manages the volume and gives it the right owner, so the non-root user can write to it. A host folder belongs to my user and would give "permission denied". It also exposes my real files to the container.
(c) --cap-drop ALL: the attacker has no special Linux powers, like raw network access or changing file owners. --security-opt no-new-privileges: the attacker cannot get more rights through a setuid program.

## Exercise 3
(a) sb: 1.03 s. sb2: 10.46 s.
(b) docker stop sends SIGTERM to PID 1. In sb, PID 1 is uvicorn, so it stops cleanly right away. In sb2, PID 1 is a shell (sh) that does not pass SIGTERM to Python. Docker waits 10 seconds, then kills it with SIGKILL.
(c) Twenty replicas times about 10 s is over 3 minutes instead of about 20 s. Each forced kill can also cut requests in progress.

## Exercise 4
(a) 1 finding (HIGH): zlib1g, CVE-2026-85091. It is a Debian system package.
(b) pip-audit only checks the Python packages in requirements.txt. zlib1g comes from the base image. Trivy scans the whole image, including the operating system.
(c) A fixed version gives the same results every time. It also stops a changed or hacked "latest" scanner from entering my pipeline by surprise.

## Exercise 5
(a) Git commit -> workflow (github.sha) -> build-args BUILD_SHA -> Dockerfile ARG/ENV BUILD_SHA -> /version
(b) latest is a moving tag, so it does not say which version runs. A server that already has latest will not pull the new one, so replicas can run different code under the same name. Rolling back is also impossible.
