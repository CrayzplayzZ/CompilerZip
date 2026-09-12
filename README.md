# ZIP TO JAR COMPILER

A real server-side Java/Minecraft project compiler. It does **not** rename ZIP files.

## Requirements

- Node.js 18+
- Python 3 (used only for safe ZIP parsing/extraction)
- A supported JDK installed on the server
- Gradle installed for projects without a Gradle wrapper
- Maven installed for Maven projects

For Minecraft projects, the uploaded project's own Gradle configuration determines Minecraft/loader/dependencies. The server does not hard-code a Minecraft version.

## Install

```bash
npm install
npm start
```

Then open `http://SERVER:3000`.

## Configuration

Environment variables:

- `PORT` — default `3000`
- `MAX_UPLOAD_BYTES` — default 250 MiB
- `MAX_EXTRACTED_BYTES` — default 1 GiB
- `BUILD_TIMEOUT_MS` — default 15 minutes
- `JOB_RETENTION_MS` — default 30 minutes
- `PYTHON` — Python executable, default `python3`

## Java versions

The safest production setup is to install the JDKs required by the projects you intend to support (commonly 8/17/21). Projects using Gradle toolchains can select their own configured JDK when those JDKs are installed. Do not download arbitrary JDKs at build time.

For a stronger multi-tenant deployment, run builds in OS/container sandboxes with CPU, memory, PID, network and disk quotas. This example already isolates each build into a unique directory and performs traversal/extraction checks, but a container/VM boundary is recommended for an internet-facing service.

## Security notes

Uploaded ZIPs are untrusted. The app:

- rejects non-ZIP extensions
- applies upload and extraction-size limits
- checks ZIP integrity
- blocks absolute paths and `..` traversal
- extracts into a per-build directory
- never serves build directories as static files
- validates the build ID before lookup/download
- only exposes a generated JAR after the real command exits with code 0 and the output has a JAR/ZIP magic header
- cleans temporary build files automatically

### Important production hardening

A Gradle/Maven build is arbitrary code execution by design: build scripts can execute commands and can access the network. Therefore **do not expose this service directly to the public internet without a container/VM sandbox**. Use a dedicated unprivileged build user, read-only base filesystem, isolated workspace, resource limits, outbound-network policy, process/PID limits, and an external job queue for serious multi-user deployment.

The backend never claims success unless the actual build command exits successfully and a real JAR is found and verified.
