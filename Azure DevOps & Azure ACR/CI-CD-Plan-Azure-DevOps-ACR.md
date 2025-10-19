# CI/CD Pipeline Plan: Azure DevOps + Azure Container Registry

## 🎯 Übersicht & Zielsetzung

### Was wird erreicht?
**Automatisierung von:** Code-Änderung → Build → Test → Deploy auf lokales Kubernetes Cluster

**Identisch zu GitHub Actions, aber Azure-native:**
- Azure Repos als Code-Speicher
- Azure Pipelines als CI/CD Engine
- Azure Container Registry (ACR) als Image Registry
- Azure Key Vault für Secrets Management
- Service Principal für sichere Kubernetes-Integration

### Wann ist diese Lösung sinnvoll?
- ✅ **Enterprise-Umgebung** mit Microsoft-Ökosystem
- ✅ **Skalierbarkeit** für große Teams und Projekte
- ✅ **Sicherheit** mit Azure-native Features
- ✅ **Integration** mit anderen Azure-Services
- ✅ **Compliance** und Audit-Anforderungen

### Vor- und Nachteile vs GitHub Actions

**Vorteile:**
- **Enterprise-Features:** Advanced Security, Compliance, Audit Logs
- **Azure-Integration:** Nahtlose Integration mit Azure-Services
- **Private Registry:** ACR mit Vulnerability Scanning
- **Managed Secrets:** Azure Key Vault Integration
- **Scalability:** Unbegrenzte Build-Minuten (gegen Bezahlung)
- **Release Management:** Sophisticated Release Pipelines

**Nachteile:**
- **Kosten:** $6/User/Monat + Build-Minuten
- **Vendor Lock-in:** Microsoft-Ökosystem
- **Komplexität:** Mehr Konfiguration erforderlich
- **Learning Curve:** Azure-spezifische Konzepte

---

## 🏗️ Architektur-Diagramm

```
Developer (Lokal)
    ↓ git push
Azure Repos
    ↓ Trigger (Push/PR)
Azure Pipelines
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

### Service Principal Integration
- **Azure AD:** Service Principal für Kubernetes-Authentifizierung
- **ACR:** Managed Identity für Registry-Zugriff
- **Key Vault:** Service Principal für Secrets-Zugriff
- **RBAC:** Granulare Berechtigungen pro Service

### Azure Key Vault für Secrets
- **Zentrale Verwaltung:** Alle Secrets an einem Ort
- **Automatische Rotation:** Scheduled Secret Updates
- **Audit Logs:** Vollständige Nachverfolgung
- **Access Policies:** Granulare Zugriffskontrolle

---

## 📦 Komponenten-Übersicht

### 1. Azure Repos (Code-Speicher)
- **Zweck:** Zentrale Code-Verwaltung im Azure-Ökosystem
- **Features:** Git-Repository, Branch Policies, Pull Request Workflows
- **Integration:** Nahtlose Verbindung zu Azure Pipelines
- **Security:** Azure AD-basierte Zugriffskontrolle

### 2. Azure Pipelines (CI/CD Engine)
- **Zweck:** Enterprise CI/CD mit erweiterten Features
- **Laufzeit:** Microsoft-gehostete oder Self-hosted Agents
- **Free Tier:** 1800 Minuten/Monat, 5 Benutzer
- **Features:** Release Management, Environment Approvals, Artifact Management

### 3. Azure Container Registry (Image Registry)
- **Zweck:** Private Container Registry mit Enterprise-Features
- **Features:** Vulnerability Scanning, Geo-replication, Content Trust
- **Security:** Private Registry, Azure AD Integration
- **Pricing:** $5/Monat (Basic), $20/Monat (Standard)

### 4. Azure Key Vault (Secrets Management)
- **Zweck:** Zentrale, sichere Secrets-Verwaltung
- **Features:** Automatic Rotation, Audit Logs, Access Policies
- **Integration:** Nahtlose Verbindung zu Azure Pipelines
- **Security:** Hardware Security Modules (HSM)

### 5. Service Principal (Auth für Kubernetes)
- **Zweck:** Sichere Authentifizierung zwischen Azure und Kubernetes
- **Features:** Certificate-based Auth, Role-based Permissions
- **Security:** Keine Passwörter, automatische Rotation
- **Integration:** Azure AD und Kubernetes RBAC

### 6. Kubernetes Cluster (Deployment-Ziel)
- **Zweck:** Container Orchestrierung (lokal oder AKS)
- **Zugriff:** Service Principal-basierte Authentifizierung
- **Netzwerk:** 192.168.2.x (generisch konfigurierbar)
- **Integration:** Azure Monitor für Observability

---

## 🔄 Implementierungs-Phasen

### Phase 1: Azure-Setup (30 Minuten)
**Ziel:** Azure-Umgebung und Services einrichten

**Schritte:**
1. **Azure Subscription** aktivieren/konfigurieren
2. **Resource Group** erstellen für CI/CD Services
3. **Azure Container Registry** erstellen (Basic Tier)
4. **Azure Key Vault** erstellen
5. **Azure DevOps Organization** einrichten
6. **Azure Repos** Repository erstellen

**Benötigte Azure Services:**
- Azure Container Registry (ACR)
- Azure Key Vault
- Azure DevOps (Repos + Pipelines)
- Azure Active Directory (für Service Principal)

### Phase 2: Service Principal erstellen (20 Minuten)
**Zweck:** Sichere Authentifizierung zwischen Azure und Kubernetes

**Service Principal für ACR:**
```bash
# ACR Pull Permissions
az ad sp create-for-rbac --name "k8s-acr-sp" \
  --role acrpull \
  --scopes /subscriptions/{subscription-id}/resourceGroups/{rg}/providers/Microsoft.ContainerRegistry/registries/{acr-name}
```

**Service Principal für Key Vault:**
```bash
# Key Vault Secrets User
az ad sp create-for-rbac --name "k8s-kv-sp" \
  --role "Key Vault Secrets User" \
  --scopes /subscriptions/{subscription-id}/resourceGroups/{rg}/providers/Microsoft.KeyVault/vaults/{kv-name}
```

**Outputs notieren:**
- Application (client) ID
- Client Secret Value
- Tenant ID

### Phase 3: Azure Pipeline konfigurieren (45 Minuten)
**Zweck:** CI/CD Pipeline mit Azure-native Features

**Pipeline-Komponenten:**
1. **Build Pipeline:** Code → Docker Image → ACR
2. **Release Pipeline:** ACR → Kubernetes Deployment
3. **Variable Groups:** Environment-spezifische Konfiguration
4. **Service Connections:** ACR, Kubernetes, Key Vault
5. **Environments:** Dev, Staging, Production mit Approvals

**Pipeline-Features:**
- **Multi-Stage Pipelines:** Build → Test → Deploy
- **Parallel Jobs:** Mehrere Services gleichzeitig
- **Conditional Steps:** Environment-spezifische Aktionen
- **Artifact Management:** Docker Images und Deployment Manifests

### Phase 4: ACR Integration (25 Minuten)
**Zweck:** Private Container Registry mit Security Features

**ACR-Konfiguration:**
1. **Admin User** aktivieren (für lokale Entwicklung)
2. **Vulnerability Scanning** aktivieren
3. **Content Trust** konfigurieren (optional)
4. **Geo-replication** (für Production, optional)

**Image-Management:**
- **Tagging Strategy:** latest, {build-id}, {version}
- **Retention Policies:** Automatische Cleanup alter Images
- **Webhooks:** Benachrichtigungen bei neuen Images
- **Quarantine:** Images vor Deployment prüfen

### Phase 5: Kubernetes-Verbindung (30 Minuten)
**Zweck:** Service Connection zwischen Azure und Kubernetes

**Service Connection Setup:**
1. **Azure DevOps** → Project Settings → Service Connections
2. **Kubernetes** Service Connection erstellen
3. **Service Principal** für Kubernetes-Authentifizierung
4. **kubeconfig** mit Service Principal konfigurieren

**Kubernetes RBAC:**
```yaml
# Service Account für Azure Pipelines
apiVersion: v1
kind: ServiceAccount
metadata:
  name: azure-pipelines-sa
  namespace: default
---
# ClusterRole mit minimalen Permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: azure-pipelines-role
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "update", "patch"]
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: azure-pipelines-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: azure-pipelines-role
subjects:
- kind: ServiceAccount
  name: azure-pipelines-sa
  namespace: default
```

### Phase 6: Deployment-Pipeline (35 Minuten)
**Zweck:** Automatisierte Deployments mit Release Management

**Release Pipeline Features:**
1. **Multi-Environment:** Dev → Staging → Production
2. **Approval Gates:** Manuelle Freigaben für Production
3. **Deployment Strategies:** Rolling Updates, Blue-Green
4. **Rollback:** Automatisch bei fehlgeschlagenen Deployments
5. **Monitoring:** Health Checks und Alerts

**Deployment-Patterns:**
- **Blue-Green:** Zero-downtime Deployments
- **Canary:** Gradual Rollouts
- **Rolling Updates:** Standard Kubernetes Strategy
- **Feature Flags:** Runtime Feature Control

### Phase 7: Release Management (20 Minuten)
**Zweck:** Sophisticated Release-Prozesse

**Release Management Features:**
1. **Release Gates:** Automatische Qualitätsprüfungen
2. **Approval Workflows:** Multi-stage Approvals
3. **Deployment History:** Vollständige Audit Trails
4. **Rollback Automation:** Ein-Klick Rollbacks
5. **Release Notes:** Automatische Release Documentation

**Environment-Strategien:**
- **Development:** Automatische Deployments
- **Staging:** Automatisch nach erfolgreichen Tests
- **Production:** Manuelle Approval erforderlich
- **Hotfix:** Beschleunigte Pipeline für kritische Fixes

---

## 🔐 Sicherheitskonzepte

### Azure Key Vault Integration
**Zweck:** Zentrale, sichere Secrets-Verwaltung

**Features:**
- **Hardware Security Modules (HSM):** Hardware-basierte Verschlüsselung
- **Automatic Rotation:** Scheduled Secret Updates
- **Access Policies:** Granulare Zugriffskontrolle
- **Audit Logs:** Vollständige Nachverfolgung aller Zugriffe

**Integration mit Azure Pipelines:**
1. **Key Vault Service Connection** erstellen
2. **Variable Groups** mit Key Vault-Referenzen
3. **Automatic Secret Injection** in Pipeline
4. **Secret Masking** in Logs

### Service Principal Permissions
**Zweck:** Minimale Berechtigungen für sichere Integration

**ACR Permissions:**
- **AcrPull:** Images aus ACR ziehen
- **AcrPush:** Images zu ACR pushen (nur für Pipelines)
- **AcrDelete:** Alte Images löschen (Cleanup)

**Key Vault Permissions:**
- **Key Vault Secrets User:** Secrets lesen
- **Key Vault Secrets Officer:** Secrets verwalten (nur für Pipelines)

**Kubernetes Permissions:**
- **Deployments:** Erstellen, aktualisieren, löschen
- **Services:** Verwalten von Services
- **ConfigMaps/Secrets:** Lesen und erstellen
- **Pods/Events:** Lesen für Monitoring

### ACR Private Registry
**Zweck:** Sichere, private Container Images

**Security Features:**
- **Private Endpoints:** Kein öffentlicher Zugriff
- **Azure AD Integration:** Benutzer-basierte Authentifizierung
- **Vulnerability Scanning:** Automatische Sicherheitsprüfungen
- **Content Trust:** Digitale Signierung von Images

**Network Security:**
- **Virtual Network Integration:** ACR in VNet
- **Private Endpoints:** Kein Internet-Zugriff erforderlich
- **Firewall Rules:** IP-basierte Zugriffskontrolle

### Managed Identities
**Zweck:** Passwortlose Authentifizierung

**System-assigned Managed Identity:**
- Automatisch erstellt und verwaltet
- Lebenszyklus an Azure-Ressource gebunden
- Keine Credentials zu verwalten

**User-assigned Managed Identity:**
- Manuell erstellt und verwaltet
- Kann mehreren Azure-Ressourcen zugewiesen werden
- Flexiblere Verwaltung

### Network Security
**Zweck:** Netzwerk-basierte Sicherheit

**Azure VNet Integration:**
- **Private Endpoints:** ACR und Key Vault in VNet
- **Network Security Groups:** Traffic-Filterung
- **Azure Firewall:** Zentralisierte Firewall-Regeln

**Kubernetes Network Policies:**
- **Pod-to-Pod Communication:** Eingeschränkte Kommunikation
- **Ingress/Egress Rules:** Kontrollierter Datenverkehr
- **Service Mesh:** Advanced Network Security (optional)

---

## 💰 Kosten-Übersicht

### Free Tier Limits
**Azure DevOps:**
- 5 Benutzer kostenlos
- 1800 Build-Minuten/Monat kostenlos
- Unbegrenzte private Repositories

**Azure Container Registry:**
- Kein Free Tier
- Basic: $5/Monat (10GB Storage)
- Standard: $20/Monat (100GB Storage)

**Azure Key Vault:**
- 10.000 Transaktionen/Monat kostenlos
- $0.03 pro 10.000 Transaktionen danach

### Skalierungs-Kosten

**Kleine Teams (5-10 Devs):**
- **Azure DevOps:** $0-30/Monat (Free Tier)
- **ACR Basic:** $5/Monat
- **Key Vault:** $0-5/Monat
- **Total: $5-40/Monat**

**Mittlere Teams (20-50 Devs):**
- **Azure DevOps:** $90-300/Monat
- **ACR Standard:** $20/Monat
- **Key Vault:** $5-20/Monat
- **Total: $115-340/Monat**

**Große Teams (100+ Devs):**
- **Azure DevOps:** $600-2000/Monat
- **ACR Premium:** $50/Monat
- **Key Vault:** $20-100/Monat
- **Total: $670-2150/Monat**

### Vergleich zu GitHub Actions
| Feature | GitHub Actions | Azure DevOps |
|---------|----------------|--------------|
| **Free Tier** | 2000 Min/Monat | 1800 Min/Monat |
| **Paid Build Minutes** | $0.008/Min | $0.04/Min |
| **User Cost** | $0 | $6/User/Monat |
| **Private Registry** | Docker Hub Pro ($5/Monat) | ACR ($5/Monat) |
| **Secrets Management** | GitHub Secrets | Azure Key Vault |
| **Release Management** | Basic | Advanced |

---

## 📁 Dateien-Struktur

### Azure DevOps Repository-Struktur
```
project-root/
├── azure-pipelines.yml          # Build Pipeline
├── release-pipeline.yml         # Release Pipeline
├── environments/
│   ├── dev/
│   │   ├── configmap.yaml
│   │   └── secrets.yaml
│   ├── staging/
│   │   ├── configmap.yaml
│   │   └── secrets.yaml
│   └── production/
│       ├── configmap.yaml
│       └── secrets.yaml
├── k8s/
│   ├── namespace.yaml
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   └── services.yaml
├── apps/
│   ├── backend/
│   │   ├── Dockerfile
│   │   └── src/
│   └── frontend/
│       ├── Dockerfile
│       └── src/
└── scripts/
    ├── deploy.sh
    └── rollback.sh
```

### Azure Pipeline-Dateien
**`azure-pipelines.yml`**
- **Trigger:** Push auf main branch, Pull Requests
- **Stages:** Build → Test → Push to ACR
- **Features:** Multi-stage, Parallel Jobs, Artifact Publishing

**`release-pipeline.yml`**
- **Trigger:** Artifact aus Build Pipeline
- **Stages:** Dev → Staging → Production
- **Features:** Approval Gates, Deployment Strategies, Rollback

### Kubernetes Manifests mit ACR
**`k8s/namespace.yaml`**
- Namespace-Definition
- Azure-spezifische Labels und Annotations

**`k8s/*-deployment.yaml`**
- Deployment-Definitionen mit ACR Images
- Image Tags als Pipeline-Variablen
- Resource Limits und Health Checks

**`k8s/services.yaml`**
- Service-Definitionen
- Load Balancing und Networking

### Environment-spezifische Configs
**`environments/{env}/configmap.yaml`**
- Environment-spezifische Konfiguration
- Nicht-sensitive Daten

**`environments/{env}/secrets.yaml`**
- Secret-Templates
- Werte werden via Azure Key Vault injected

---

## 🔄 Migration-Pfad

### Von GitHub Actions zu Azure DevOps
**Schritt 1: Repository Migration**
1. **Azure Repos** Repository erstellen
2. **Code** von GitHub zu Azure Repos migrieren
3. **Branch Policies** konfigurieren
4. **Pull Request Workflows** einrichten

**Schritt 2: Pipeline Migration**
1. **Build Pipeline** von GitHub Actions zu Azure Pipelines
2. **Secrets** von GitHub Secrets zu Azure Key Vault
3. **Variables** zu Variable Groups migrieren
4. **Environments** und Approval Gates einrichten

**Schritt 3: Registry Migration**
1. **ACR** erstellen und konfigurieren
2. **Images** von Docker Hub zu ACR migrieren
3. **Service Principal** für ACR-Zugriff erstellen
4. **Kubernetes** auf ACR umstellen

### Von Docker Hub zu ACR
**Schritt 1: ACR Setup**
1. **Azure Container Registry** erstellen
2. **Admin User** aktivieren
3. **Vulnerability Scanning** aktivieren
4. **Retention Policies** konfigurieren

**Schritt 2: Image Migration**
1. **Images** von Docker Hub zu ACR pushen
2. **Tags** und **Versions** beibehalten
3. **Kubernetes Manifests** auf ACR umstellen
4. **Service Principal** für Pull-Zugriff konfigurieren

**Schritt 3: Pipeline Update**
1. **Docker Login** auf ACR umstellen
2. **Image Tags** auf ACR-Format anpassen
3. **Pull Secrets** in Kubernetes aktualisieren
4. **Monitoring** auf ACR-Metriken umstellen

### Hybrid-Ansätze
**Option 1: Dual Registry**
- **Development:** Docker Hub (kostenlos)
- **Production:** ACR (sicher)

**Option 2: Gradual Migration**
- **Phase 1:** GitHub Actions + Docker Hub
- **Phase 2:** Azure Pipelines + Docker Hub
- **Phase 3:** Azure Pipelines + ACR

**Option 3: Multi-Cloud**
- **Azure:** ACR + Azure Pipelines
- **AWS:** ECR + CodePipeline
- **Google:** GCR + Cloud Build

---

## 🏆 Best Practices

### Pipeline-Templates
**Zweck:** Wiederverwendbare Pipeline-Komponenten

**Template-Typen:**
- **Build Templates:** Standardisierte Build-Prozesse
- **Deploy Templates:** Wiederverwendbare Deployment-Logik
- **Test Templates:** Einheitliche Test-Frameworks
- **Security Templates:** Security Scanning und Compliance

**Template-Struktur:**
```yaml
# templates/build-template.yml
parameters:
- name: serviceName
  type: string
- name: dockerfilePath
  type: string
  default: 'Dockerfile'

steps:
- task: Docker@2
  inputs:
    command: 'build'
    dockerfile: '${{ parameters.dockerfilePath }}'
    tags: |
      $(ACR_NAME).azurecr.io/${{ parameters.serviceName }}:$(Build.BuildId)
```

### Variable Groups
**Zweck:** Zentrale Konfiguration für verschiedene Umgebungen

**Variable Group-Typen:**
- **Environment-specific:** Dev, Staging, Production
- **Service-specific:** Backend, Frontend, Database
- **Security:** Secrets und Credentials
- **Configuration:** URLs, Ports, Feature Flags

**Variable Group-Struktur:**
```yaml
# Variable Group: "Production-Config"
variables:
- group: Production-Config
- name: ACR_NAME
  value: 'myregistry'
- name: K8S_NAMESPACE
  value: 'production'
```

### Environment Approvals
**Zweck:** Kontrollierte Deployments in kritische Umgebungen

**Approval-Typen:**
- **Manual Approval:** Benutzer muss explizit genehmigen
- **Automated Approval:** Basierend auf Tests und Metriken
- **Scheduled Approval:** Deployments nur zu bestimmten Zeiten
- **Multi-stage Approval:** Mehrere Personen müssen genehmigen

**Approval-Konfiguration:**
1. **Environment** in Azure DevOps erstellen
2. **Approval Gates** konfigurieren
3. **Approvers** definieren
4. **Timeout** für Approvals setzen

### Artifact Management
**Zweck:** Zentrale Verwaltung von Build-Artefakten

**Artifact-Typen:**
- **Docker Images:** Container Images in ACR
- **Deployment Manifests:** Kubernetes YAML-Dateien
- **Test Reports:** Test-Ergebnisse und Coverage
- **Release Notes:** Automatisch generierte Release-Dokumentation

**Artifact-Strategien:**
- **Retention Policies:** Automatische Cleanup alter Artefakte
- **Versioning:** Semantische Versionierung für Releases
- **Promotion:** Artefakte zwischen Umgebungen fördern
- **Rollback:** Schneller Zugriff auf vorherige Versionen

---

## 🔧 Troubleshooting

### Service Principal Probleme
**"Authentication failed for Service Principal"**
- **Ursache:** Service Principal Credentials falsch oder abgelaufen
- **Lösung:** Service Principal neu erstellen, Credentials aktualisieren
- **Prävention:** Automatische Rotation von Service Principal Secrets

**"Insufficient permissions for Service Principal"**
- **Ursache:** Service Principal hat nicht genügend Berechtigungen
- **Lösung:** RBAC-Rollen für Service Principal erweitern
- **Prävention:** Least-Privilege-Prinzip befolgen, Permissions dokumentieren

### ACR Auth-Fehler
**"Failed to push to ACR"**
- **Ursache:** ACR Admin User nicht aktiviert oder falsche Credentials
- **Lösung:** ACR Admin User aktivieren, Credentials in Pipeline aktualisieren
- **Prävention:** Managed Identity für ACR-Zugriff verwenden

**"Image pull failed from ACR"**
- **Ursache:** Kubernetes kann nicht auf ACR zugreifen
- **Lösung:** ImagePullSecrets in Kubernetes konfigurieren
- **Prävention:** Service Principal für Kubernetes ACR-Zugriff einrichten

### Pipeline-Debugging
**"Pipeline step failed"**
- **Ursache:** Verschiedene mögliche Ursachen
- **Lösung:** Pipeline-Logs analysieren, Debug-Modus aktivieren
- **Prävention:** Umfassende Tests vor Pipeline-Deployment

**"Variable not found"**
- **Ursache:** Variable Group nicht verknüpft oder Variable nicht definiert
- **Lösung:** Variable Groups überprüfen, Variablen definieren
- **Prävention:** Variable Groups dokumentieren, Standard-Werte definieren

### Azure Key Vault Probleme
**"Failed to retrieve secret from Key Vault"**
- **Ursache:** Service Principal hat keine Berechtigung für Key Vault
- **Lösung:** Key Vault Access Policy für Service Principal erstellen
- **Prävention:** Managed Identity für Key Vault-Zugriff verwenden

**"Secret not found in Key Vault"**
- **Ursache:** Secret existiert nicht oder falscher Name
- **Lösung:** Secret in Key Vault erstellen, Namen überprüfen
- **Prävention:** Secret-Namen dokumentieren, Standard-Werte definieren

### Kubernetes Integration Probleme
**"kubectl connection refused"**
- **Ursache:** Service Connection falsch konfiguriert oder Kubernetes nicht erreichbar
- **Lösung:** Service Connection überprüfen, kubeconfig validieren
- **Prävention:** Service Connection testen, Netzwerk-Konnektivität prüfen

**"Deployment failed in Kubernetes"**
- **Ursache:** Kubernetes-Manifest fehlerhaft oder Ressourcen nicht verfügbar
- **Lösung:** Kubernetes-Logs analysieren, Manifest validieren
- **Prävention:** Manifest-Tests, Resource-Quotas überprüfen

---

## 📊 Monitoring & Observability

### Azure Monitor Integration
**Zweck:** Zentrale Überwachung aller Azure-Services

**Azure Monitor Features:**
- **Application Insights:** Application Performance Monitoring
- **Log Analytics:** Zentrale Log-Aggregation
- **Metrics:** Performance-Metriken für alle Services
- **Alerts:** Proaktive Benachrichtigungen bei Problemen

**Integration mit Azure Pipelines:**
1. **Application Insights** in Pipeline integrieren
2. **Custom Metrics** für Deployment-Status
3. **Alerts** für fehlgeschlagene Deployments
4. **Dashboards** für Pipeline-Übersicht

### ACR Monitoring
**Zweck:** Überwachung der Container Registry

**ACR-Metriken:**
- **Image Push/Pull Rates:** Registry-Nutzung
- **Storage Usage:** Verbrauchter Speicherplatz
- **Vulnerability Scans:** Sicherheitsstatus der Images
- **Retention Policies:** Cleanup-Aktivitäten

**Monitoring-Tools:**
- **Azure Monitor:** Native ACR-Metriken
- **Azure Security Center:** Vulnerability Management
- **Custom Dashboards:** Grafana mit Azure Data Source

### Pipeline Monitoring
**Zweck:** Überwachung der CI/CD-Pipeline

**Pipeline-Metriken:**
- **Build Success Rate:** Erfolgsrate der Builds
- **Deployment Duration:** Zeit für Deployments
- **Test Coverage:** Code-Coverage-Metriken
- **Artifact Size:** Größe der generierten Artefakte

**Monitoring-Strategien:**
- **Pipeline Analytics:** Azure DevOps Analytics
- **Custom Dashboards:** Power BI mit Azure DevOps Data
- **Alerts:** Fehlgeschlagene Builds und Deployments
- **Reports:** Wöchentliche/Monatliche Pipeline-Reports

### Kubernetes Monitoring
**Zweck:** Überwachung des Kubernetes Clusters

**Kubernetes-Metriken:**
- **Pod Status:** Running, Pending, Failed Pods
- **Resource Usage:** CPU, Memory, Storage
- **Deployment Status:** Rollout-Status und Health
- **Service Health:** Service-Verfügbarkeit

**Monitoring-Integration:**
- **Azure Monitor for Containers:** Native Kubernetes-Monitoring
- **Prometheus + Grafana:** Custom Monitoring-Stack
- **Application Insights:** Application-level Monitoring
- **Azure Log Analytics:** Zentrale Log-Aggregation

---

## 🚀 Nächste Schritte

### Phase 1: Azure-Setup (Woche 1)
- [ ] Azure Subscription und Resource Group
- [ ] Azure Container Registry erstellen
- [ ] Azure Key Vault einrichten
- [ ] Azure DevOps Organization

### Phase 2: Service Principal & Auth (Woche 2)
- [ ] Service Principal für ACR erstellen
- [ ] Service Principal für Key Vault erstellen
- [ ] Kubernetes RBAC konfigurieren
- [ ] Service Connections in Azure DevOps

### Phase 3: Pipeline-Entwicklung (Woche 3)
- [ ] Build Pipeline erstellen
- [ ] Release Pipeline konfigurieren
- [ ] Variable Groups einrichten
- [ ] Environment Approvals

### Phase 4: ACR Integration (Woche 4)
- [ ] Images zu ACR migrieren
- [ ] Vulnerability Scanning aktivieren
- [ ] Retention Policies konfigurieren
- [ ] Kubernetes auf ACR umstellen

### Phase 5: Production-Ready (Woche 5)
- [ ] Security Hardening
- [ ] Monitoring und Alerting
- [ ] Documentation
- [ ] Team Training

### Erweiterte Features (Optional)
- [ ] **Blue-Green Deployments** - Zero-downtime Updates
- [ ] **Canary Releases** - Gradual Rollouts
- [ ] **Multi-Region Deployments** - Geo-replication
- [ ] **Cost Optimization** - Resource Right-sizing
- [ ] **Compliance Automation** - Security und Compliance Checks

---

## 📚 Weitere Ressourcen

### Azure-Dokumentation
- [Azure DevOps Documentation](https://docs.microsoft.com/en-us/azure/devops/)
- [Azure Container Registry Documentation](https://docs.microsoft.com/en-us/azure/container-registry/)
- [Azure Key Vault Documentation](https://docs.microsoft.com/en-us/azure/key-vault/)
- [Azure Pipelines Documentation](https://docs.microsoft.com/en-us/azure/devops/pipelines/)

### Community & Support
- [Azure DevOps Community](https://developercommunity.visualstudio.com/spaces/21/index.html)
- [Azure Container Registry Community](https://github.com/Azure/acr)
- [Microsoft Learn](https://docs.microsoft.com/en-us/learn/)
- [Azure Friday](https://channel9.msdn.com/Shows/Azure-Friday)

### Tools & Extensions
- [Azure DevOps Marketplace](https://marketplace.visualstudio.com/azuredevops)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/)
- [Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/)
- [Azure Resource Manager Templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/)

### Best Practices & Patterns
- [Azure Well-Architected Framework](https://docs.microsoft.com/en-us/azure/architecture/framework/)
- [Azure DevOps Best Practices](https://docs.microsoft.com/en-us/azure/devops/user-guide/project-management-tool-guidance-best-practices)
- [Container Security Best Practices](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-security-best-practices)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/)

---

*Dieses Dokument dient als umfassender Leitfaden für die Implementierung einer Enterprise-CI/CD-Pipeline mit Azure DevOps und Azure Container Registry für lokale Kubernetes Cluster. Alle Platzhalter (IPs, Namen, Credentials) sollten durch eure spezifischen Azure-Werte ersetzt werden.*
