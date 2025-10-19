# GitHub Actions + Azure ACR Migration Checklist

## 🎯 Übersicht

Diese Checkliste führt euch durch die Migration eines bestehenden Projekts zu einer CI/CD-Pipeline mit GitHub Actions und Azure Container Registry (ACR). Der Cursor Agent wird euch durch jeden Schritt führen.

## 📋 Vorbereitung (20 Minuten)

### Phase 1: Projekt-Analyse
- [ ] **Projekt-Sprache identifiziert** (Node.js, Python, .NET, etc.)
- [ ] **Framework erkannt** (Express, Flask, ASP.NET, etc.)
- [ ] **Port-Konfiguration** (Standard-Ports: 3000, 5000, 8080)
- [ ] **Dependencies analysiert** (package.json, requirements.txt, etc.)
- [ ] **Build-Scripts identifiziert** (npm run build, dotnet build, etc.)
- [ ] **Test-Scripts gefunden** (npm test, pytest, etc.)

### Phase 2: Azure-Setup
- [ ] **Azure Subscription** aktiv und konfiguriert
- [ ] **Azure CLI** installiert und konfiguriert (`az login`)
- [ ] **Resource Group** erstellt oder vorhanden
- [ ] **Azure Container Registry** erstellt (Basic Tier - $5/Monat)
- [ ] **ACR Admin User** aktiviert
- [ ] **Vulnerability Scanning** aktiviert

### Phase 3: Service Principal Setup
- [ ] **Service Principal** für GitHub Actions erstellt
- [ ] **ACR Permissions** vergeben (acrpush, acrpull)
- [ ] **Service Principal Credentials** notiert:
  - [ ] Application (client) ID
  - [ ] Client Secret Value
  - [ ] Directory (tenant) ID
- [ ] **Service Principal** lokal getestet

## 🔧 Code-Generierung (35 Minuten)

### Phase 4: Dockerfile erstellen
- [ ] **Multi-Stage Dockerfile** generiert
- [ ] **Build Stage** - Dependencies installieren und kompilieren
- [ ] **Production Stage** - Nur Runtime und kompilierter Code
- [ ] **Non-root User** - Security Best Practice
- [ ] **Health Checks** - Liveness und Readiness Probes
- [ ] **.dockerignore** - Optimierte Ignore-Patterns
- [ ] **Dockerfile lokal getestet** - `docker build` erfolgreich

### Phase 5: GitHub Actions Workflows
- [ ] **CI Workflow** (.github/workflows/ci.yml)
  - [ ] Trigger: Push, Pull Request
  - [ ] Steps: Checkout → Setup → Build → Test
  - [ ] Kein Deployment
- [ ] **CD Workflow** (.github/workflows/cd.yml)
  - [ ] Trigger: Push auf main branch
  - [ ] Steps: Build → Test → Docker → Push to ACR → Deploy
  - [ ] Azure ACR Integration
  - [ ] Kubernetes Deployment
- [ ] **Manual Deploy Workflow** (.github/workflows/manual-deploy.yml)
  - [ ] Trigger: workflow_dispatch
  - [ ] Deploy spezifische Version
  - [ ] Rollback-Funktionalität
- [ ] **Security Scan Workflow** (.github/workflows/security-scan.yml)
  - [ ] Trivy Vulnerability Scanning
  - [ ] GitHub Security Tab Integration

### Phase 6: Kubernetes Manifests
- [ ] **namespace.yaml** - Namespace und ConfigMap
- [ ] **secrets.yaml** - Secrets Template
- [ ] **acr-secret.yaml** - ACR ImagePullSecrets
- [ ] **deployment.yaml** - Deployment mit ACR Image
- [ ] **service.yaml** - Service für Netzwerk-Zugriff
- [ ] **ingress.yaml** - Optional für externe Zugriffe
- [ ] **Resource Limits** - CPU und Memory Limits
- [ ] **Health Checks** - Liveness und Readiness Probes

## 🚀 Setup und Konfiguration (25 Minuten)

### Phase 7: Azure ACR Setup
- [ ] **ACR erstellt** (Basic Tier - $5/Monat)
- [ ] **ACR Name** notiert (z.B. myregistry.azurecr.io)
- [ ] **ACR Login Server** notiert
- [ ] **Vulnerability Scanning** aktiviert
- [ ] **Retention Policies** konfiguriert (30 Tage)
- [ ] **ACR lokal getestet** - `az acr login`

### Phase 8: Service Principal Konfiguration
- [ ] **Service Principal** für ACR erstellt
- [ ] **ACR Push/Pull Permissions** vergeben
- [ ] **Service Principal Credentials** gesichert
- [ ] **Service Principal** lokal getestet
- [ ] **ACR Access** mit Service Principal getestet

### Phase 9: GitHub Secrets konfigurieren
- [ ] **AZURE_CLIENT_ID** - Service Principal Client ID
- [ ] **AZURE_CLIENT_SECRET** - Service Principal Client Secret
- [ ] **AZURE_TENANT_ID** - Azure Tenant ID
- [ ] **ACR_NAME** - ACR Registry Name
- [ ] **ACR_LOGIN_SERVER** - ACR Login Server
- [ ] **KUBE_CONFIG** - Base64-encoded kubeconfig
- [ ] **K8S_NAMESPACE** - Kubernetes Namespace Name

### Phase 10: Kubernetes Setup
- [ ] **Namespace** in Kubernetes erstellt
- [ ] **ACR ImagePullSecrets** in Kubernetes erstellt
- [ ] **Secrets** in Kubernetes erstellt (Application Secrets)
- [ ] **ConfigMap** in Kubernetes erstellt
- [ ] **kubeconfig** für GitHub Actions Base64-encodiert
- [ ] **RBAC** für GitHub Actions konfiguriert (optional)

### Phase 11: GitHub Repository Setup
- [ ] **Workflow-Dateien** committet
- [ ] **Kubernetes Manifests** committet
- [ ] **Dockerfile** committet
- [ ] **.dockerignore** committet
- [ ] **Branch Protection** konfiguriert (optional)
- [ ] **Required Status Checks** aktiviert (optional)

## 🧪 Testing und Validierung (20 Minuten)

### Phase 12: Lokales Testing
- [ ] **Dockerfile lokal gebaut** - `docker build -t test-app .`
- [ ] **Docker Image lokal getestet** - `docker run -p 3000:3000 test-app`
- [ ] **ACR Login lokal getestet** - `az acr login`
- [ ] **Image zu ACR gepusht** - `docker push myregistry.azurecr.io/test-app:latest`
- [ ] **Health Endpoints** funktionieren - `/health`, `/ready`
- [ ] **Environment Variables** korrekt gesetzt
- [ ] **Application** läuft ohne Fehler

### Phase 13: CI/CD Testing
- [ ] **Ersten Push** durchgeführt
- [ ] **CI Workflow** erfolgreich gelaufen
- [ ] **CD Workflow** erfolgreich gelaufen
- [ ] **Docker Image** zu ACR gepusht
- [ ] **Vulnerability Scanning** erfolgreich
- [ ] **Kubernetes Deployment** erfolgreich
- [ ] **Application** im Browser/API erreichbar

### Phase 14: End-to-End Testing
- [ ] **Application** funktioniert korrekt
- [ ] **ACR Images** in Azure Portal sichtbar
- [ ] **Vulnerability Scan Results** überprüft
- [ ] **Logs** in Kubernetes überprüft
- [ ] **Pod Status** - alle Pods "Running"
- [ ] **Service** funktioniert
- [ ] **Rollback** getestet (optional)

## 📊 Monitoring und Wartung (15 Minuten)

### Phase 15: Monitoring Setup
- [ ] **Pod Status** überwachen - `kubectl get pods`
- [ ] **Service Status** überwachen - `kubectl get services`
- [ ] **Deployment Status** überwachen - `kubectl rollout status`
- [ ] **Logs** überwachen - `kubectl logs -f deployment/app-name`
- [ ] **Events** überwachen - `kubectl get events`
- [ ] **ACR Usage** überwachen - Azure Portal
- [ ] **Vulnerability Scan Results** überwachen

### Phase 16: Dokumentation
- [ ] **README.md** aktualisiert mit Deployment-Anleitung
- [ ] **ACR Setup** dokumentiert
- [ ] **Service Principal Setup** dokumentiert
- [ ] **Troubleshooting** dokumentiert
- [ ] **Rollback-Prozess** dokumentiert
- [ ] **Team-Training** durchgeführt
- [ ] **Deployment-Prozess** dokumentiert

## 🔍 Qualitätssicherung

### Code-Qualität
- [ ] **Dockerfile** folgt Best Practices
- [ ] **Multi-Stage Build** implementiert
- [ ] **Security** - Non-root User, minimale Base Images
- [ ] **Performance** - Layer Caching optimiert
- [ ] **Health Checks** implementiert

### CI/CD-Qualität
- [ ] **Workflows** syntaktisch korrekt
- [ ] **Azure ACR Integration** korrekt konfiguriert
- [ ] **Service Principal** korrekt konfiguriert
- [ ] **Secrets** korrekt gesetzt
- [ ] **Triggers** korrekt gesetzt
- [ ] **Error Handling** implementiert
- [ ] **Rollback** funktioniert

### Kubernetes-Qualität
- [ ] **Manifests** syntaktisch korrekt
- [ ] **ACR ImagePullSecrets** korrekt konfiguriert
- [ ] **Resource Limits** gesetzt
- [ ] **Health Checks** konfiguriert
- [ ] **Security Context** implementiert
- [ ] **Namespace** korrekt verwendet

### Azure ACR-Qualität
- [ ] **ACR** korrekt konfiguriert
- [ ] **Vulnerability Scanning** aktiviert
- [ ] **Retention Policies** konfiguriert
- [ ] **Service Principal** hat korrekte Permissions
- [ ] **Images** erfolgreich gepusht
- [ ] **ACR Access** funktioniert

## 🚨 Troubleshooting

### Häufige Probleme
- [ ] **"Failed to login to ACR"** - Service Principal Credentials prüfen
- [ ] **"ACR push failed"** - Service Principal Permissions prüfen
- [ ] **"ImagePullBackOff"** - ACR ImagePullSecrets prüfen
- [ ] **"kubectl connection refused"** - kubeconfig prüfen
- [ ] **"CrashLoopBackOff"** - Logs prüfen, Health Checks
- [ ] **"Workflow failed"** - GitHub Actions Logs prüfen
- [ ] **"Vulnerability scan failed"** - ACR Vulnerability Scanning prüfen

### Debug-Befehle
```bash
# Azure ACR
az acr login --name myregistry
az acr repository list --name myregistry
az acr repository show-tags --name myregistry --repository app-name

# Service Principal
az ad sp list --display-name "github-actions-myregistry"
az ad sp show --id {SERVICE_PRINCIPAL_ID}

# Kubernetes
kubectl get pods -n namespace
kubectl describe pod pod-name -n namespace
kubectl logs pod-name -n namespace

# GitHub Actions
# Repository → Actions → Workflow Run → Logs
```

## 📈 Erfolgs-Metriken

Nach erfolgreicher Migration solltet ihr haben:

- [ ] **Automatische Builds** - Jeder Push triggert Build
- [ ] **Automatische Tests** - Tests laufen bei jedem Build
- [ ] **Automatische Deployments** - Erfolgreiche Builds werden deployed
- [ ] **Private Registry** - Images sicher in ACR gespeichert
- [ ] **Vulnerability Scanning** - Automatische Sicherheitsprüfungen
- [ ] **Zero-Downtime** - Rolling Updates ohne Ausfallzeit
- [ ] **Rollback-Fähigkeit** - Schneller Rollback bei Problemen
- [ ] **Monitoring** - Überwachung von Pods und Services
- [ ] **Dokumentation** - Vollständige Deployment-Dokumentation

## 💰 Kosten-Übersicht

### Aktuelle Kosten
- **GitHub Actions:** 2000 Min/Monat kostenlos
- **Azure Container Registry:** $5/Monat (Basic Tier)
- **Azure Key Vault:** $0-5/Monat (je nach Nutzung)
- **Kubernetes:** Eure Hardware ($0)
- **Total: $5-10/Monat**

### Bei Skalierung
- **GitHub Actions:** $0.008/Min (sehr günstig)
- **ACR Standard:** $20/Monat (100GB Storage)
- **Azure Key Vault:** $5-20/Monat
- **Kubernetes:** Eure Hardware ($0)
- **Total: $25-50/Monat**

## 🎯 Nächste Schritte

### Sofort nach Migration
- [ ] **Team informieren** über neuen Workflow
- [ ] **Erste Deployments** durchführen
- [ ] **ACR Monitoring** einrichten
- [ ] **Vulnerability Scanning** überwachen
- [ ] **Dokumentation** vervollständigen

### Kurzfristig (1-2 Wochen)
- [ ] **Performance optimieren** - Resource Limits anpassen
- [ ] **Security härten** - Network Policies, Pod Security
- [ ] **ACR Retention Policies** optimieren
- [ ] **Monitoring erweitern** - Prometheus, Grafana
- [ ] **Backup-Strategie** - Datenbank-Backups

### Langfristig (1-3 Monate)
- [ ] **Multi-Environment** - Dev, Staging, Production
- [ ] **ACR Geo-replication** - Für Production
- [ ] **Advanced Monitoring** - Application Performance Monitoring
- [ ] **ACR Content Trust** - Digitale Signierung
- [ ] **Cost Optimization** - Resource Right-sizing

## 🔒 Security Best Practices

### ACR Security
- [ ] **Private Registry** - Keine öffentlichen Images
- [ ] **Vulnerability Scanning** - Automatische Sicherheitsprüfungen
- [ ] **Retention Policies** - Automatische Cleanup alter Images
- [ ] **Service Principal** - Minimale Permissions
- [ ] **Access Logs** - Überwachung aller Zugriffe

### Kubernetes Security
- [ ] **ImagePullSecrets** - Sichere Registry-Authentifizierung
- [ ] **Non-root User** - Container laufen nicht als root
- [ ] **Resource Limits** - Verhindert Ressourcen-Überlastung
- [ ] **Network Policies** - Eingeschränkte Netzwerk-Kommunikation
- [ ] **Pod Security** - Security Context implementiert

---

*Diese Checkliste ist darauf ausgelegt, mit dem Cursor Agent durchgeführt zu werden. Der Agent wird euch durch jeden Schritt führen und bei Problemen helfen.*
