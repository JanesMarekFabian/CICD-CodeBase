# CI/CD Migration Variables Questionnaire

## 🎯 Zweck

Dieses Fragebogen-System sammelt alle benötigten Variablen für die CI/CD-Migration. Der Cursor Agent wird diese Fragen interaktiv stellen und die Antworten für die Code-Generierung verwenden.

## 📋 Fragebogen-Kategorien

### Kategorie 1: Projekt-Grundlagen

#### 1.1 Programmiersprache und Framework
**Frage:** Welche Programmiersprache verwendet euer Projekt?
- [ ] Node.js
- [ ] Python
- [ ] .NET Core
- [ ] Java
- [ ] Go
- [ ] PHP
- [ ] Andere: ___________

**Frage:** Welches Framework wird verwendet?
- [ ] Express.js (Node.js)
- [ ] React (Frontend)
- [ ] Flask (Python)
- [ ] Django (Python)
- [ ] ASP.NET Core (.NET)
- [ ] Spring Boot (Java)
- [ ] Andere: ___________

**Frage:** Welcher Typ von Anwendung ist es?
- [ ] Backend API
- [ ] Frontend SPA
- [ ] Full-Stack Application
- [ ] Microservice
- [ ] Static Website
- [ ] Andere: ___________

#### 1.2 Port und Netzwerk
**Frage:** Auf welchem Port läuft die Anwendung?
- [ ] 3000 (Standard Node.js)
- [ ] 5000 (Standard Flask)
- [ ] 8080 (Standard Java/Go)
- [ ] 80 (HTTP)
- [ ] 443 (HTTPS)
- [ ] Andere: ___________

**Frage:** Benötigt die Anwendung externe Zugriffe?
- [ ] Ja, über HTTP/HTTPS
- [ ] Ja, über spezifischen Port
- [ ] Nein, nur intern
- [ ] Unsicher

#### 1.3 Dependencies und Build
**Frage:** Welcher Package Manager wird verwendet?
- [ ] npm (Node.js)
- [ ] pip (Python)
- [ ] nuget (.NET)
- [ ] maven (Java)
- [ ] gradle (Java)
- [ ] composer (PHP)
- [ ] Andere: ___________

**Frage:** Welche Build-Befehle werden verwendet?
- [ ] `npm run build`
- [ ] `npm start`
- [ ] `python -m flask run`
- [ ] `dotnet build`
- [ ] `mvn clean package`
- [ ] Andere: ___________

**Frage:** Gibt es Tests?
- [ ] Ja, Unit Tests
- [ ] Ja, Integration Tests
- [ ] Ja, E2E Tests
- [ ] Nein, keine Tests
- [ ] Tests geplant

**Frage:** Welche Test-Befehle werden verwendet?
- [ ] `npm test`
- [ ] `pytest`
- [ ] `dotnet test`
- [ ] `mvn test`
- [ ] Andere: ___________

### Kategorie 2: Infrastruktur und Deployment

#### 2.1 Kubernetes Cluster
**Frage:** Läuft bereits ein Kubernetes Cluster?
- [ ] Ja, k3s
- [ ] Ja, k8s
- [ ] Ja, AKS (Azure)
- [ ] Ja, EKS (AWS)
- [ ] Ja, GKE (Google)
- [ ] Nein, muss erstellt werden

**Frage:** Master Node IP-Adresse?
- [ ] 192.168.2.48 (Standard)
- [ ] 192.168.1.101 (Alt)
- [ ] Andere: ___________

**Frage:** Worker Node IP-Adressen?
- [ ] 192.168.2.22 (Standard)
- [ ] Mehrere Nodes: ___________
- [ ] Nur Master Node

**Frage:** Ist kubeconfig bereits konfiguriert?
- [ ] Ja, funktioniert
- [ ] Ja, aber muss aktualisiert werden
- [ ] Nein, muss konfiguriert werden
- [ ] Unsicher

#### 2.2 Container Registry
**Frage:** Welche Container Registry soll verwendet werden?
- [ ] Docker Hub (kostenlos, public)
- [ ] Docker Hub Pro (kostenpflichtig, private)
- [ ] Azure Container Registry (ACR) ⭐ **EMPFOHLEN**
- [ ] Amazon ECR
- [ ] Google Container Registry
- [ ] Andere: ___________

**Frage:** Registry Name (falls ACR)?
- [ ] myregistry.azurecr.io
- [ ] Andere: ___________

**Frage:** Registry Username?
- [ ] Docker Hub Username: ___________
- [ ] ACR Username: ___________
- [ ] Andere: ___________

**Frage:** Registry Authentication?
- [ ] Username/Password
- [ ] Access Token
- [ ] Service Principal (ACR) ⭐ **EMPFOHLEN**
- [ ] Andere: ___________

#### 2.2.1 Azure Key Vault (Empfohlen)
**Frage:** Soll Azure Key Vault für Secrets Management verwendet werden?
- [ ] Ja, Azure Key Vault ⭐ **EMPFOHLEN**
- [ ] Nein, GitHub Secrets
- [ ] Nein, Kubernetes Secrets
- [ ] Unsicher

**Frage:** Azure Key Vault Name?
- [ ] myappkeyvault
- [ ] Andere: ___________

**Frage:** Azure Subscription verfügbar?
- [ ] Ja, Azure Account vorhanden
- [ ] Nein, muss erstellt werden
- [ ] Unsicher

#### 2.3 Namespace und Konfiguration
**Frage:** Gewünschter Kubernetes Namespace?
- [ ] my-app
- [ ] production
- [ ] development
- [ ] Andere: ___________

**Frage:** Wie viele Pod Replicas?
- [ ] 1 (Single Instance)
- [ ] 2 (Standard)
- [ ] 3 (High Availability)
- [ ] Mehr: ___________

**Frage:** Environment (Umgebung)?
- [ ] Development
- [ ] Staging
- [ ] Production
- [ ] Multi-Environment

### Kategorie 3: Sicherheit und Secrets

#### 3.1 Secrets Management
**Frage:** Welche Secrets werden benötigt?
- [ ] Datenbank-Passwort
- [ ] API Keys
- [ ] Session Secret
- [ ] OAuth Credentials
- [ ] SSL Zertifikate
- [ ] Andere: ___________

**Frage:** Wie sollen Secrets verwaltet werden?
- [ ] GitHub Secrets (einfach)
- [ ] Azure Key Vault (sicher) ⭐ **EMPFOHLEN**
- [ ] Kubernetes Secrets (standard)
- [ ] Externe Secret Management
- [ ] Andere: ___________

**Frage:** Soll automatische .env Synchronisation für lokale Entwicklung verwendet werden?
- [ ] Ja, mit Azure Key Vault ⭐ **EMPFOHLEN**
- [ ] Ja, mit anderem System
- [ ] Nein, manuelle Verwaltung
- [ ] Unsicher

#### 3.2 Sicherheitsanforderungen
**Frage:** Sollen Images privat gespeichert werden?
- [ ] Ja, private Registry erforderlich
- [ ] Nein, public Registry ist OK
- [ ] Teilweise, je nach Environment

**Frage:** Soll Vulnerability Scanning durchgeführt werden?
- [ ] Ja, automatisch bei jedem Build
- [ ] Ja, manuell vor Production
- [ ] Nein, nicht erforderlich
- [ ] Unsicher

**Frage:** Sollen Network Policies verwendet werden?
- [ ] Ja, strenge Netzwerk-Isolation
- [ ] Ja, grundlegende Policies
- [ ] Nein, Standard-Netzwerk
- [ ] Unsicher

### Kategorie 4: CI/CD-Strategie

#### 4.1 Budget und Kosten
**Frage:** Wie hoch ist das monatliche Budget?
- [ ] $0 (kostenlos)
- [ ] $5-10 (günstig)
- [ ] $10-50 (mittel)
- [ ] $50+ (unbegrenzt)

**Frage:** Team-Größe?
- [ ] 1-2 Entwickler
- [ ] 3-5 Entwickler
- [ ] 6-10 Entwickler
- [ ] 10+ Entwickler

#### 4.2 CI/CD Engine
**Frage:** Welche CI/CD Engine bevorzugt ihr?
- [ ] GitHub Actions (einfach, kostenlos)
- [ ] Azure DevOps (Enterprise)
- [ ] GitLab CI
- [ ] Jenkins
- [ ] Andere: ___________

**Frage:** Habt ihr bereits Erfahrung mit CI/CD?
- [ ] Ja, viel Erfahrung
- [ ] Ja, grundlegende Erfahrung
- [ ] Nein, erste CI/CD Pipeline
- [ ] Unsicher

#### 4.3 Deployment-Strategie
**Frage:** Welche Deployment-Strategie bevorzugt ihr?
- [ ] Rolling Updates (Standard)
- [ ] Blue-Green Deployments
- [ ] Canary Releases
- [ ] Recreate (Downtime OK)
- [ ] Unsicher

**Frage:** Wie schnell sollen Deployments sein?
- [ ] Sofort (automatisch bei Push)
- [ ] Nach Tests (automatisch nach erfolgreichen Tests)
- [ ] Manuell (nur auf Anfrage)
- [ ] Geplant (zu bestimmten Zeiten)

### Kategorie 5: Monitoring und Observability

#### 5.1 Logging
**Frage:** Wie sollen Logs verwaltet werden?
- [ ] Kubernetes Standard Logs
- [ ] Zentralisiertes Logging (ELK, Loki)
- [ ] Cloud Logging (Azure Monitor, CloudWatch)
- [ ] Keine besonderen Anforderungen

**Frage:** Log-Level?
- [ ] Debug (detailliert)
- [ ] Info (standard)
- [ ] Warning (nur Probleme)
- [ ] Error (nur Fehler)

#### 5.2 Monitoring
**Frage:** Welche Metriken sollen überwacht werden?
- [ ] Pod Status
- [ ] CPU/Memory Usage
- [ ] Application Performance
- [ ] Business Metrics
- [ ] Keine besonderen Anforderungen

**Frage:** Sollen Alerts konfiguriert werden?
- [ ] Ja, bei Pod Failures
- [ ] Ja, bei hoher Ressourcen-Nutzung
- [ ] Ja, bei Application Errors
- [ ] Nein, manuelles Monitoring
- [ ] Unsicher

### Kategorie 6: Environment-spezifische Konfiguration

#### 6.1 Database
**Frage:** Wird eine Datenbank verwendet?
- [ ] Ja, PostgreSQL
- [ ] Ja, MySQL
- [ ] Ja, MongoDB
- [ ] Ja, SQLite
- [ ] Ja, Andere: ___________
- [ ] Nein, keine Datenbank

**Frage:** Wo läuft die Datenbank?
- [ ] Lokal im Kubernetes Cluster
- [ ] Externe Cloud-Datenbank
- [ ] Lokale Installation
- [ ] Docker Container
- [ ] Andere: ___________

#### 6.2 External Services
**Frage:** Welche externen Services werden verwendet?
- [ ] APIs (REST, GraphQL)
- [ ] Message Queues (Redis, RabbitMQ)
- [ ] File Storage (S3, Azure Blob)
- [ ] Email Services
- [ ] Payment Services
- [ ] Andere: ___________

**Frage:** Wie werden externe Services konfiguriert?
- [ ] Environment Variables
- [ ] Config Files
- [ ] Service Discovery
- [ ] Hardcoded URLs
- [ ] Andere: ___________

## 🔄 Interaktiver Ablauf

### Schritt 1: Automatische Analyse
Der Agent analysiert euer Projekt automatisch:
- [ ] Programmiersprache erkennen
- [ ] Framework identifizieren
- [ ] Dependencies analysieren
- [ ] Build-Scripts finden
- [ ] Port-Konfiguration erkennen

### Schritt 2: Interaktive Fragen
Der Agent stellt die relevanten Fragen basierend auf der Analyse:
- [ ] Bestätigung der erkannten Informationen
- [ ] Spezifische Konfigurationsfragen
- [ ] Infrastruktur-spezifische Fragen
- [ ] Sicherheits- und Budget-Fragen

### Schritt 3: Validierung
Der Agent validiert alle gesammelten Informationen:
- [ ] Vollständigkeit prüfen
- [ ] Konsistenz überprüfen
- [ ] Konflikte identifizieren
- [ ] Empfehlungen geben

### Schritt 4: Bestätigung
Der Agent zeigt eine Zusammenfassung:
- [ ] Alle gesammelten Variablen anzeigen
- [ ] Template-Auswahl bestätigen
- [ ] Kosten-Übersicht zeigen
- [ ] Nächste Schritte erklären

## 📊 Variablen-Template

Nach der Befragung erstellt der Agent eine Variablen-Datei:

```yaml
# .env.template
# Projekt-Konfiguration
APP_NAME=my-app
APP_VERSION=1.0.0
APP_PORT=3000
APP_LANGUAGE=nodejs
APP_FRAMEWORK=express

# Kubernetes-Konfiguration
K8S_MASTER_IP=192.168.2.48
K8S_WORKER_IP=192.168.2.22
K8S_NAMESPACE=my-app
K8S_REPLICAS=2

# Registry-Konfiguration
REGISTRY_TYPE=acr
REGISTRY_NAME=myregistry.azurecr.io
REGISTRY_USERNAME=myregistry
REGISTRY_PASSWORD=***

# CI/CD-Konfiguration
CI_ENGINE=github-actions
CD_STRATEGY=rolling-update
DEPLOYMENT_TRIGGER=auto

# Sicherheits-Konfiguration
SECRETS_MANAGEMENT=github-secrets
VULNERABILITY_SCANNING=true
PRIVATE_REGISTRY=true

# Monitoring-Konfiguration
LOGGING_LEVEL=info
MONITORING_ENABLED=true
ALERTS_ENABLED=true
```

## 🎯 Agent-Anweisungen

### Für den Cursor Agent:

1. **Projekt analysieren:** Lies die Projekt-Dateien und erkenne automatisch die Grundkonfiguration
2. **Fragen stellen:** Stelle nur die relevanten Fragen basierend auf der Analyse
3. **Validieren:** Überprüfe alle Antworten auf Konsistenz und Vollständigkeit
4. **Empfehlen:** Gib Empfehlungen basierend auf den Antworten
5. **Bestätigen:** Zeige eine Zusammenfassung vor der Code-Generierung

### Beispiel-Interaktion:

```
Agent: Ich habe euer Projekt analysiert. Es ist eine Node.js Express API auf Port 3000. 
       Stimmt das?

User: Ja, genau.

Agent: Perfekt! Für euer Budget von $5/Monat empfehle ich GitHub Actions + Azure ACR.
       Soll ich mit dieser Konfiguration fortfahren?

User: Ja, das klingt gut.

Agent: Ausgezeichnet! Ich sammle jetzt die letzten Informationen:
       - Kubernetes Master IP: 192.168.2.48?
       - Registry Name: myregistry.azurecr.io?
       - Namespace: my-app?
```

---

*Dieses Fragebogen-System ist darauf ausgelegt, mit dem Cursor Agent interaktiv durchgeführt zu werden. Der Agent wird die Fragen intelligent stellen und basierend auf euren Antworten die beste Konfiguration empfehlen.*
