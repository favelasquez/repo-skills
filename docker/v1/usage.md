---
name: docker
description: Detecta el stack del proyecto y genera Dockerfile y .dockerignore optimizados
license: Apache-2.0
metadata:
  author: transportationamerica-setup
  version: "1.0"
scope:
  - docker
  - dockerfile
  - devops
  - containers
permissions:
  allow:
    - filesystem
    - repositories
  deny:
    - database
---

# Docker v1 — Dockerfile Generator

You are a Docker expert. When this skill is invoked, **execute the full Docker setup flow** for the current project.

## Flow to execute

### 1. Detect the stack
Scan the project root for the following files to identify the technology and runtime version:

| File | Technology |
|------|-----------|
| `package.json` | Node.js — read `engines.node` or detect from `.nvmrc`, `.node-version` |
| `angular.json` | Angular (Node.js base) |
| `requirements.txt` / `pyproject.toml` / `setup.py` | Python — check `python_requires` or `.python-version` |
| `*.csproj` / `*.sln` | .NET — read `<TargetFramework>` tag |
| `go.mod` | Go — read `go X.XX` directive |
| `pom.xml` / `build.gradle` | Java — check `<java.version>` or `sourceCompatibility` |
| `Gemfile` | Ruby — check `ruby 'X.X.X'` |
| `Cargo.toml` | Rust — check `rust-version` or use `stable` |

Run the following commands to confirm installed versions:
- Node.js: `node --version`
- Python: `python --version` or `python3 --version`
- .NET: `dotnet --version`
- Go: `go version`
- Java: `java -version`

### 2. Ask for the port

Before generating the Dockerfile, use **AskUserQuestion** to ask the user which port the application should expose:

> "¿En qué puerto corre tu aplicación? (Ej: 3000, 8080, 4200)"

Use the user's answer as `<port>` throughout the rest of the flow. If the user does not provide a value, fall back to the stack default (Node.js → 3000, Python → 8000, .NET → 8080, Go → 8080).

### 3. Generate the Dockerfile

Create a **multi-stage, production-ready** `Dockerfile` based on the detected stack.

#### Node.js / Angular
```dockerfile
# Build stage
FROM node:<VERSION>-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
RUN npm run build

# Production stage
FROM node:<VERSION>-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

> For Angular (static output), replace the production stage with `nginx:alpine` and copy `dist/<project-name>/` to `/usr/share/nginx/html`.

#### Python
```dockerfile
FROM python:<VERSION>-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "-m", "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### .NET
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:<VERSION> AS builder
WORKDIR /app
COPY *.csproj ./
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:<VERSION>
WORKDIR /app
COPY --from=builder /app/out .
EXPOSE 8080
ENTRYPOINT ["dotnet", "<ProjectName>.dll"]
```

#### Go
```dockerfile
FROM golang:<VERSION>-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]
```

### 4. Generate .dockerignore

Always create a `.dockerignore` file:

```
node_modules
dist
.git
.gitignore
*.md
.env
.env.*
*.log
coverage
.DS_Store
```

Adapt it to the detected stack (e.g. add `__pycache__`, `*.pyc` for Python; `bin/`, `obj/` for .NET).

### 5. Ask if Docker Desktop is running

After creating the files, use **AskUserQuestion** with a single yes/no question:

> "¿Docker Desktop está ejecutándose? Si es así, construiré y levantaré el contenedor automáticamente."

Options:
- **Sí, está ejecutándose** — proceed to step 6
- **No / No lo sé** — skip step 6 and go directly to step 7

### 6. Build and run the container (only if user confirmed Docker is running)

Run the following commands sequentially using the Bash tool:

```bash
docker build -t <project-name> .
docker run -d -p <port>:<port> --name <project-name> <project-name>
```

- Use `<project-name>` from `package.json` `name` field (or the folder name if not available).
- Use `<port>` detected from the server source (e.g. `3000`).
- After running, confirm the container started with:
  ```bash
  docker ps --filter "name=<project-name>"
  ```

### 7. Report to the user

Inform the user:
- Detected stack and version used
- Files created (`Dockerfile`, `.dockerignore`)
- If the container was started: URL where the app is running (`http://localhost:<port>`)
- If Docker was not running: how to build and run manually:
  ```bash
  docker build -t <project-name> .
  docker run -p <port>:<port> <project-name>
  ```

## Rules
- Always use **specific version tags** (never `latest`) for the base image
- Always use **multi-stage builds** to minimize the final image size
- Set `ENV NODE_ENV=production` (or equivalent) in production stage
- Never copy `.env` files into the image — use runtime environment variables
- Use `alpine` or `slim` variants when available
