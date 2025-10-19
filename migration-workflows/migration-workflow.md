# CI/CD Migration Workflow

## 🎯 Übersicht

Dieser Workflow führt euch Schritt für Schritt durch die Migration eines bestehenden Projekts zu einer vollständigen CI/CD-Pipeline. Der Cursor Agent wird euch dabei unterstützen, alle notwendigen Komponenten zu erstellen und zu konfigurieren.

## 📋 Workflow-Phasen

### Phase 1: Projekt-Analyse und Vorbereitung
**Dauer:** 10-15 Minuten

#### Schritt 1.1: Projekt-Übersicht
Der Agent analysiert euer Projekt und stellt folgende Fragen:

**Projekt-Grundlagen:**
- [ ] **Programmiersprache:** Welche Sprache verwendet das Projekt? (Node.js, Python, .NET, Java, etc.)
- [ ] **Framework:** Welches Framework wird verwendet? (Express, Flask, ASP.NET, Spring, etc.)
- [ ] **Projekt-Typ:** Backend API, Frontend SPA, Full-Stack, Microservice?
- [ ] **Port:** Auf welchem Port läuft die Anwendung? (Standard-Ports: 3000, 5000, 8080, etc.)

**Dependencies und Build:**
- [ ] **Package Manager:** npm, pip, nuget, maven, gradle?
- [ ] **Build-Scripts:** Welche Build-Befehle werden verwendet?
- [ ] **Test-Scripts:** Gibt es Tests? Welche Test-Frameworks?
- [ ] **Environment Variables:** Welche Umgebungsvariablen werden benötigt?

#### Schritt 1.2: Infrastruktur-Analyse
- [ ] **Kubernetes Cluster:** Läuft bereits ein Cluster? (IP-Adressen notieren)
- [ ] **Container Registry:** Welche Registry soll verwendet werden?
- [ ] **Secrets Management:** Wie werden Secrets aktuell verwaltet? (Empfehlung: Azure Key Vault)
- [ ] **Monitoring:** Gibt es bereits Monitoring-Lösungen?
- [ ] **Azure Account:** Ist bereits ein Azure Account vorhanden?

### Phase 2: CI/CD-Strategie Auswahl
**Dauer:** 5-10 Minuten

#### Schritt 2.1: Anforderungs-Analyse
Der Agent fragt nach euren Anforderungen:

**Kosten:**
- [ ] **Budget:** Wie hoch ist das monatliche Budget? ($0, $5-20, $20+)
- [ ] **Team-Größe:** Wie viele Entwickler arbeiten am Projekt?

**Sicherheit:**
- [ ] **Private Registry:** Sollen Images privat gespeichert werden?
- [ ] **Vulnerability Scanning:** Sollen Sicherheitsprüfungen durchgeführt werden?
- [ ] **Secrets Management:** Wie sicher sollen Secrets verwaltet werden? (Empfehlung: Azure Key Vault)
- [ ] **Enterprise-Grade Security:** Sollen Audit Trail und RBAC verwendet werden?

**Skalierbarkeit:**
- [ ] **Zukünftige Projekte:** Werden weitere Projekte migriert?
- [ ] **Team-Wachstum:** Wird das Team wachsen?
- [ ] **Enterprise-Features:** Werden erweiterte Features benötigt?

#### Schritt 2.2: Template-Auswahl
Basierend auf euren Antworten wählt der Agent das passende Template:

**Option A: GitHub Actions + Docker Hub**
- ✅ **Kosten:** $0/Monat
- ✅ **Einfachheit:** Sehr einfach zu setup
- ❌ **Sicherheit:** Public Images, begrenzte Rate Limits
- **Empfehlung:** Für kleine, Open Source Projekte

**Option B: GitHub Actions + Azure ACR + Key Vault** ⭐
- ✅ **Kosten:** $5,01-5,10/Monat
- ✅ **Sicherheit:** Private Registry, Vulnerability Scanning, Enterprise-Grade Secrets Management
- ✅ **Einfachheit:** GitHub Actions ist einfach zu lernen
- ✅ **Enterprise-Features:** Azure Key Vault, Audit Trail, RBAC
- **Empfehlung:** Für die meisten Projekte (BESTE WAHL)

**Option C: Azure DevOps + Azure ACR**
- ✅ **Sicherheit:** Enterprise-Features, Advanced Security
- ✅ **Skalierbarkeit:** Unbegrenzte Build-Minuten
- ❌ **Kosten:** $6/User/Monat + $5/Monat
- **Empfehlung:** Für Enterprise-Umgebungen

### Phase 3: Variablen-Sammlung
**Dauer:** 10-15 Minuten

#### Schritt 3.1: Infrastruktur-Variablen
Der Agent sammelt alle benötigten Infrastruktur-Informationen:

**Kubernetes Cluster:**
- [ ] **Master IP:** IP-Adresse des Kubernetes Master (z.B. 192.168.2.48)
- [ ] **Worker IPs:** IP-Adressen der Worker Nodes (z.B. 192.168.2.22)
- [ ] **Namespace:** Gewünschter Namespace-Name (z.B. my-app)
- [ ] **kubeconfig:** Ist kubeconfig bereits konfiguriert?

**Container Registry:**
- [ ] **Registry Name:** Name der Registry (z.B. myregistry.azurecr.io)
- [ ] **Registry Username:** Username für Registry-Zugriff
- [ ] **Registry Password/Token:** Passwort oder Token
- [ ] **Image Name:** Name für das Docker Image (z.B. my-app)

**Azure Key Vault (Empfohlen):**
- [ ] **Key Vault Name:** Name der Key Vault (z.B. myappkeyvault)
- [ ] **Service Principal:** Client ID, Client Secret, Tenant ID
- [ ] **Subscription ID:** Azure Subscription ID
- [ ] **Resource Group:** Azure Resource Group Name

#### Schritt 3.2: Anwendungs-Variablen
- [ ] **App Name:** Name der Anwendung
- [ ] **App Version:** Aktuelle Version
- [ ] **Environment:** Development, Staging, Production?
- [ ] **Replicas:** Wie viele Pods sollen laufen? (Standard: 2)

#### Schritt 3.3: Secrets und Konfiguration
- [ ] **Database URL:** Falls vorhanden
- [ ] **API Keys:** Externe API-Schlüssel
- [ ] **Session Secret:** Für Session-Management
- [ ] **Custom Environment Variables:** Projekt-spezifische Variablen

**Azure Key Vault Secrets (Empfohlen):**
- [ ] **Database Password:** Für sichere Speicherung
- [ ] **Redis Password:** Für Cache-Sicherheit
- [ ] **API Keys:** Für externe Services
- [ ] **OAuth Secrets:** Für Authentication
- [ ] **SSL Certificates:** Für HTTPS

### Phase 4: Code-Generierung
**Dauer:** 15-20 Minuten

#### Schritt 4.1: Dockerfile-Generierung
Der Agent erstellt ein optimiertes Dockerfile:

**Multi-Stage Build:**
- [ ] **Build Stage:** Dependencies installieren und Code kompilieren
- [ ] **Production Stage:** Nur Runtime und kompilierter Code
- [ ] **Health Checks:** Liveness und Readiness Probes
- [ ] **Security:** Non-root User, minimale Base Images

#### Schritt 4.2: CI/CD Workflow-Generierung
Der Agent erstellt GitHub Actions Workflows:

**Build Workflow:**
- [ ] **Trigger:** Push, Pull Request
- [ ] **Steps:** Checkout → Setup → Build → Test
- [ ] **Artifacts:** Build-Ergebnisse speichern

**Deploy Workflow:**
- [ ] **Trigger:** Push auf main branch
- [ ] **Steps:** Build → Test → Docker → Push → Deploy
- [ ] **Rolling Updates:** Zero-downtime Deployments

**Azure Key Vault Integration:**
- [ ] **Azure Login:** Service Principal Authentication
- [ ] **Secret Retrieval:** Automatisches Abrufen aus Key Vault
- [ ] **Kubernetes Secrets:** Automatische Erstellung aus Key Vault

#### Schritt 4.3: Kubernetes Manifests-Generierung
Der Agent erstellt alle benötigten Kubernetes-Dateien:

**Namespace und Config:**
- [ ] **namespace.yaml:** Namespace-Definition
- [ ] **configmap.yaml:** Environment-spezifische Konfiguration
- [ ] **secrets.yaml:** Template für Secrets (aus Key Vault)

**Azure Key Vault Integration:**
- [ ] **Key Vault Secrets:** Automatische Synchronisation
- [ ] **Service Principal:** Für Key Vault Zugriff
- [ ] **RBAC:** Berechtigungen für GitHub Actions

**Deployment und Services:**
- [ ] **deployment.yaml:** Pod-Definition mit Image und Ressourcen
- [ ] **service.yaml:** Service-Definition für Netzwerk-Zugriff
- [ ] **ingress.yaml:** Optional für externe Zugriffe

### Phase 5: Setup und Konfiguration
**Dauer:** 20-30 Minuten

#### Schritt 5.1: Azure Services Setup
**Für Azure ACR:**
- [ ] **ACR erstellen:** Azure Container Registry anlegen
- [ ] **Service Principal:** Für GitHub Actions erstellen
- [ ] **Permissions:** ACR Push/Pull Rechte vergeben
- [ ] **GitHub Secrets:** Credentials in GitHub speichern

**Für Azure Key Vault (Empfohlen):**
- [ ] **Key Vault erstellen:** Azure Key Vault anlegen
- [ ] **Service Principal:** Für Key Vault Zugriff erstellen
- [ ] **RBAC:** Berechtigungen für GitHub Actions konfigurieren
- [ ] **Secrets speichern:** Alle Application Secrets in Key Vault
- [ ] **GitHub Secrets:** Service Principal Credentials in GitHub

**Für Docker Hub:**
- [ ] **Account:** Docker Hub Account erstellen/konfigurieren
- [ ] **Repository:** Private Repository anlegen
- [ ] **Access Token:** Token für GitHub Actions erstellen
- [ ] **GitHub Secrets:** Token in GitHub speichern

#### Schritt 5.2: Kubernetes-Setup
- [ ] **kubeconfig:** Für GitHub Actions konfigurieren
- [ ] **Namespace:** In Kubernetes erstellen
- [ ] **Secrets:** Application Secrets erstellen
- [ ] **RBAC:** Service Account für GitHub Actions

#### Schritt 5.3: GitHub-Setup
- [ ] **Repository:** GitHub Repository vorbereiten
- [ ] **Secrets:** Alle Secrets konfigurieren (inkl. Key Vault Credentials)
- [ ] **Workflows:** Workflow-Dateien committen
- [ ] **Branch Protection:** Main Branch schützen

#### Schritt 5.4: Lokale Entwicklung Setup
**Azure Key Vault Integration:**
- [ ] **Sync Script:** .env Synchronisation Script erstellen
- [ ] **Lokale .env:** Automatische Synchronisation mit Key Vault
- [ ] **Development Workflow:** Nahtlose lokale Entwicklung

### Phase 6: Testing und Validierung
**Dauer:** 15-20 Minuten

#### Schritt 6.1: Lokales Testing
- [ ] **Dockerfile:** Lokal bauen und testen
- [ ] **Image:** Image lokal starten und testen
- [ ] **Health Checks:** Health Endpoints testen
- [ ] **Environment Variables:** Konfiguration validieren

**Azure Key Vault Testing:**
- [ ] **Key Vault Access:** Secrets aus Key Vault abrufen
- [ ] **Local .env Sync:** Automatische Synchronisation testen
- [ ] **Secret Rotation:** Secret-Updates testen

#### Schritt 6.2: CI/CD Testing
- [ ] **Workflow:** Ersten Push durchführen
- [ ] **Build:** Build-Prozess überwachen
- [ ] **Registry:** Image Push überprüfen
- [ ] **Deployment:** Kubernetes Deployment überwachen

**Azure Key Vault CI/CD Testing:**
- [ ] **Secret Retrieval:** GitHub Actions holt Secrets aus Key Vault
- [ ] **Kubernetes Secrets:** Automatische Erstellung aus Key Vault
- [ ] **Application Runtime:** Secrets werden korrekt geladen

#### Schritt 6.3: End-to-End Testing
- [ ] **Application:** Anwendung im Browser/API testen
- [ ] **Logs:** Kubernetes Logs überprüfen
- [ ] **Metrics:** Pod Status und Ressourcen überwachen
- [ ] **Rollback:** Rollback-Funktionalität testen

### Phase 7: Dokumentation und Training
**Dauer:** 10-15 Minuten

#### Schritt 7.1: Dokumentation
- [ ] **README:** Deployment-Anleitung erstellen
- [ ] **Troubleshooting:** Häufige Probleme dokumentieren
- [ ] **Rollback:** Rollback-Prozess dokumentieren
- [ ] **Monitoring:** Monitoring-Anleitung erstellen

#### Schritt 7.2: Team-Training
- [ ] **Workflow:** Team über neuen Workflow informieren
- [ ] **Secrets:** Secrets-Management erklären
- [ ] **Deployment:** Deployment-Prozess demonstrieren
- [ ] **Troubleshooting:** Debugging-Techniken zeigen

## 🔄 Workflow-Entscheidungsbaum

```
Projekt-Analyse
    ↓
Budget < $10/Monat?
    ├─ Ja → GitHub Actions + Docker Hub
    └─ Nein → Sicherheitsanforderungen?
        ├─ Hoch → Azure DevOps + ACR + Key Vault
        └─ Mittel → GitHub Actions + ACR + Key Vault (EMPFOHLEN)
```

## 🎯 **Neue Standard-Empfehlung: Azure Key Vault Integration**

**Für alle Projekte mit Budget > $5/Monat:**
- ✅ **GitHub Actions + Azure ACR + Azure Key Vault**
- ✅ **Enterprise-Grade Secrets Management**
- ✅ **Automatische .env Synchronisation**
- ✅ **Audit Trail und RBAC**
- ✅ **Kosten:** Nur $5,01-5,10/Monat

## 📊 Erfolgs-Metriken

Nach erfolgreicher Migration solltet ihr haben:

- [ ] **Automatische Builds:** Jeder Push triggert Build
- [ ] **Automatische Tests:** Tests laufen bei jedem Build
- [ ] **Automatische Deployments:** Erfolgreiche Builds werden deployed
- [ ] **Zero-Downtime:** Rolling Updates ohne Ausfallzeit
- [ ] **Rollback-Fähigkeit:** Schneller Rollback bei Problemen
- [ ] **Monitoring:** Überwachung von Pods und Services
- [ ] **Security:** Private Images und sichere Secrets

**Mit Azure Key Vault Integration:**
- [ ] **Enterprise-Grade Secrets Management:** Alle Secrets in Azure Key Vault
- [ ] **Automatische .env Synchronisation:** Lokale Entwicklung mit Key Vault
- [ ] **Audit Trail:** Vollständige Nachverfolgung aller Secret-Zugriffe
- [ ] **RBAC:** Feingranulare Berechtigungen für Secrets
- [ ] **Secret Rotation:** Automatische Rotation von Secrets
- [ ] **Compliance:** Erfüllt Enterprise-Sicherheitsstandards

## 🚨 Troubleshooting

### Häufige Probleme und Lösungen

**Build schlägt fehl:**
- [ ] Dependencies prüfen
- [ ] Build-Scripts validieren
- [ ] Environment Variables überprüfen

**Deployment schlägt fehl:**
- [ ] Kubernetes Cluster erreichbar?
- [ ] Image in Registry vorhanden?
- [ ] Secrets korrekt konfiguriert?

**Application läuft nicht:**
- [ ] Health Checks funktionieren?
- [ ] Port-Konfiguration korrekt?
- [ ] Environment Variables gesetzt?

## 📚 Nächste Schritte

Nach erfolgreicher Migration:

1. **Monitoring erweitern:** Prometheus, Grafana
2. **Logging zentralisieren:** ELK Stack, Loki
3. **Security härten:** Network Policies, Pod Security
4. **Performance optimieren:** Resource Limits, HPA
5. **Backup-Strategie:** Datenbank-Backups, Config-Backups

## 🎯 Checkliste für Agent

Der Cursor Agent sollte folgende Checkliste abarbeiten:

### Vor der Migration:
- [ ] Projekt analysiert und verstanden
- [ ] Alle benötigten Variablen gesammelt
- [ ] Passendes Template ausgewählt
- [ ] Benutzer über Kosten und Komplexität informiert

### Während der Migration:
- [ ] Schritt für Schritt durch Workflow führen
- [ ] Bei jedem Schritt Bestätigung einholen
- [ ] Code-Generierung erklären
- [ ] Fehler sofort beheben

### Nach der Migration:
- [ ] Vollständige Funktionalität testen
- [ ] Dokumentation erstellen
- [ ] Team schulen
- [ ] Nächste Schritte planen

---

*Dieser Workflow ist darauf ausgelegt, mit dem Cursor Agent interaktiv durchgeführt zu werden. Der Agent wird euch durch jeden Schritt führen und bei Problemen helfen.*
