# Dockerfile Generator Prompt

## 🎯 Zweck

Dieser Prompt hilft dem Cursor Agent dabei, optimierte Dockerfiles für verschiedene Programmiersprachen und Frameworks zu generieren. Der Agent soll Multi-Stage Builds, Security Best Practices und Performance-Optimierungen implementieren.

## 📋 Prompt für den Agent

### Kontext
Du bist ein Experte für Container-Entwicklung und sollst ein optimiertes Dockerfile für ein bestehendes Projekt erstellen. Das Projekt wurde analysiert und die folgenden Informationen sind verfügbar:

**Projekt-Informationen:**
- Programmiersprache: `{LANGUAGE}`
- Framework: `{FRAMEWORK}`
- Port: `{PORT}`
- Package Manager: `{PACKAGE_MANAGER}`
- Build Command: `{BUILD_COMMAND}`
- Start Command: `{START_COMMAND}`
- Dependencies: `{DEPENDENCIES}`

### Aufgabe
Erstelle ein Multi-Stage Dockerfile mit folgenden Anforderungen:

#### 1. Multi-Stage Build
- **Build Stage:** Dependencies installieren, Code kompilieren, Tests ausführen
- **Production Stage:** Nur Runtime und kompilierter Code, minimale Base Images

#### 2. Security Best Practices
- Non-root User verwenden
- Minimale Base Images (Alpine Linux bevorzugt)
- Keine Build-Tools in Production Image
- Explizite Versionen für alle Images

#### 3. Performance Optimierungen
- Layer Caching optimieren (Dependencies vor Code kopieren)
- .dockerignore Datei erstellen
- Minimale Image-Größe
- Health Checks implementieren

#### 4. Framework-spezifische Optimierungen
Basierend auf der Programmiersprache:

**Node.js:**
- npm ci für deterministische Builds
- node_modules Caching
- Production Dependencies nur

**Python:**
- pip install --no-cache-dir
- Virtual Environment
- Requirements.txt optimieren

**.NET Core:**
- dotnet restore mit Caching
- dotnet publish für optimierte Builds
- Runtime-only Image

**Java:**
- Maven/Gradle Layer Caching
- JRE-only Runtime Image
- JVM Optimierungen

### Output-Format
Erstelle folgende Dateien:

1. **Dockerfile** - Multi-Stage Build
2. **.dockerignore** - Ignore-Patterns
3. **Dockerfile.optimized** - Alternative mit zusätzlichen Optimierungen

### Beispiel-Struktur

```dockerfile
# Multi-Stage Build für {LANGUAGE} {FRAMEWORK}
FROM {BASE_IMAGE} AS builder

# Build Stage
WORKDIR /app
COPY package*.json ./
RUN {INSTALL_DEPENDENCIES}
COPY . .
RUN {BUILD_COMMAND}

# Production Stage
FROM {RUNTIME_IMAGE}
WORKDIR /app

# Non-root User
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Copy built application
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --chown=nextjs:nodejs package*.json ./

USER nextjs

# Health Check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:{PORT}/health || exit 1

EXPOSE {PORT}
CMD ["{START_COMMAND}"]
```

## 🔧 Framework-spezifische Templates

### Node.js Express
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
RUN addgroup -g 1001 -S nodejs && \
    adduser -S express -u 1001
COPY --from=builder --chown=express:nodejs /app/dist ./dist
COPY --from=builder --chown=express:nodejs /app/node_modules ./node_modules
COPY --chown=express:nodejs package*.json ./
USER express
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

### Python Flask
```dockerfile
FROM python:3.11-alpine AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN python -m compileall .

FROM python:3.11-alpine
WORKDIR /app
RUN addgroup -g 1001 -S python && \
    adduser -S flask -u 1001
COPY --from=builder --chown=flask:python /app .
USER flask
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1
EXPOSE 5000
CMD ["python", "-m", "flask", "run", "--host=0.0.0.0"]
```

### .NET Core
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS builder
WORKDIR /app
COPY *.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:7.0
WORKDIR /app
RUN addgroup -g 1001 -S dotnet && \
    adduser -S app -u 1001
COPY --from=builder --chown=app:dotnet /app/out .
USER app
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
EXPOSE 8080
CMD ["dotnet", "app.dll"]
```

## 📝 .dockerignore Templates

### Allgemein
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.vscode
.idea
*.log
dist
build
```

### Node.js spezifisch
```
node_modules
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.npm
.yarn-integrity
.env.local
.env.development.local
.env.test.local
.env.production.local
```

### Python spezifisch
```
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env
pip-log.txt
pip-delete-this-directory.txt
.tox
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.log
.git
.mypy_cache
.pytest_cache
.hypothesis
```

## 🎯 Agent-Anweisungen

### Für den Cursor Agent:

1. **Analysiere das Projekt:** Verstehe die Programmiersprache, Framework und Dependencies
2. **Wähle das passende Template:** Verwende das Framework-spezifische Template
3. **Passe die Variablen an:** Ersetze alle Platzhalter mit den tatsächlichen Werten
4. **Optimiere für das Projekt:** Füge projekt-spezifische Optimierungen hinzu
5. **Erstelle .dockerignore:** Generiere eine passende .dockerignore Datei
6. **Validiere das Dockerfile:** Überprüfe auf Syntax und Best Practices

### Beispiel-Interaktion:

```
Agent: Ich erstelle ein Dockerfile für euer Node.js Express Projekt.
       Ich verwende Multi-Stage Build mit Alpine Linux für optimale Performance.

User: Das klingt gut. Könntest du auch Health Checks hinzufügen?

Agent: Natürlich! Ich füge Health Checks für den /health Endpoint hinzu.
       Das Dockerfile ist jetzt bereit. Soll ich es in euer Projekt einfügen?

User: Ja, bitte!

Agent: Perfekt! Ich habe das Dockerfile und die .dockerignore Datei erstellt.
       Das Image wird etwa 80% kleiner sein als ein Standard-Image.
```

## 🔍 Validierung

Nach der Generierung soll der Agent folgende Punkte überprüfen:

- [ ] **Multi-Stage Build:** Build und Production Stages getrennt
- [ ] **Security:** Non-root User, minimale Base Images
- [ ] **Performance:** Layer Caching optimiert
- [ ] **Health Checks:** Liveness und Readiness Probes
- [ ] **Size:** Image-Größe minimiert
- [ ] **Syntax:** Dockerfile-Syntax korrekt
- [ ] **Best Practices:** Alle Best Practices befolgt

---

*Dieser Prompt ist darauf ausgelegt, mit dem Cursor Agent verwendet zu werden, um optimierte Dockerfiles für verschiedene Programmiersprachen und Frameworks zu generieren.*
