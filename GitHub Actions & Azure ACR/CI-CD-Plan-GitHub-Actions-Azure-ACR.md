# CI/CD Pipeline Plan: GitHub Actions + Azure Container Registry

## 🎯 Übersicht & Zielsetzung

### Was wird erreicht?
**Automatisierung von:** Code-Änderung → Build → Test → Deploy auf lokales Kubernetes Cluster

**Best of both worlds:**
- GitHub Actions als CI/CD Engine (kostenlos, einfach, große Community)
- Azure Container Registry als Image Registry (privat, sicher, günstig)
- Lokales Kubernetes Cluster als Deployment-Ziel
- Azure Key Vault für Secrets Management (optional)

### Wann ist diese Lösung sinnvoll?
- ✅ **Kostenoptimiert** - GitHub Actions kostenlos + ACR nur $5/Monat
- ✅ **Sicherheit** - Private Registry mit Vulnerability Scanning
- ✅ **Einfachheit** - GitHub Actions ist einfacher als Azure Pipelines
- ✅ **Flexibilität** - Keine Vendor Lock-in bei GitHub Actions
- ✅ **Skalierbarkeit** - ACR wächst mit euren Anforderungen mit
- ✅ **Zukunftssicher** - Einfacher Wechsel zu Azure DevOps später möglich

### Vor- und Nachteile

**Vorteile:**
- **Kosten:** $0 (GitHub Actions) + $5/Monat (ACR) = sehr günstig
- **Einfachheit:** GitHub Actions ist einfacher zu lernen als Azure Pipelines
- **Sicherheit:** Private Registry mit Vulnerability Scanning
- **Community:** Große GitHub Actions Community und Marketplace
- **Flexibilität:** Keine Vendor Lock-in bei CI/CD Engine
- **Azure-Integration:** Bereits im Azure-Ökosystem für spätere Migration

**Nachteile:**
- **Hybrid-Setup:** Zwei verschiedene Provider (GitHub + Azure)
- **Secrets Management:** Komplexer als reine Azure-Lösung
- **Support:** Verschiedene Support-Kanäle für GitHub und Azure
- **Billing:** Separate Rechnungen für GitHub und Azure

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
Azure Container Registry (ACR)
    ↓
[6] Service Principal Auth
[7] Kubernetes Deployment aktualisieren
[8] Rollout überwachen
    ↓
Kubernetes Cluster (lokal)
    ↓
[9] Pods starten mit neuem Image
[10] Health Checks durchführen
```

### Hybrid-Integration
- **GitHub Actions:** CI/CD Engine mit großer Community
- **Azure ACR:** Private Container Registry mit Enterprise-Features
- **Service Principal:** Sichere Authentifizierung zwischen GitHub und Azure
- **Azure Key Vault:** Optional für erweiterte Secrets-Verwaltung

### Netzwerk-Kommunikation
- **GitHub Actions → ACR:** HTTPS (Image Push via Service Principal)
- **GitHub Actions → Kubernetes:** HTTPS (kubectl API)
- **Kubernetes → ACR:** HTTPS (Image Pull via Service Principal)
- **Lokaler Cluster:** 192.168.2.x Netzwerk

---

## 📦 Komponenten-Übersicht

### 1. GitHub Repository (Code-Speicher)
- **Zweck:** Zentrale Code-Verwaltung
- **Features:** Version Control, Branching, Pull Requests
- **Trigger:** Push, Pull Request, Tag, Schedule
- **Kosten:** Kostenlos

### 2. GitHub Actions (CI/CD Engine)
- **Zweck:** Automatisierungs-Engine
- **Laufzeit:** GitHub-gehostete Runner (Ubuntu, Windows, macOS)
- **Free Tier:** 2000 Minuten/Monat
- **Workflows:** YAML-basierte Konfiguration
- **Kosten:** Kostenlos (innerhalb Free Tier)

### 3. Azure Container Registry (Image Registry)
- **Zweck:** Private Container Image Speicherung
- **Features:** Vulnerability Scanning, Azure AD Integration, Content Trust
- **Security:** Private Registry, Service Principal Auth
- **Kosten:** $5/Monat (Basic Tier)

### 4. Service Principal (Auth zwischen GitHub und Azure)
- **Zweck:** Sichere Authentifizierung zwischen GitHub Actions und ACR
- **Features:** Certificate-based Auth, Role-based Permissions
- **Security:** Keine Passwörter, automatische Rotation möglich
- **Integration:** Azure AD und Kubernetes RBAC

### 5. Azure Key Vault (Optional - Secrets Management)
- **Zweck:** Zentrale, sichere Secrets-Verwaltung
- **Features:** Automatic Rotation, Audit Logs, Access Policies
- **Integration:** Nahtlose Verbindung zu GitHub Actions
- **Kosten:** $0-5/Monat (je nach Nutzung)

### 6. Kubernetes Cluster (Deployment-Ziel)
- **Zweck:** Container Orchestrierung
- **Typ:** Lokales k3s/k8s Cluster
- **Zugriff:** kubectl via kubeconfig
- **Netzwerk:** 192.168.2.x (generisch konfigurierbar)

---

## 🔄 Implementierungs-Phasen

### Phase 1: Azure-Setup (20 Minuten)
**Ziel:** Azure Container Registry und Service Principal einrichten

**Schritte:**
1. **Azure Subscription** aktivieren/konfigurieren
2. **Resource Group** erstellen für ACR
3. **Azure Container Registry** erstellen (Basic Tier - $5/Monat)
4. **Service Principal** für ACR-Zugriff erstellen
5. **ACR Admin User** aktivieren (für lokale Entwicklung)

**Benötigte Azure Services:**
- Azure Container Registry (ACR)
- Azure Active Directory (für Service Principal)
- Azure Key Vault (optional)

### Phase 2: Service Principal erstellen (15 Minuten)
**Zweck:** Sichere Authentifizierung zwischen GitHub Actions und ACR

**Service Principal für ACR:**
```bash
# ACR Push/Pull Permissions
az ad sp create-for-rbac --name "github-actions-acr-sp" \
  --role acrpush \
  --scopes /subscriptions/{subscription-id}/resourceGroups/{rg}/providers/Microsoft.ContainerRegistry/registries/{acr-name}
```

**Outputs notieren:**
- Application (client) ID
- Client Secret Value
- Tenant ID

**Alternative: ACR Admin User (einfacher)**
- ACR Admin User aktivieren
- Username/Password als GitHub Secrets speichern
- Weniger sicher, aber einfacher zu setup

### Phase 3: GitHub Secrets konfigurieren (10 Minuten)
**Zweck:** Sichere Speicherung von Azure-Credentials

**GitHub Repository → Settings → Secrets and variables → Actions → New repository secret:**

**Service Principal Secrets:**
- `AZURE_CLIENT_ID` - Application (client) ID
- `AZURE_CLIENT_SECRET` - Client Secret Value
- `AZURE_TENANT_ID` - Directory (tenant) ID
- `ACR_NAME` - Azure Container Registry Name
- `ACR_LOGIN_SERVER` - ACR Login Server (z.B. myregistry.azurecr.io)

**Oder ACR Admin User Secrets:**
- `ACR_USERNAME` - ACR Admin Username
- `ACR_PASSWORD` - ACR Admin Password
- `ACR_LOGIN_SERVER` - ACR Login Server

**Kubernetes Secrets:**
- `KUBE_CONFIG` - Base64-encoded kubeconfig

### Phase 4: GitHub Actions Workflows (45 Minuten)
**Zweck:** CI/CD Pipeline mit ACR-Integration

**Workflow-Komponenten:**
1. **Trigger:** Push auf main branch, Pull Requests
2. **Environment:** Ubuntu Latest Runner
3. **Steps:** Checkout → Setup → Build → Test → Docker → Push to ACR → Deploy
4. **ACR Integration:** Login → Build → Push → Deploy
5. **Kubernetes Integration:** kubectl → Deploy → Monitor

**Workflow-Features:**
- **Multi-Stage Docker Builds:** Optimierte Images
- **ACR Authentication:** Service Principal oder Admin User
- **Image Tagging:** latest + git-sha für Rollback-Fähigkeit
- **Rolling Updates:** Zero-downtime Deployments
- **Health Checks:** Liveness und Readiness Probes

### Phase 5: Kubernetes Integration (25 Minuten)
**Zweck:** Sichere Verbindung zwischen GitHub Actions und Kubernetes

**Komponenten:**
1. **kubeconfig Setup:** Base64-decodieren und konfigurieren
2. **kubectl Installation:** In GitHub Actions Runner
3. **ACR ImagePullSecrets:** Service Principal für Image Pull
4. **Namespace Management:** Automatische Erstellung
5. **Deployment Updates:** Rolling Updates mit neuen Image Tags

**Kubernetes RBAC für ACR:**
```yaml
# Service Account für ACR-Zugriff
apiVersion: v1
kind: ServiceAccount
metadata:
  name: acr-pull-secret
  namespace: default
---
# Secret für ACR-Authentifizierung
apiVersion: v1
kind: Secret
metadata:
  name: acr-secret
  namespace: default
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-acr-credentials>
```

### Phase 6: Deployment-Automatisierung (20 Minuten)
**Zweck:** Zero-Downtime Deployments mit ACR Images

**Strategien:**
- **Rolling Updates:** Neue Pods starten, bevor alte gestoppt werden
- **Health Checks:** Liveness und Readiness Probes
- **Rollback:** Automatisch bei fehlgeschlagenen Deployments
- **Image Pull:** Service Principal-basierte ACR-Authentifizierung

**Deployment-Patterns:**
- **Image Tagging:** latest + git-sha für Rollback-Fähigkeit
- **Environment Variables:** Via ConfigMaps und Secrets
- **Resource Limits:** CPU/Memory Limits für Stabilität
- **ACR Integration:** Private Images mit Vulnerability Scanning

### Phase 7: Monitoring & Rollback (15 Minuten)
**Zweck:** Überwachung und Fehlerbehandlung

**Monitoring:**
- **Deployment Status:** kubectl rollout status
- **Pod Health:** kubectl get pods
- **ACR Metrics:** Image Push/Pull Rates, Storage Usage
- **Logs:** kubectl logs für Debugging

**Rollback-Strategien:**
- **Automatisch:** Bei fehlgeschlagenen Health Checks
- **Manuell:** kubectl rollout undo
- **Versioniert:** Zu spezifischen ACR Image Tags zurück

---

## 🔐 Sicherheitskonzepte

### Service Principal Authentication
**Zweck:** Sichere Authentifizierung zwischen GitHub Actions und ACR

**Best Practices:**
- **Minimale Permissions:** Nur ACR Push/Pull Rechte
- **Automatische Rotation:** Service Principal Secrets regelmäßig erneuern
- **Audit Logs:** Alle ACR-Zugriffe protokollieren
- **Network Security:** ACR Private Endpoints (optional)

**Service Principal Setup:**
1. **Azure AD App Registration** erstellen
2. **ACR Permissions** zuweisen (acrpush, acrpull)
3. **Client Secret** generieren
4. **GitHub Secrets** konfigurieren

### ACR Private Registry
**Zweck:** Sichere, private Container Images

**Security Features:**
- **Private Endpoints:** Kein öffentlicher Zugriff
- **Azure AD Integration:** Benutzer-basierte Authentifizierung
- **Vulnerability Scanning:** Automatische Sicherheitsprüfungen
- **Content Trust:** Digitale Signierung von Images (optional)

**Network Security:**
- **Virtual Network Integration:** ACR in VNet (optional)
- **Private Endpoints:** Kein Internet-Zugriff erforderlich
- **Firewall Rules:** IP-basierte Zugriffskontrolle

### GitHub Secrets Management
**Zweck:** Sichere Speicherung von Azure-Credentials

**Best Practices:**
- **Niemals Secrets in Code committen**
- **Regelmäßige Rotation** von Service Principal Secrets
- **Minimale Permissions** für Secrets
- **Audit Logs** für Secret-Zugriffe

**Secret-Typen:**
- **Azure Credentials:** Service Principal oder ACR Admin User
- **ACR Configuration:** Registry Name, Login Server
- **Kubernetes Access:** kubeconfig für Cluster-Zugriff

### Kubernetes RBAC
**Zweck:** Minimale Berechtigungen für GitHub Actions

**Konfiguration:**
- **Service Account** für GitHub Actions
- **ClusterRole** mit minimalen Permissions
- **ClusterRoleBinding** für Namespace-spezifische Rechte
- **ACR ImagePullSecrets** für private Image-Zugriffe

**Permissions:**
- **Deployments:** Erstellen, aktualisieren, löschen
- **Services:** Verwalten von Services
- **ConfigMaps/Secrets:** Lesen und erstellen
- **Pods/Events:** Lesen für Monitoring

---

## 📁 Dateien-Struktur

### Repository-Struktur
```
project-root/
├── .github/
│   └── workflows/
│       ├── ci.yml              # Build & Test
│       ├── cd.yml              # Deploy to ACR
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
│   ├── acr-secret.yaml         # ACR ImagePullSecrets
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   └── services.yaml
├── .env.example
└── README.md
```

### Workflow-Dateien
**`.github/workflows/ci.yml`**
- **Trigger:** Push, Pull Request
- **Steps:** Checkout → Setup → Build → Test
- **ACR:** Kein Push, nur Tests

**`.github/workflows/cd.yml`**
- **Trigger:** Push auf main branch
- **Steps:** Build → Test → Docker → Push to ACR → Deploy
- **ACR Integration:** Login → Build → Push → Deploy

**`.github/workflows/manual-deploy.yml`**
- **Trigger:** workflow_dispatch
- **Steps:** Deploy spezifische ACR Image Version
- **Features:** Rollbacks und Hotfixes

### Kubernetes Manifests mit ACR
**`k8s/namespace.yaml`**
- Namespace-Definition
- Labels und Annotations

**`k8s/acr-secret.yaml`**
- ACR ImagePullSecrets
- Service Principal Credentials
- Für private Image-Zugriffe

**`k8s/configmap.yaml`**
- Environment-spezifische Konfiguration
- Nicht-sensitive Daten

**`k8s/secrets.yaml`**
- Template für Application Secrets
- Werte werden via GitHub Actions injected

**`k8s/*-deployment.yaml`**
- Deployment-Definitionen mit ACR Images
- Image Tags als Variablen
- Resource Limits und Health Checks

**`k8s/services.yaml`**
- Service-Definitionen
- Load Balancing und Networking

---

## 🏆 Best Practices

### Image Tagging-Strategien
**Zweck:** Versionierung und Rollback-Fähigkeit mit ACR

**Tags:**
- `latest` - Immer neueste Version (für schnelle Tests)
- `{git-sha}` - Exakte Version (für präzise Rollbacks)
- `{version}` - Semantische Versionierung (für Releases)

**ACR-spezifische Features:**
- **Retention Policies:** Automatische Cleanup alter Images
- **Vulnerability Scanning:** Sicherheitsprüfungen bei jedem Push
- **Content Trust:** Digitale Signierung (optional)

### Branch-Strategien
**Zweck:** Kontrollierte Deployments mit ACR

**Branches:**
- `main` - Production-ready Code → ACR latest
- `develop` - Integration Branch → ACR develop
- `feature/*` - Feature Development → Kein ACR Push
- `hotfix/*` - Kritische Fixes → ACR hotfix

**Deployment-Regeln:**
- `main` → ACR latest → Production Deployment
- `develop` → ACR develop → Staging Deployment
- `feature/*` → Kein ACR Push (nur Tests)

### Environment-Management
**Zweck:** Verschiedene Umgebungen mit ACR verwalten

**Environments:**
- **Development:** Lokale Entwicklung mit ACR latest
- **Staging:** Test-Umgebung mit ACR develop
- **Production:** Live-System mit ACR latest

**ACR-Integration:**
- **Environment-spezifische Tags:** dev, staging, prod
- **Separate ACR Repositories:** Optional für große Teams
- **Retention Policies:** Verschiedene Policies pro Environment

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

**ACR-Integration:**
- **Image Health:** Vulnerability Scanning in ACR
- **Deployment Health:** Kubernetes Health Checks
- **Rollback:** Automatisch bei fehlgeschlagenen Health Checks

---

## 🔧 Troubleshooting

### ACR Authentication Probleme
**"Failed to push to ACR"**
- **Ursache:** Service Principal Credentials falsch oder ACR Admin User nicht aktiviert
- **Lösung:** Service Principal neu erstellen oder ACR Admin User aktivieren
- **Prävention:** Credentials in GitHub Secrets testen

**"Image pull failed from ACR"**
- **Ursache:** Kubernetes kann nicht auf ACR zugreifen
- **Lösung:** ImagePullSecrets in Kubernetes konfigurieren
- **Prävention:** ACR Service Principal für Kubernetes einrichten

### GitHub Actions Probleme
**"Azure login failed"**
- **Ursache:** Service Principal Credentials falsch in GitHub Secrets
- **Lösung:** GitHub Secrets überprüfen, Service Principal neu erstellen
- **Prävention:** Credentials lokal testen vor GitHub Secrets Setup

**"ACR login failed"**
- **Ursache:** ACR Name oder Login Server falsch
- **Lösung:** ACR Name und Login Server in GitHub Secrets überprüfen
- **Prävention:** ACR-URLs dokumentieren

### Kubernetes Integration Probleme
**"kubectl connection refused"**
- **Ursache:** kubeconfig falsch encodiert oder Kubernetes nicht erreichbar
- **Lösung:** kubeconfig neu encodieren, Netzwerk-Konnektivität prüfen
- **Prävention:** kubeconfig lokal testen

**"ImagePullBackOff" mit ACR Images**
- **Ursache:** ACR ImagePullSecrets nicht konfiguriert
- **Lösung:** ACR Secret in Kubernetes erstellen
- **Prävention:** ImagePullSecrets in Deployment-Manifest

### ACR-spezifische Probleme
**"ACR rate limit exceeded"**
- **Ursache:** Zu viele ACR-Requests (sehr selten bei Basic Tier)
- **Lösung:** ACR Standard Tier upgraden oder Requests reduzieren
- **Prävention:** ACR Usage überwachen

**"Vulnerability scan failed"**
- **Ursache:** ACR Vulnerability Scanning temporär nicht verfügbar
- **Lösung:** Scan manuell in ACR Portal starten
- **Prävention:** Vulnerability Scanning als Warning, nicht als Error

---

## 📊 Monitoring & Observability

### ACR Monitoring
**Zweck:** Überwachung der Azure Container Registry

**ACR-Metriken:**
- **Image Push/Pull Rates:** Registry-Nutzung
- **Storage Usage:** Verbrauchter Speicherplatz
- **Vulnerability Scans:** Sicherheitsstatus der Images
- **Retention Policies:** Cleanup-Aktivitäten

**Monitoring-Tools:**
- **Azure Monitor:** Native ACR-Metriken
- **Azure Security Center:** Vulnerability Management
- **Custom Dashboards:** Grafana mit Azure Data Source

### GitHub Actions Monitoring
**Zweck:** Überwachung der CI/CD-Pipeline

**Pipeline-Metriken:**
- **Build Success Rate:** Erfolgsrate der Builds
- **Deployment Duration:** Zeit für Deployments
- **ACR Push Success:** Erfolgsrate der ACR Pushes
- **Test Coverage:** Code-Coverage-Metriken

**Monitoring-Strategien:**
- **GitHub Actions Analytics:** Native GitHub Analytics
- **Custom Dashboards:** GitHub API + Grafana
- **Alerts:** Fehlgeschlagene Builds und ACR Pushes
- **Reports:** Wöchentliche/Monatliche Pipeline-Reports

### Kubernetes Monitoring
**Zweck:** Überwachung des Kubernetes Clusters

**Kubernetes-Metriken:**
- **Pod Status:** Running, Pending, Failed Pods
- **Resource Usage:** CPU, Memory, Storage
- **Deployment Status:** Rollout-Status und Health
- **ACR Image Pulls:** Erfolgsrate der Image Pulls

**Monitoring-Integration:**
- **kubectl:** Native Kubernetes-Monitoring
- **Prometheus + Grafana:** Custom Monitoring-Stack
- **Azure Monitor for Containers:** Optional für erweiterte Metriken

---

## 💰 Kosten-Übersicht

### Development (Aktuell)
- **GitHub Actions:** 2000 Min/Monat kostenlos
- **Azure Container Registry:** $5/Monat (Basic)
- **Azure Key Vault:** $0-5/Monat (je nach Nutzung)
- **Kubernetes:** Eure Hardware ($0)
- **Total: $5-10/Monat**

### Skalierung (Bei Wachstum)
- **GitHub Actions:** $0.008/Min (sehr günstig)
- **ACR Standard:** $20/Monat (100GB Storage)
- **Azure Key Vault:** $5-20/Monat
- **Kubernetes:** Eure Hardware ($0)
- **Total: $25-50/Monat**

### Vergleich zu anderen Ansätzen
| Ansatz | CI/CD | Registry | Total/Monat |
|--------|-------|----------|-------------|
| **GitHub + Docker Hub** | $0 | $0-5 | $0-5 |
| **GitHub + ACR** | $0 | $5 | $5 |
| **Azure DevOps + ACR** | $6/User | $5 | $11+ |
| **GitHub + ACR (unser Ansatz)** | $0 | $5 | $5 |

### ROI-Analyse
**Kosten vs Nutzen:**
- **Setup-Zeit:** 2-3 Stunden
- **Monatliche Kosten:** $5
- **Zeitersparnis:** 5-10 Stunden/Monat (manuelle Deployments)
- **ROI:** Nach 1-2 Monaten positiv

---

## 🚀 Nächste Schritte

### Phase 1: Azure-Setup (Woche 1)
- [ ] Azure Subscription und Resource Group
- [ ] Azure Container Registry erstellen (Basic)
- [ ] Service Principal für ACR erstellen
- [ ] GitHub Secrets konfigurieren

### Phase 2: GitHub Actions Integration (Woche 2)
- [ ] GitHub Actions Workflows erstellen
- [ ] ACR Authentication in Workflows
- [ ] Docker Build und Push zu ACR
- [ ] Erste ACR Images testen

### Phase 3: Kubernetes Integration (Woche 3)
- [ ] ACR ImagePullSecrets konfigurieren
- [ ] Kubernetes Manifests mit ACR Images
- [ ] Erste automatische Deployments
- [ ] Health Checks implementieren

### Phase 4: Optimierung (Woche 4)
- [ ] Multi-Stage Docker Builds
- [ ] ACR Retention Policies
- [ ] Vulnerability Scanning
- [ ] Monitoring und Alerting

### Phase 5: Production-Ready (Woche 5)
- [ ] Security Hardening
- [ ] Performance Optimierung
- [ ] Documentation
- [ ] Team Training

### Erweiterte Features (Optional)
- [ ] **Azure Key Vault Integration** - Erweiterte Secrets-Verwaltung
- [ ] **ACR Content Trust** - Digitale Signierung von Images
- [ ] **Multi-Region ACR** - Geo-replication für Production
- [ ] **ACR Webhooks** - Automatische Benachrichtigungen
- [ ] **Cost Optimization** - ACR Storage Optimierung

---

## 📚 Weitere Ressourcen

### Dokumentation
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Azure Container Registry Documentation](https://docs.microsoft.com/en-us/azure/container-registry/)
- [Azure Key Vault Documentation](https://docs.microsoft.com/en-us/azure/key-vault/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

### Community
- [GitHub Actions Community](https://github.com/actions)
- [Azure Container Registry Community](https://github.com/Azure/acr)
- [Kubernetes Community](https://kubernetes.io/community/)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/github-actions)

### Tools & Extensions
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/)
- [Azure Container Registry Tools](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-tools-overview)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### Best Practices
- [GitHub Actions Best Practices](https://docs.github.com/en/actions/learn-github-actions/best-practices-for-github-actions)
- [Azure Container Registry Best Practices](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-best-practices)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/)
- [Container Security Best Practices](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-security-best-practices)

---

*Dieses Dokument dient als umfassender Leitfaden für die Implementierung einer hybriden CI/CD-Pipeline mit GitHub Actions und Azure Container Registry für lokale Kubernetes Cluster. Diese Lösung kombiniert die Einfachheit von GitHub Actions mit der Sicherheit und Skalierbarkeit von Azure ACR.*
