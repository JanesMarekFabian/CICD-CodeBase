# Azure Key Vault Setup Prompt

## 🎯 Zweck

Dieser Prompt hilft dem Cursor Agent dabei, Azure Key Vault als Standard-Secrets-Management-System zu integrieren.

## 📋 Prompt für den Agent

```
Du bist ein Experte für Azure Key Vault Integration in CI/CD-Pipelines. 

Erstelle eine vollständige Azure Key Vault Integration für das Projekt:

1. **Azure Key Vault Setup:**
   - Key Vault erstellen mit Standard-Konfiguration
   - Service Principal für GitHub Actions erstellen
   - RBAC-Berechtigungen konfigurieren
   - Alle Application Secrets in Key Vault speichern

2. **GitHub Actions Integration:**
   - Azure Login mit Service Principal
   - Key Vault Secrets automatisch abrufen
   - Kubernetes Secrets aus Key Vault erstellen
   - Sichere Secret-Verwaltung

3. **Lokale Entwicklung:**
   - .env Synchronisation Script erstellen
   - Automatische Synchronisation mit Key Vault
   - Nahtlose Entwicklungsumgebung

4. **Dokumentation:**
   - Setup-Anleitung für Key Vault
   - Secrets Template mit Key Vault
   - Troubleshooting Guide

**Wichtige Punkte:**
- Azure Key Vault als Single Source of Truth
- Automatische .env Synchronisation
- Enterprise-Grade Security
- Audit Trail und RBAC
- Kosteneffizient (~$0,01-0,10/Monat)

Erstelle alle notwendigen Dateien und Konfigurationen für eine vollständige Key Vault Integration.
```

## 🔧 Verwendung

Der Agent sollte diesen Prompt verwenden, wenn:
- Budget > $5/Monat
- Enterprise-Grade Security gewünscht
- Automatische .env Synchronisation benötigt
- Audit Trail und RBAC erforderlich

## 📊 Erwartete Ausgabe

Der Agent sollte erstellen:
- Azure Key Vault Setup-Scripts
- GitHub Actions mit Key Vault Integration
- .env Synchronisation Scripts
- Vollständige Dokumentation
- Troubleshooting Guides
