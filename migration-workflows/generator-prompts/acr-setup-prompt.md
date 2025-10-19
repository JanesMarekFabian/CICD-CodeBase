# Azure Container Registry Setup Prompt

## 🎯 Zweck

Dieser Prompt hilft dem Cursor Agent dabei, eine vollständige Anleitung für das Setup von Azure Container Registry (ACR) zu erstellen. Der Agent soll Schritt-für-Schritt Anweisungen für ACR-Setup und GitHub Actions Integration bereitstellen.

## 📋 Prompt für den Agent

### Kontext
Du bist ein Experte für Azure Container Registry und Azure DevOps. Du sollst eine vollständige Setup-Anleitung für Azure Container Registry erstellen, die mit GitHub Actions integriert wird. Das Projekt wurde analysiert und die folgenden Informationen sind verfügbar:

**Projekt-Informationen:**
- App Name: `{APP_NAME}`
- Azure Subscription: `{AZURE_SUBSCRIPTION}`
- Resource Group: `{RESOURCE_GROUP}`
- ACR Name: `{ACR_NAME}`
- Location: `{AZURE_LOCATION}`

### Aufgabe
Erstelle eine vollständige ACR-Setup-Anleitung mit folgenden Anforderungen:

#### 1. Azure-Setup
- **Resource Group:** Erstellen oder verwenden bestehende
- **ACR erstellen:** Azure Container Registry anlegen
- **ACR konfigurieren:** Admin User, Vulnerability Scanning, Retention Policies

#### 2. Service Principal Setup
- **Service Principal erstellen:** Für GitHub Actions Integration
- **Permissions vergeben:** ACR Push/Pull Rechte
- **Credentials sammeln:** Client ID, Client Secret, Tenant ID

#### 3. GitHub Actions Integration
- **GitHub Secrets konfigurieren:** ACR-Credentials in GitHub speichern
- **Workflow anpassen:** ACR-Login in GitHub Actions
- **Image Push testen:** Ersten Push zu ACR durchführen

#### 4. Kubernetes Integration
- **ImagePullSecrets erstellen:** Für Kubernetes ACR-Zugriff
- **Service Principal für K8s:** Kubernetes ACR-Zugriff konfigurieren
- **Deployment testen:** Image aus ACR in Kubernetes deployen

### Output-Format
Erstelle folgende Dateien:

1. **acr-setup-guide.md** - Vollständige Setup-Anleitung
2. **acr-commands.md** - Azure CLI Befehle
3. **github-secrets.md** - GitHub Secrets Konfiguration
4. **k8s-acr-integration.md** - Kubernetes ACR Integration

## 🔧 Setup-Anleitung Template

### Azure Container Registry Setup Guide
```markdown
# Azure Container Registry Setup Guide

## Voraussetzungen
- Azure Subscription aktiv
- Azure CLI installiert und konfiguriert
- GitHub Repository mit Admin-Rechten
- Kubernetes Cluster läuft

## Schritt 1: Azure Resource Group erstellen
```bash
# Resource Group erstellen
az group create \
  --name {RESOURCE_GROUP} \
  --location {AZURE_LOCATION}

# Resource Group anzeigen
az group show --name {RESOURCE_GROUP}
```

## Schritt 2: Azure Container Registry erstellen
```bash
# ACR erstellen (Basic Tier - $5/Monat)
az acr create \
  --resource-group {RESOURCE_GROUP} \
  --name {ACR_NAME} \
  --sku Basic \
  --location {AZURE_LOCATION} \
  --admin-enabled true

# ACR Details anzeigen
az acr show --name {ACR_NAME} --resource-group {RESOURCE_GROUP}
```

## Schritt 3: ACR konfigurieren
```bash
# Vulnerability Scanning aktivieren
az acr update \
  --name {ACR_NAME} \
  --resource-group {RESOURCE_GROUP} \
  --admin-enabled true

# Retention Policy konfigurieren (30 Tage)
az acr config retention update \
  --name {ACR_NAME} \
  --resource-group {RESOURCE_GROUP} \
  --days 30 \
  --status Enabled
```

## Schritt 4: Service Principal erstellen
```bash
# Service Principal für GitHub Actions erstellen
az ad sp create-for-rbac \
  --name "github-actions-{ACR_NAME}" \
  --role acrpush \
  --scopes /subscriptions/{AZURE_SUBSCRIPTION}/resourceGroups/{RESOURCE_GROUP}/providers/Microsoft.ContainerRegistry/registries/{ACR_NAME}

# Output notieren:
# {
#   "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
#   "displayName": "github-actions-{ACR_NAME}",
#   "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
#   "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
# }
```

## Schritt 5: GitHub Secrets konfigurieren
GitHub Repository → Settings → Secrets and variables → Actions → New repository secret:

- `AZURE_CLIENT_ID`: appId aus Service Principal
- `AZURE_CLIENT_SECRET`: password aus Service Principal
- `AZURE_TENANT_ID`: tenant aus Service Principal
- `ACR_NAME`: {ACR_NAME}
- `ACR_LOGIN_SERVER`: {ACR_NAME}.azurecr.io

## Schritt 6: GitHub Actions Workflow anpassen
```yaml
- name: Login to Azure ACR
  uses: azure/docker-login@v1
  with:
    login-server: ${{ secrets.ACR_LOGIN_SERVER }}
    username: ${{ secrets.AZURE_CLIENT_ID }}
    password: ${{ secrets.AZURE_CLIENT_SECRET }}

- name: Build and push Docker image
  run: |
    docker build -t ${{ secrets.ACR_LOGIN_SERVER }}/${{ env.IMAGE_NAME }}:latest \
                 -t ${{ secrets.ACR_LOGIN_SERVER }}/${{ env.IMAGE_NAME }}:${{ github.sha }} .
    docker push ${{ secrets.ACR_LOGIN_SERVER }}/${{ env.IMAGE_NAME }}:latest
    docker push ${{ secrets.ACR_LOGIN_SERVER }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

## Schritt 7: Kubernetes ACR Integration
```bash
# ACR Secret für Kubernetes erstellen
kubectl create secret docker-registry acr-secret \
  --docker-server={ACR_NAME}.azurecr.io \
  --docker-username={AZURE_CLIENT_ID} \
  --docker-password={AZURE_CLIENT_SECRET} \
  --namespace={K8S_NAMESPACE}

# Secret in Deployment verwenden
# imagePullSecrets:
# - name: acr-secret
```

## Schritt 8: Testen
```bash
# Lokal zu ACR pushen
docker login {ACR_NAME}.azurecr.io
docker tag {APP_NAME}:latest {ACR_NAME}.azurecr.io/{APP_NAME}:latest
docker push {ACR_NAME}.azurecr.io/{APP_NAME}:latest

# Image in ACR anzeigen
az acr repository list --name {ACR_NAME}

# Kubernetes Deployment testen
kubectl set image deployment/{APP_NAME} {APP_NAME}={ACR_NAME}.azurecr.io/{APP_NAME}:latest -n {K8S_NAMESPACE}
```
```

## 🔧 Azure CLI Commands Template

### ACR Management Commands
```bash
# ACR erstellen
az acr create \
  --resource-group {RESOURCE_GROUP} \
  --name {ACR_NAME} \
  --sku Basic \
  --location {AZURE_LOCATION} \
  --admin-enabled true

# ACR anzeigen
az acr show --name {ACR_NAME} --resource-group {RESOURCE_GROUP}

# ACR aktualisieren
az acr update \
  --name {ACR_NAME} \
  --resource-group {RESOURCE_GROUP} \
  --admin-enabled true

# ACR löschen
az acr delete --name {ACR_NAME} --resource-group {RESOURCE_GROUP}

# ACR Repository auflisten
az acr repository list --name {ACR_NAME}

# ACR Tags auflisten
az acr repository show-tags --name {ACR_NAME} --repository {APP_NAME}

# ACR Images löschen
az acr repository delete --name {ACR_NAME} --repository {APP_NAME} --tag latest
```

### Service Principal Commands
```bash
# Service Principal erstellen
az ad sp create-for-rbac \
  --name "github-actions-{ACR_NAME}" \
  --role acrpush \
  --scopes /subscriptions/{AZURE_SUBSCRIPTION}/resourceGroups/{RESOURCE_GROUP}/providers/Microsoft.ContainerRegistry/registries/{ACR_NAME}

# Service Principal anzeigen
az ad sp list --display-name "github-actions-{ACR_NAME}"

# Service Principal löschen
az ad sp delete --id {SERVICE_PRINCIPAL_ID}
```

## 🔒 GitHub Secrets Configuration

### Required Secrets
```yaml
# GitHub Repository Secrets
AZURE_CLIENT_ID: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
AZURE_CLIENT_SECRET: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
AZURE_TENANT_ID: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
ACR_NAME: "{ACR_NAME}"
ACR_LOGIN_SERVER: "{ACR_NAME}.azurecr.io"
```

### Secret Setup Instructions
1. **GitHub Repository öffnen**
2. **Settings → Secrets and variables → Actions**
3. **New repository secret** für jeden Secret
4. **Name und Value eingeben**
5. **Add secret** klicken

## 🎯 Agent-Anweisungen

### Für den Cursor Agent:

1. **Analysiere die Anforderungen:** Verstehe Azure-Setup, ACR-Konfiguration und Integration
2. **Erstelle Setup-Anleitung:** Schritt-für-Schritt Anweisungen für ACR-Setup
3. **Azure CLI Befehle:** Alle notwendigen Azure CLI Befehle bereitstellen
4. **GitHub Integration:** GitHub Secrets und Workflow-Konfiguration
5. **Kubernetes Integration:** ACR-Integration in Kubernetes
6. **Test-Anweisungen:** Vollständige Test-Prozedur
7. **Troubleshooting:** Häufige Probleme und Lösungen

### Beispiel-Interaktion:

```
Agent: Ich erstelle eine vollständige ACR-Setup-Anleitung für euer Projekt.
       Ich sehe, ihr habt bereits eine Azure Subscription. Ich erstelle:
       - ACR-Setup mit Azure CLI Befehlen
       - Service Principal für GitHub Actions
       - GitHub Secrets Konfiguration
       - Kubernetes ACR Integration

User: Perfekt! Könntest du auch Vulnerability Scanning hinzufügen?

Agent: Natürlich! Ich füge ACR Vulnerability Scanning Konfiguration hinzu.
       Das scannt eure Images automatisch auf Sicherheitslücken.

User: Das ist großartig!

Agent: Die komplette ACR-Setup-Anleitung ist fertig! Sie enthält alle Schritte
       von der ACR-Erstellung bis zur Kubernetes-Integration.
```

## 🔍 Validierung

Nach der Generierung soll der Agent folgende Punkte überprüfen:

- [ ] **Azure CLI Befehle:** Alle Befehle syntaktisch korrekt
- [ ] **Service Principal:** Korrekte Permissions und Scopes
- [ ] **GitHub Secrets:** Alle notwendigen Secrets aufgelistet
- [ ] **Kubernetes Integration:** ImagePullSecrets korrekt konfiguriert
- [ ] **Test-Prozedur:** Vollständige Test-Anweisungen
- [ ] **Troubleshooting:** Häufige Probleme und Lösungen
- [ ] **Security:** Security Best Practices befolgt

## 📝 Kosten-Übersicht

### ACR Pricing (Stand 2024)
- **Basic:** $5/Monat (10GB Storage)
- **Standard:** $20/Monat (100GB Storage)
- **Premium:** $50/Monat (500GB Storage)

### Zusätzliche Kosten
- **Storage:** $0.10/GB/Monat
- **Bandwidth:** $0.10/GB
- **Vulnerability Scanning:** Inklusive
- **Geo-replication:** Nur Premium

## 🚨 Troubleshooting

### Häufige Probleme

**"Failed to login to ACR"**
- Service Principal Credentials prüfen
- ACR Admin User aktiviert?
- Network-Zugriff zu ACR?

**"Image pull failed in Kubernetes"**
- ImagePullSecrets korrekt konfiguriert?
- Service Principal hat ACR-Zugriff?
- Image existiert in ACR?

**"GitHub Actions ACR login failed"**
- GitHub Secrets korrekt gesetzt?
- Service Principal Permissions?
- ACR Name und Login Server korrekt?

---

*Dieser Prompt ist darauf ausgelegt, mit dem Cursor Agent verwendet zu werden, um eine vollständige Azure Container Registry Setup-Anleitung zu erstellen.*
