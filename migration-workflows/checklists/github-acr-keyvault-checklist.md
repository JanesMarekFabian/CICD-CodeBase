# GitHub Actions + Azure ACR + Key Vault Checklist

## 🎯 Übersicht

Diese Checkliste führt durch die Migration zu einer CI/CD-Pipeline mit GitHub Actions, Azure Container Registry und Azure Key Vault.

## 📋 Voraussetzungen

### Azure Account
- [ ] Azure Account mit aktiver Subscription
- [ ] Azure CLI installiert und konfiguriert
- [ ] Resource Group erstellt

### GitHub Repository
- [ ] GitHub Repository erstellt
- [ ] Code committed und gepusht
- [ ] GitHub Actions aktiviert

### Kubernetes Cluster
- [ ] Kubernetes Cluster läuft
- [ ] kubectl installiert und konfiguriert
- [ ] kubeconfig verfügbar

## 🔧 Setup Phase

### Azure Services
- [ ] Azure Container Registry (ACR) erstellt
- [ ] Azure Key Vault erstellt
- [ ] Service Principal für GitHub Actions erstellt
- [ ] RBAC-Berechtigungen konfiguriert

### Secrets in Key Vault
- [ ] Database Password gespeichert
- [ ] Redis Password gespeichert
- [ ] Session Secret gespeichert
- [ ] API Keys gespeichert
- [ ] OAuth Secrets gespeichert

### GitHub Secrets
- [ ] ACR_REGISTRY gesetzt
- [ ] ACR_USERNAME gesetzt
- [ ] ACR_PASSWORD gesetzt
- [ ] AZURE_KEY_VAULT_NAME gesetzt
- [ ] AZURE_CLIENT_ID gesetzt
- [ ] AZURE_CLIENT_SECRET gesetzt
- [ ] AZURE_TENANT_ID gesetzt
- [ ] AZURE_SUBSCRIPTION_ID gesetzt
- [ ] KUBERNETES_MASTER_IP gesetzt
- [ ] KUBECONFIG gesetzt

## 🐳 Docker & Kubernetes

### Dockerfiles
- [ ] Backend Dockerfile erstellt
- [ ] Frontend Dockerfile erstellt
- [ ] Multi-Stage Builds konfiguriert
- [ ] Health Checks implementiert
- [ ] Security Best Practices angewendet

### Kubernetes Manifests
- [ ] Namespace erstellt
- [ ] ConfigMap erstellt
- [ ] Secrets Template erstellt
- [ ] Backend Deployment erstellt
- [ ] Frontend Deployment erstellt
- [ ] Services erstellt
- [ ] Ingress erstellt (optional)

## 🚀 CI/CD Pipeline

### GitHub Actions
- [ ] CI/CD Workflow erstellt
- [ ] PR Check Workflow erstellt
- [ ] Azure Login konfiguriert
- [ ] Key Vault Integration implementiert
- [ ] Docker Build & Push konfiguriert
- [ ] Kubernetes Deployment konfiguriert
- [ ] Security Scanning aktiviert

### Lokale Entwicklung
- [ ] .env Synchronisation Script erstellt
- [ ] Lokale .env Dateien synchronisiert
- [ ] Development Workflow getestet

## 🧪 Testing

### Lokale Tests
- [ ] Docker Builds getestet
- [ ] Health Checks validiert
- [ ] Environment Variables getestet
- [ ] Key Vault Access getestet

### CI/CD Tests
- [ ] Erster Push durchgeführt
- [ ] Build-Prozess überwacht
- [ ] Registry Push überprüft
- [ ] Kubernetes Deployment überwacht
- [ ] Key Vault Integration getestet

### End-to-End Tests
- [ ] Anwendung im Browser getestet
- [ ] API Endpoints getestet
- [ ] Logs überprüft
- [ ] Metrics überwacht
- [ ] Rollback getestet

## 📊 Monitoring & Security

### Monitoring
- [ ] Pod Status überwacht
- [ ] Resource Usage überwacht
- [ ] Health Checks funktionieren
- [ ] Logs zentralisiert

### Security
- [ ] Private Registry verwendet
- [ ] Secrets in Key Vault gespeichert
- [ ] Vulnerability Scanning aktiv
- [ ] RBAC konfiguriert
- [ ] Audit Trail aktiviert

## 🎯 Erfolgs-Kriterien

Nach erfolgreicher Migration:
- [ ] Automatische Builds bei jedem Push
- [ ] Automatische Tests bei jedem Build
- [ ] Automatische Deployments auf Kubernetes
- [ ] Zero-Downtime Updates
- [ ] Enterprise-Grade Secrets Management
- [ ] Automatische .env Synchronisation
- [ ] Audit Trail für alle Secret-Zugriffe
- [ ] RBAC für Secrets
- [ ] Secret Rotation möglich
- [ ] Compliance mit Enterprise-Standards

## 💰 Kosten

- [ ] Azure ACR: $5/Monat
- [ ] Azure Key Vault: ~$0,01-0,10/Monat
- [ ] Gesamt: ~$5,01-5,10/Monat

## 🚨 Troubleshooting

### Häufige Probleme
- [ ] Build schlägt fehl: Dependencies prüfen
- [ ] Deployment schlägt fehl: Kubernetes Cluster erreichbar?
- [ ] Secrets funktionieren nicht: Key Vault Berechtigungen prüfen
- [ ] .env Sync schlägt fehl: Azure CLI Login prüfen

### Nützliche Commands
```bash
# Key Vault Secrets auflisten
az keyvault secret list --vault-name myappkeyvault

# .env synchronisieren
./scripts/sync-env-from-keyvault.sh

# Kubernetes Status prüfen
kubectl get pods -n my-app

# GitHub Actions Logs prüfen
# Repository → Actions → Workflow Run → Logs
```

## 🎉 Abschluss

- [ ] Alle Checklisten-Punkte abgehakt
- [ ] Anwendung läuft erfolgreich
- [ ] CI/CD Pipeline funktioniert
- [ ] Team über neuen Workflow informiert
- [ ] Dokumentation erstellt
- [ ] Nächste Schritte geplant

---

**🎯 Herzlichen Glückwunsch! Du hast jetzt eine Enterprise-Grade CI/CD-Pipeline mit Azure Key Vault!**
