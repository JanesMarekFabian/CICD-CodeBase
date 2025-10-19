# CI/CD Pipeline Plan: GitHub Actions + Docker Hub

## 🎯 Übersicht & Zielsetzung

### Was wird erreicht?
**Automatisierung von:** Code-Änderung → Build → Test → Deploy auf lokales Kubernetes Cluster

**Ohne CI/CD müsstest du:**
- Manuell Docker Images bauen
- Manuell zu Docker Hub pushen
- Manuell auf Kubernetes Cluster zugreifen
- Manuell Kubernetes-Befehle ausführen
- Manuell prüfen, ob alles läuft

**Mit CI/CD:** `git push` → GitHub Actions macht alles automatisch in 5-10 Minuten!

### Wann ist diese Lösung sinnvoll?
- ✅ **Kostenlos** für Open Source und kleine Teams
- ✅ **Einfach zu verstehen** und zu implementieren
- ✅ **Open Source** - keine Vendor Lock-in
- ✅ **Große Community** - viele Beispiele und Hilfe
- ✅ **Flexibel** - funktioniert mit jedem Kubernetes Cluster

### Vor- und Nachteile

**Vorteile:**
- Kostenlos (2000 Min/Monat GitHub Actions)
- Docker Hub unbegrenzt public repos kostenlos
- Einfache Integration mit GitHub
- Große Community und Dokumentation
- Keine Vendor Lock-in

**Nachteile:**
- Docker Hub Rate Limits bei vielen Pulls
- Keine Enterprise-Features (Advanced Security, etc.)
- Public Images sichtbar für alle
- Begrenzte Build-Minuten im Free Tier

---

## 🏗️ Architektur-Diagramm

```
Developer (Lokal)
    ↓ git push
GitHub Repository
    ↓ Trigger (Push/PR)
GitHub Actions Runner
    ↓
[1] Code auschecken
[2] Dependencies installieren
[3] Code kompilieren/bauen
[4] Tests ausführen
[5] Docker Image bauen
    ↓
Docker Hub (Image speichern)
    ↓
[6] kubeconfig konfigurieren
[7] Kubernetes Deployment aktualisieren
[8] Rollout überwachen
    ↓
Kubernetes Cluster (lokal)
    ↓
[9] Pods starten mit neuem Image
[10] Health Checks durchführen
```

### Netzwerk-Kommunikation
- **GitHub Actions → Docker Hub:** HTTPS (Image Push)
- **GitHub Actions → Kubernetes:** HTTPS (kubectl API)
- **Kubernetes → Docker Hub:** HTTPS (Image Pull)
- **Lokaler Cluster:** 192.168.2.x Netzwerk

---

## 📦 Komponenten-Übersicht

### 1. GitHub Repository (Code-Speicher)
- **Zweck:** Zentrale Code-Verwaltung
- **Features:** Version Control, Branching, Pull Requests
- **Trigger:** Push, Pull Request, Tag, Schedule

### 2. GitHub Actions (CI/CD Engine)
- **Zweck:** Automatisierungs-Engine
- **Laufzeit:** GitHub-gehostete Runner (Ubuntu, Windows, macOS)
- **Free Tier:** 2000 Minuten/Monat
- **Workflows:** YAML-basierte Konfiguration

### 3. Docker Hub (Image Registry)
- **Zweck:** Container Image Speicherung
- **Free Tier:** Unbegrenzt public repositories
- **Rate Limits:** 100 Pulls/6h für anonyme Nutzer
- **Authentication:** Docker Hub Token

### 4. Kubernetes Cluster (Deployment-Ziel)
- **Zweck:** Container Orchestrierung
- **Typ:** Lokales k3s/k8s Cluster
- **Zugriff:** kubectl via kubeconfig
- **Netzwerk:** 192.168.2.x (generisch konfigurierbar)

### 5. Secrets Management (GitHub Secrets)
- **Zweck:** Sichere Speicherung von Credentials
- **Verschlüsselung:** AES-256
- **Zugriff:** Nur in GitHub Actions verfügbar
- **Beispiele:** DOCKER_USERNAME, DOCKER_PASSWORD, KUBE_CONFIG

---

## 🔄 Implementierungs-Phasen

### Phase 1: Vorbereitung (15 Minuten)
**Ziel:** Accounts und Zugriffe einrichten

**Schritte:**
1. GitHub Repository erstellen/konfigurieren
2. Docker Hub Account erstellen
3. Docker Hub Repositories anlegen
4. GitHub Secrets konfigurieren
5. kubeconfig Base64-encodieren

**Benötigte Secrets:**
- `DOCKER_USERNAME` - Docker Hub Username
- `DOCKER_PASSWORD` - Docker Hub Token
- `KUBE_CONFIG` - Base64-encoded kubeconfig

### Phase 2: Dockerfile-Struktur (30 Minuten)
**Ziel:** Multi-Stage Builds für optimale Images

**Konzepte:**
- **Build Stage:** Dependencies installieren, Code kompilieren
- **Production Stage:** Nur Runtime + kompilierter Code
- **Ergebnis:** 90% kleinere Images (z.B. 1GB → 100MB)

**Framework-agnostische Struktur:**
- Build-Tools in separater Stage
- Alpine Linux für kleinere Base Images
- .dockerignore für bessere Performance
- Health Check Endpoints

### Phase 3: GitHub Actions Workflows (45 Minuten)
**Ziel:** Automatisierte Build- und Deploy-Pipeline

**Workflow-Komponenten:**
1. **Trigger:** Push auf main branch, Pull Requests
2. **Environment:** Ubuntu Latest Runner
3. **Steps:** Checkout → Setup → Build → Test → Docker → Push → Deploy
4. **Matrix Builds:** Für mehrere Services/Sprachen
5. **Conditional Deploy:** Nur bei erfolgreichen Tests

**Workflow-Typen:**
- **CI Workflow:** Build + Test (bei jedem Push/PR)
- **CD Workflow:** Deploy (nur bei Push auf main)
- **Manual Workflow:** Manueller Deploy für spezifische Tags

### Phase 4: Kubernetes Integration (30 Minuten)
**Ziel:** Sichere Verbindung zwischen GitHub Actions und Kubernetes

**Komponenten:**
1. **kubeconfig Setup:** Base64-decodieren und konfigurieren
2. **kubectl Installation:** In GitHub Actions Runner
3. **Namespace Management:** Automatische Erstellung
4. **Secrets Injection:** Aus GitHub Secrets in Kubernetes
5. **Image Updates:** Rolling Updates mit neuen Image Tags

**Sicherheitsaspekte:**
- kubeconfig mit minimalen Permissions
- Service Account für GitHub Actions
- RBAC-Regeln für Deployment-spezifische Rechte

### Phase 5: Deployment-Automatisierung (20 Minuten)
**Ziel:** Zero-Downtime Deployments

**Strategien:**
- **Rolling Updates:** Neue Pods starten, bevor alte gestoppt werden
- **Health Checks:** Liveness und Readiness Probes
- **Rollback:** Automatisch bei fehlgeschlagenen Deployments
- **Blue-Green:** Optional für kritische Services

**Deployment-Patterns:**
- **Image Tagging:** latest + git-sha für Rollback-Fähigkeit
- **Environment Variables:** Via ConfigMaps und Secrets
- **Resource Limits:** CPU/Memory Limits für Stabilität

### Phase 6: Monitoring & Rollback (15 Minuten)
**Ziel:** Überwachung und Fehlerbehandlung

**Monitoring:**
- **Deployment Status:** kubectl rollout status
- **Pod Health:** kubectl get pods
- **Logs:** kubectl logs für Debugging
- **Events:** kubectl get events für Fehleranalyse

**Rollback-Strategien:**
- **Automatisch:** Bei fehlgeschlagenen Health Checks
- **Manuell:** kubectl rollout undo
- **Versioniert:** Zu spezifischen Image Tags zurück

---

## 🔐 Sicherheitskonzepte

### GitHub Secrets Management
**Zweck:** Sichere Speicherung von Credentials

**Best Practices:**
- Niemals Secrets in Code committen
- Regelmäßige Rotation von Tokens
- Minimale Permissions für Secrets
- Audit Logs für Secret-Zugriffe

**Secret-Typen:**
- **Docker Hub Token:** Für Image Push/Pull
- **kubeconfig:** Für Kubernetes-Zugriff
- **Application Secrets:** Datenbank-Passwörter, API Keys

### Docker Hub Token-basierte Auth
**Zweck:** Sichere Authentifizierung ohne Passwort

**Vorteile:**
- Keine Passwort-Exposition
- Token-spezifische Permissions
- Einfache Revocation
- Audit Trail

**Setup:**
1. Docker Hub → Account Settings → Security
2. New Access Token erstellen
3. Token als DOCKER_PASSWORD Secret speichern

### Kubernetes RBAC
**Zweck:** Minimale Berechtigungen für GitHub Actions

**Konfiguration:**
- Service Account für GitHub Actions
- ClusterRole mit minimalen Permissions
- ClusterRoleBinding für Namespace-spezifische Rechte

**Permissions:**
- Deployments erstellen/aktualisieren
- Services verwalten
- ConfigMaps/Secrets lesen
- Pods/Events lesen für Monitoring

### Image Scanning (Optional)
**Zweck:** Sicherheitslücken in Container Images finden

**Tools:**
- **Trivy:** Open Source Vulnerability Scanner
- **Snyk:** Commercial mit GitHub Integration
- **GitHub Security Advisories:** Automatische Scans

**Integration:**
- In GitHub Actions Workflow einbauen
- Build stoppen bei kritischen Vulnerabilities
- Reports in GitHub Security Tab

---

## 📁 Dateien-Struktur

### Repository-Struktur
```
project-root/
├── .github/
│   └── workflows/
│       ├── ci.yml              # Build & Test
│       ├── cd.yml              # Deploy
│       └── manual-deploy.yml   # Manual Deploy
├── apps/
│   ├── backend/
│   │   ├── Dockerfile
│   │   ├── .dockerignore
│   │   └── src/
│   └── frontend/
│       ├── Dockerfile
│       ├── .dockerignore
│       └── src/
├── k8s/
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── secrets.yaml
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   └── services.yaml
├── .env.example
└── README.md
```

### Workflow-Dateien
**`.github/workflows/ci.yml`**
- Trigger: Push, Pull Request
- Steps: Checkout → Setup → Build → Test
- Kein Deployment

**`.github/workflows/cd.yml`**
- Trigger: Push auf main branch
- Steps: Build → Test → Docker → Push → Deploy
- Vollständige Deployment-Pipeline

**`.github/workflows/manual-deploy.yml`**
- Trigger: workflow_dispatch
- Steps: Deploy spezifische Version
- Für Rollbacks und Hotfixes

### Kubernetes Manifests
**`k8s/namespace.yaml`**
- Namespace-Definition
- Labels und Annotations

**`k8s/configmap.yaml`**
- Environment-spezifische Konfiguration
- Nicht-sensitive Daten

**`k8s/secrets.yaml`**
- Template für Secrets
- Werte werden via GitHub Actions injected

**`k8s/*-deployment.yaml`**
- Deployment-Definitionen
- Image Tags als Variablen
- Resource Limits und Health Checks

**`k8s/services.yaml`**
- Service-Definitionen
- Load Balancing und Networking

---

## 🏆 Best Practices

### Image Tagging-Strategien
**Zweck:** Versionierung und Rollback-Fähigkeit

**Tags:**
- `latest` - Immer neueste Version (für schnelle Tests)
- `{git-sha}` - Exakte Version (für präzise Rollbacks)
- `{version}` - Semantische Versionierung (für Releases)

**Warum beide?**
- Rollback zu jeder Version in Sekunden möglich
- Traceability von Deployments zu Code-Commits
- Staging/Production mit verschiedenen Tags

### Branch-Strategien
**Zweck:** Kontrollierte Deployments

**Branches:**
- `main` - Production-ready Code
- `develop` - Integration Branch
- `feature/*` - Feature Development
- `hotfix/*` - Kritische Fixes

**Deployment-Regeln:**
- `main` → Production Deployment
- `develop` → Staging Deployment
- `feature/*` → Kein Deployment (nur Tests)

### Environment-Management
**Zweck:** Verschiedene Umgebungen verwalten

**Environments:**
- **Development:** Lokale Entwicklung
- **Staging:** Test-Umgebung
- **Production:** Live-System

**Konfiguration:**
- Environment-spezifische ConfigMaps
- Separate Kubernetes Namespaces
- Environment-spezifische Secrets
- Feature Flags für Gradual Rollouts

### Health Checks & Probes
**Zweck:** Automatische Fehlererkennung und Recovery

**Liveness Probe:**
- "Lebt der Container noch?"
- Neustart bei Fehler
- Beispiel: HTTP GET /health

**Readiness Probe:**
- "Ist der Container bereit für Traffic?"
- Kein Traffic wenn nicht ready
- Beispiel: HTTP GET /ready

**Startup Probe:**
- "Ist der Container gestartet?"
- Verhindert Liveness-Probe während Startup
- Beispiel: HTTP GET /startup

---

## 🔧 Troubleshooting

### Häufige Fehler

**"Failed to push to Docker Hub"**
- **Ursache:** Falsches Docker Hub Token oder Rate Limit
- **Lösung:** Token neu erstellen, Rate Limits prüfen
- **Prävention:** Docker Hub Pro für höhere Limits

**"kubectl: connection refused"**
- **Ursache:** kubeconfig falsch encodiert oder veraltet
- **Lösung:** kubeconfig neu encodieren mit `cat ~/.kube/config | base64 -w 0`
- **Prävention:** Regelmäßige kubeconfig Updates

**"ImagePullBackOff" in Kubernetes**
- **Ursache:** Image existiert nicht auf Docker Hub oder Auth-Fehler
- **Lösung:** Docker Hub Repository prüfen, Image manuell pushen
- **Prävention:** Image-Tags in Workflow-Logs prüfen

**"CrashLoopBackOff" nach Deployment**
- **Ursache:** Container startet nicht (z.B. DB-Verbindung fehlt)
- **Lösung:** Logs prüfen mit `kubectl logs <pod-name>`
- **Prävention:** Health Checks und Startup Probes

**Workflow schlägt fehl bei "npm ci"**
- **Ursache:** `package-lock.json` fehlt oder veraltet
- **Lösung:** Lokal `npm install` → `package-lock.json` committen
- **Prävention:** package-lock.json in Git tracken

### Debug-Strategien

**GitHub Actions Logs:**
1. Repository → Actions Tab
2. Workflow Run auswählen
3. Job → Step auswählen
4. Logs analysieren

**Kubernetes Debugging:**
```bash
# Pod Status prüfen
kubectl get pods -n <namespace>

# Pod Details
kubectl describe pod <pod-name> -n <namespace>

# Logs anschauen
kubectl logs <pod-name> -n <namespace> -f

# Events prüfen
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

**Docker Hub Debugging:**
1. Docker Hub → Repository
2. Tags Tab → Image Tags prüfen
3. Build History → Build Logs
4. Rate Limits → Usage prüfen

### Rollback-Prozesse

**Zu vorheriger Version:**
```bash
kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

**Zu spezifischer Version:**
```bash
kubectl set image deployment/<deployment-name> \
  <container-name>=<image>:<tag> \
  -n <namespace>
```

**Rollout-Status prüfen:**
```bash
kubectl rollout status deployment/<deployment-name> -n <namespace>
kubectl rollout history deployment/<deployment-name> -n <namespace>
```

**Automatischer Rollback:**
- In Kubernetes Deployment konfigurieren
- `progressDeadlineSeconds` für Timeout
- `maxUnavailable` und `maxSurge` für Rolling Updates

---

## 📊 Monitoring & Observability

### Deployment-Überwachung
**Zweck:** Status von Deployments verfolgen

**Metriken:**
- Deployment Status (Progressing, Available, ReplicaFailure)
- Pod Status (Running, Pending, Failed)
- Replica Count (Desired vs Available)
- Rollout Duration

**Tools:**
- `kubectl rollout status` - Deployment Progress
- `kubectl get pods` - Pod Status
- `kubectl top pods` - Resource Usage

### Log-Aggregation
**Zweck:** Zentrale Log-Verwaltung

**Strategien:**
- **kubectl logs** - Einzelne Pod Logs
- **Fluentd/Fluent Bit** - Log Collection
- **ELK Stack** - Elasticsearch, Logstash, Kibana
- **Loki** - Lightweight Alternative zu ELK

### Health Monitoring
**Zweck:** Proaktive Fehlererkennung

**Health Checks:**
- **Application Health:** /health Endpoint
- **Database Connectivity:** DB Connection Tests
- **External Dependencies:** API Availability
- **Resource Usage:** CPU, Memory, Disk

**Alerting:**
- **GitHub Actions:** Workflow Failure Notifications
- **Kubernetes:** Event-based Alerts
- **Application:** Custom Health Check Alerts

---

## 🚀 Nächste Schritte

### Phase 1: Grundlagen (Woche 1)
- [ ] GitHub Repository Setup
- [ ] Docker Hub Account und Repositories
- [ ] GitHub Secrets konfigurieren
- [ ] Erste Dockerfile erstellen

### Phase 2: CI/CD Pipeline (Woche 2)
- [ ] GitHub Actions Workflows erstellen
- [ ] Kubernetes Manifests entwickeln
- [ ] Erste automatische Deployments testen
- [ ] Health Checks implementieren

### Phase 3: Optimierung (Woche 3)
- [ ] Multi-Stage Docker Builds
- [ ] Image Tagging-Strategie
- [ ] Rollback-Prozesse
- [ ] Monitoring und Logging

### Phase 4: Production-Ready (Woche 4)
- [ ] Security Hardening
- [ ] Performance Optimierung
- [ ] Documentation
- [ ] Team Training

### Erweiterte Features (Optional)
- [ ] **Blue-Green Deployments** - Zero-downtime Updates
- [ ] **Canary Releases** - Gradual Rollouts
- [ ] **Feature Flags** - Runtime Feature Control
- [ ] **Multi-Environment** - Dev/Staging/Prod
- [ ] **Cost Optimization** - Resource Right-sizing

---

## 💰 Kosten-Übersicht

### Development (Aktuell)
- **GitHub Actions:** 2000 Min/Monat kostenlos
- **Docker Hub:** Unbegrenzt public repos kostenlos
- **Kubernetes:** Eure Hardware ($0)
- **Total: $0/Monat**

### Skalierung (Bei Wachstum)
- **GitHub Actions:** $0.008/Min (sehr günstig)
- **Docker Hub Pro:** $5/Monat (höhere Rate Limits)
- **Kubernetes:** Eure Hardware ($0)
- **Total: $5-50/Monat** (je nach Build-Volumen)

### Vergleich zu Alternativen
- **Azure DevOps:** $6/User/Monat + $40/1000 Min
- **GitLab CI:** $4/User/Monat + 400 Min/Monat
- **Jenkins:** $0 (Self-hosted) + Server-Kosten

---

## 📚 Weitere Ressourcen

### Dokumentation
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Hub Documentation](https://docs.docker.com/docker-hub/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [k3s Documentation](https://k3s.io/)

### Community
- [GitHub Actions Community](https://github.com/actions)
- [Docker Community](https://forums.docker.com/)
- [Kubernetes Community](https://kubernetes.io/community/)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/github-actions)

### Tools & Extensions
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)
- [Docker Hub Official Images](https://hub.docker.com/search?q=&type=image&image_filter=official)
- [Kubernetes Tools](https://kubernetes.io/docs/tasks/tools/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

*Dieses Dokument dient als umfassender Leitfaden für die Implementierung einer CI/CD-Pipeline mit GitHub Actions und Docker Hub für lokale Kubernetes Cluster. Alle Platzhalter (IPs, Namen, Credentials) sollten durch eure spezifischen Werte ersetzt werden.*
