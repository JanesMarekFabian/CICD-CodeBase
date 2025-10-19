# Azure DevOps + Azure ACR Migration Checklist

## 🎯 Übersicht

Diese Checkliste führt euch durch die Migration eines bestehenden Projekts zu einer Enterprise CI/CD-Pipeline mit Azure DevOps und Azure Container Registry (ACR). Der Cursor Agent wird euch durch jeden Schritt führen.

## 📋 Vorbereitung (25 Minuten)

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
- [ ] **Azure Container Registry** erstellt (Standard Tier - $20/Monat)
- [ ] **Azure Key Vault** erstellt (optional)
- [ ] **ACR Admin User** aktiviert
- [ ] **Vulnerability Scanning** aktiviert

### Phase 3: Azure DevOps Setup
- [ ] **Azure DevOps Organization** erstellt oder vorhanden
- [ ] **Azure DevOps Project** erstellt
- [ ] **Azure Repos** Repository erstellt
- [ ] **Azure Pipelines** aktiviert
- [ ] **Azure Artifacts** aktiviert (optional)

### Phase 4: Service Principal Setup
- [ ] **Service Principal** für ACR erstellt
- [ ] **Service Principal** für Key Vault erstellt (optional)
- [ ] **ACR Permissions** vergeben (acrpush, acrpull)
- [ ] **Key Vault Permissions** vergeben (optional)
- [ ] **Service Principal Credentials** notiert:
  - [ ] Application (client) ID
  - [ ] Client Secret Value
  - [ ] Directory (tenant) ID

## 🔧 Code-Generierung (40 Minuten)

### Phase 5: Dockerfile erstellen
- [ ] **Multi-Stage Dockerfile** generiert
- [ ] **Build Stage** - Dependencies installieren und kompilieren
- [ ] **Production Stage** - Nur Runtime und kompilierter Code
- [ ] **Non-root User** - Security Best Practice
- [ ] **Health Checks** - Liveness und Readiness Probes
- [ ] **.dockerignore** - Optimierte Ignore-Patterns
- [ ] **Dockerfile lokal getestet** - `docker build` erfolgreich

### Phase 6: Azure Pipelines
- [ ] **Build Pipeline** (azure-pipelines.yml)
  - [ ] Trigger: Push, Pull Request
  - [ ] Steps: Checkout → Setup → Build → Test → Docker → Push to ACR
  - [ ] ACR Integration
  - [ ] Artifact Publishing
- [ ] **Release Pipeline** (release-pipeline.yml)
  - [ ] Trigger: Build Artifact
  - [ ] Stages: Dev → Staging → Production
  - [ ] Approval Gates
  - [ ] Kubernetes Deployment
- [ ] **Manual Deploy Pipeline** (manual-deploy.yml)
  - [ ] Trigger: workflow_dispatch
  - [ ] Deploy spezifische Version
  - [ ] Rollback-Funktionalität

### Phase 7: Kubernetes Manifests
- [ ] **namespace.yaml** - Namespace und ConfigMap
- [ ] **secrets.yaml** - Secrets Template
- [ ] **acr-secret.yaml** - ACR ImagePullSecrets
- [ ] **deployment.yaml** - Deployment mit ACR Image
- [ ] **service.yaml** - Service für Netzwerk-Zugriff
- [ ] **ingress.yaml** - Optional für externe Zugriffe
- [ ] **hpa.yaml** - Horizontal Pod Autoscaler (optional)
- [ ] **Resource Limits** - CPU und Memory Limits
- [ ] **Health Checks** - Liveness und Readiness Probes

## 🚀 Setup und Konfiguration (30 Minuten)

### Phase 8: Azure ACR Setup
- [ ] **ACR erstellt** (Standard Tier - $20/Monat)
- [ ] **ACR Name** notiert (z.B. myregistry.azurecr.io)
- [ ] **ACR Login Server** notiert
- [ ] **Vulnerability Scanning** aktiviert
- [ ] **Retention Policies** konfiguriert (30 Tage)
- [ ] **Content Trust** aktiviert (optional)
- [ ] **ACR lokal getestet** - `az acr login`

### Phase 9: Azure Key Vault Setup (Optional)
- [ ] **Key Vault** erstellt
- [ ] **Secrets** in Key Vault gespeichert
- [ ] **Access Policies** konfiguriert
- [ ] **Service Principal** für Key Vault erstellt
- [ ] **Key Vault** lokal getestet

### Phase 10: Azure DevOps Konfiguration
- [ ] **Service Connections** erstellt:
  - [ ] Azure Container Registry
  - [ ] Azure Key Vault (optional)
  - [ ] Kubernetes (optional)
- [ ] **Variable Groups** erstellt:
  - [ ] Environment-spezifische Variablen
  - [ ] Registry-Konfiguration
  - [ ] Kubernetes-Konfiguration
- [ ] **Environments** erstellt:
  - [ ] Development
  - [ ] Staging
  - [ ] Production
- [ ] **Approval Gates** konfiguriert (optional)

### Phase 11: Service Principal Konfiguration
- [ ] **Service Principal** für ACR erstellt
- [ ] **ACR Push/Pull Permissions** vergeben
- [ ] **Key Vault Permissions** vergeben (optional)
- [ ] **Service Principal Credentials** gesichert
- [ ] **Service Principal** lokal getestet
- [ ] **ACR Access** mit Service Principal getestet

### Phase 12: Kubernetes Setup
- [ ] **Namespace** in Kubernetes erstellt
- [ ] **ACR ImagePullSecrets** in Kubernetes erstellt
- [ ] **Secrets** in Kubernetes erstellt (Application Secrets)
- [ ] **ConfigMap** in Kubernetes erstellt
- [ ] **kubeconfig** für Azure Pipelines konfiguriert
- [ ] **RBAC** für Azure Pipelines konfiguriert

### Phase 13: Azure Repos Setup
- [ ] **Code** zu Azure Repos gepusht
- [ ] **Pipeline-Dateien** committet
- [ ] **Kubernetes Manifests** committet
- [ ] **Dockerfile** committet
- [ ] **.dockerignore** committet
- [ ] **Branch Policies** konfiguriert (optional)
- [ ] **Required Status Checks** aktiviert (optional)

## 🧪 Testing und Validierung (25 Minuten)

### Phase 14: Lokales Testing
- [ ] **Dockerfile lokal gebaut** - `docker build -t test-app .`
- [ ] **Docker Image lokal getestet** - `docker run -p 3000:3000 test-app`
- [ ] **ACR Login lokal getestet** - `az acr login`
- [ ] **Image zu ACR gepusht** - `docker push myregistry.azurecr.io/test-app:latest`
- [ ] **Health Endpoints** funktionieren - `/health`, `/ready`
- [ ] **Environment Variables** korrekt gesetzt
- [ ] **Application** läuft ohne Fehler

### Phase 15: Azure Pipelines Testing
- [ ] **Ersten Push** zu Azure Repos durchgeführt
- [ ] **Build Pipeline** erfolgreich gelaufen
- [ ] **Release Pipeline** erfolgreich gelaufen
- [ ] **Docker Image** zu ACR gepusht
- [ ] **Vulnerability Scanning** erfolgreich
- [ ] **Kubernetes Deployment** erfolgreich
- [ ] **Application** im Browser/API erreichbar

### Phase 16: End-to-End Testing
- [ ] **Application** funktioniert korrekt
- [ ] **ACR Images** in Azure Portal sichtbar
- [ ] **Vulnerability Scan Results** überprüft
- [ ] **Azure Pipelines Logs** überprüft
- [ ] **Logs** in Kubernetes überprüft
- [ ] **Pod Status** - alle Pods "Running"
- [ ] **Service** funktioniert
- [ ] **Rollback** getestet (optional)

## 📊 Monitoring und Wartung (20 Minuten)

### Phase 17: Monitoring Setup
- [ ] **Pod Status** überwachen - `kubectl get pods`
- [ ] **Service Status** überwachen - `kubectl get services`
- [ ] **Deployment Status** überwachen - `kubectl rollout status`
- [ ] **Logs** überwachen - `kubectl logs -f deployment/app-name`
- [ ] **Events** überwachen - `kubectl get events`
- [ ] **ACR Usage** überwachen - Azure Portal
- [ ] **Vulnerability Scan Results** überwachen
- [ ] **Azure Pipelines** überwachen - Azure DevOps Portal

### Phase 18: Dokumentation
- [ ] **README.md** aktualisiert mit Deployment-Anleitung
- [ ] **Azure DevOps Setup** dokumentiert
- [ ] **ACR Setup** dokumentiert
- [ ] **Service Principal Setup** dokumentiert
- [ ] **Key Vault Setup** dokumentiert (optional)
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

### Azure Pipelines-Qualität
- [ ] **Pipelines** syntaktisch korrekt
- [ ] **Azure ACR Integration** korrekt konfiguriert
- [ ] **Service Connections** korrekt konfiguriert
- [ ] **Variable Groups** korrekt konfiguriert
- [ ] **Environments** korrekt konfiguriert
- [ ] **Approval Gates** konfiguriert (optional)
- [ ] **Error Handling** implementiert
- [ ] **Rollback** funktioniert

### Kubernetes-Qualität
- [ ] **Manifests** syntaktisch korrekt
- [ ] **ACR ImagePullSecrets** korrekt konfiguriert
- [ ] **Resource Limits** gesetzt
- [ ] **Health Checks** konfiguriert
- [ ] **Security Context** implementiert
- [ ] **Namespace** korrekt verwendet
- [ ] **HPA** konfiguriert (optional)

### Azure ACR-Qualität
- [ ] **ACR** korrekt konfiguriert
- [ ] **Vulnerability Scanning** aktiviert
- [ ] **Retention Policies** konfiguriert
- [ ] **Content Trust** aktiviert (optional)
- [ ] **Service Principal** hat korrekte Permissions
- [ ] **Images** erfolgreich gepusht
- [ ] **ACR Access** funktioniert

### Azure Key Vault-Qualität (Optional)
- [ ] **Key Vault** korrekt konfiguriert
- [ ] **Secrets** korrekt gespeichert
- [ ] **Access Policies** korrekt konfiguriert
- [ ] **Service Principal** hat korrekte Permissions
- [ ] **Key Vault Access** funktioniert

## 🚨 Troubleshooting

### Häufige Probleme
- [ ] **"Failed to login to ACR"** - Service Principal Credentials prüfen
- [ ] **"ACR push failed"** - Service Principal Permissions prüfen
- [ ] **"ImagePullBackOff"** - ACR ImagePullSecrets prüfen
- [ ] **"kubectl connection refused"** - kubeconfig prüfen
- [ ] **"CrashLoopBackOff"** - Logs prüfen, Health Checks
- [ ] **"Pipeline failed"** - Azure Pipelines Logs prüfen
- [ ] **"Vulnerability scan failed"** - ACR Vulnerability Scanning prüfen
- [ ] **"Key Vault access failed"** - Key Vault Permissions prüfen

### Debug-Befehle
```bash
# Azure ACR
az acr login --name myregistry
az acr repository list --name myregistry
az acr repository show-tags --name myregistry --repository app-name

# Service Principal
az ad sp list --display-name "azure-devops-myregistry"
az ad sp show --id {SERVICE_PRINCIPAL_ID}

# Azure Key Vault
az keyvault secret list --vault-name mykeyvault
az keyvault secret show --vault-name mykeyvault --name mysecret

# Kubernetes
kubectl get pods -n namespace
kubectl describe pod pod-name -n namespace
kubectl logs pod-name -n namespace

# Azure Pipelines
# Azure DevOps → Pipelines → Build/Release → Logs
```

## 📈 Erfolgs-Metriken

Nach erfolgreicher Migration solltet ihr haben:

- [ ] **Automatische Builds** - Jeder Push triggert Build
- [ ] **Automatische Tests** - Tests laufen bei jedem Build
- [ ] **Automatische Deployments** - Erfolgreiche Builds werden deployed
- [ ] **Private Registry** - Images sicher in ACR gespeichert
- [ ] **Vulnerability Scanning** - Automatische Sicherheitsprüfungen
- [ ] **Release Management** - Sophisticated Release Pipelines
- [ ] **Environment Approvals** - Kontrollierte Deployments
- [ ] **Zero-Downtime** - Rolling Updates ohne Ausfallzeit
- [ ] **Rollback-Fähigkeit** - Schneller Rollback bei Problemen
- [ ] **Monitoring** - Überwachung von Pods und Services
- [ ] **Dokumentation** - Vollständige Deployment-Dokumentation

## 💰 Kosten-Übersicht

### Aktuelle Kosten
- **Azure DevOps:** $6/User/Monat (ab 6. User)
- **Azure Container Registry:** $20/Monat (Standard Tier)
- **Azure Key Vault:** $0-20/Monat (je nach Nutzung)
- **Kubernetes:** Eure Hardware ($0)
- **Total: $26-46/Monat**

### Bei Skalierung
- **Azure DevOps:** $6/User/Monat + $0.04/Min Build
- **ACR Premium:** $50/Monat (500GB Storage)
- **Azure Key Vault:** $20-100/Monat
- **Kubernetes:** Eure Hardware ($0)
- **Total: $76-156/Monat**

## 🎯 Nächste Schritte

### Sofort nach Migration
- [ ] **Team informieren** über neuen Workflow
- [ ] **Erste Deployments** durchführen
- [ ] **ACR Monitoring** einrichten
- [ ] **Vulnerability Scanning** überwachen
- [ ] **Azure Pipelines** überwachen
- [ ] **Dokumentation** vervollständigen

### Kurzfristig (1-2 Wochen)
- [ ] **Performance optimieren** - Resource Limits anpassen
- [ ] **Security härten** - Network Policies, Pod Security
- [ ] **ACR Retention Policies** optimieren
- [ ] **Monitoring erweitern** - Prometheus, Grafana
- [ ] **Backup-Strategie** - Datenbank-Backups
- [ ] **Release Management** optimieren

### Langfristig (1-3 Monate)
- [ ] **Multi-Environment** - Dev, Staging, Production
- [ ] **ACR Geo-replication** - Für Production
- [ ] **Advanced Monitoring** - Application Performance Monitoring
- [ ] **ACR Content Trust** - Digitale Signierung
- [ ] **Cost Optimization** - Resource Right-sizing
- [ ] **Enterprise Features** - Advanced Security, Compliance

## 🔒 Security Best Practices

### Azure ACR Security
- [ ] **Private Registry** - Keine öffentlichen Images
- [ ] **Vulnerability Scanning** - Automatische Sicherheitsprüfungen
- [ ] **Retention Policies** - Automatische Cleanup alter Images
- [ ] **Content Trust** - Digitale Signierung (optional)
- [ ] **Service Principal** - Minimale Permissions
- [ ] **Access Logs** - Überwachung aller Zugriffe

### Azure Key Vault Security (Optional)
- [ ] **Secrets Management** - Zentrale Secrets-Verwaltung
- [ ] **Access Policies** - Granulare Zugriffskontrolle
- [ ] **Audit Logs** - Vollständige Nachverfolgung
- [ ] **Automatic Rotation** - Scheduled Secret Updates
- [ ] **Network Security** - Private Endpoints (optional)

### Kubernetes Security
- [ ] **ImagePullSecrets** - Sichere Registry-Authentifizierung
- [ ] **Non-root User** - Container laufen nicht als root
- [ ] **Resource Limits** - Verhindert Ressourcen-Überlastung
- [ ] **Network Policies** - Eingeschränkte Netzwerk-Kommunikation
- [ ] **Pod Security** - Security Context implementiert

### Azure DevOps Security
- [ ] **Service Connections** - Sichere Verbindungen zu Azure
- [ ] **Variable Groups** - Sichere Variablen-Verwaltung
- [ ] **Environments** - Kontrollierte Deployment-Umgebungen
- [ ] **Approval Gates** - Manuelle Freigaben für kritische Deployments
- [ ] **Audit Logs** - Überwachung aller Pipeline-Aktivitäten

---

*Diese Checkliste ist darauf ausgelegt, mit dem Cursor Agent durchgeführt zu werden. Der Agent wird euch durch jeden Schritt führen und bei Problemen helfen.*
