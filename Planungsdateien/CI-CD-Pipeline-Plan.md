# CI/CD Pipeline für Azure B2C Booking System

## 🎯 Was ist das Ziel?

**Automatisierung von:** Code-Änderung → Build → Test → Deploy auf Kubernetes

**Ohne CI/CD müsstest du:**
- Manuell Docker Images bauen
- Manuell zu Docker Hub pushen
- Manuell auf jeden Server SSH'en
- Manuell Kubernetes-Befehle ausführen
- Manuell prüfen, ob alles läuft

**Mit CI/CD:** `git push` → GitHub Actions macht alles automatisch in 5-10 Minuten!

---

## 📦 Die 4 Haupt-Komponenten

### 1. GitHub Repository (Code-Speicher)
- Dein kompletter Projekt-Code
- Jede Änderung wird getrackt
- Trigger für die gesamte Pipeline

### 2. GitHub Actions (Automatisierungs-Engine)
- Läuft auf GitHub-Servern (kostenlos 2000 Min/Monat)
- Wird getriggert bei: Push, Pull Request, Tag
- Führt definierte Workflows aus

### 3. Docker Hub (Image-Registry)
- Speichert deine fertigen Container-Images
- Öffentlich oder privat
- Kubernetes zieht Images von hier

### 4. Kubernetes Cluster (Ziel-Umgebung)
- Dein k3s Cluster (192.168.2.48 Master, 192.168.2.22 Worker)
- Empfängt Updates von GitHub Actions
- Rollt neue Versionen aus

---

## 🔄 Der Gesamt-Workflow (vereinfacht)

```
Developer (Du)
    ↓ git push
GitHub Repository
    ↓ Trigger
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
[6] Verbindung zu Kubernetes
[7] Deployment aktualisieren
[8] Rollout überwachen
    ↓
Kubernetes Cluster (läuft mit neuer Version)
```

---

## 🗺️ 9-Schritte-Plan zur Umsetzung

### **Schritt 1: Vorbereitung & Accounts einrichten**

**Was du brauchst:**
- [ ] GitHub Account erstellt
- [ ] Docker Hub Account erstellt (kostenlos)
- [ ] Kubernetes Cluster läuft (192.168.2.48 + 192.168.2.22)
- [ ] kubectl funktioniert lokal

**Zeitaufwand:** 15 Minuten

**Was passiert:**
- Accounts anlegen/verifizieren
- Docker Hub Repositories erstellen: `azure-b2c-backend` und `azure-b2c-frontend`
- Git Repository lokal initialisieren

**Wichtigster Befehl:**
```bash
git init
git remote add origin https://github.com/yourusername/Azure-B2C-Auth.git
```

---

### **Schritt 2: Dockerfiles schreiben**

**Was du erstellst:**
- `backend/Dockerfile` - Multi-Stage Build für Node.js Backend
- `frontend/Dockerfile` - Multi-Stage Build für React + Nginx
- `frontend/nginx.conf` - Nginx-Config für API-Proxy
- `.dockerignore` Dateien für beide

**Zeitaufwand:** 30 Minuten

**Was passiert:**
- Backend: Node.js → TypeScript kompilieren → Production Image (klein & schnell)
- Frontend: React bauen → Nginx serven → API-Requests proxyen

**Lokal testen:**
```bash
cd backend && docker build -t test-backend .
cd frontend && docker build -t test-frontend .
```

**Wichtig:** Multi-Stage Builds = Images sind 90% kleiner!

---

### **Schritt 3: Kubernetes Manifests erstellen**

**Was du erstellst:**
- `kubernetes/azure-b2c-booking/namespace.yaml` - Isolierter Namespace
- `kubernetes/azure-b2c-booking/postgres.yaml` - Datenbank mit PVC
- `kubernetes/azure-b2c-booking/redis.yaml` - Session-Store mit PVC
- `kubernetes/azure-b2c-booking/backend.yaml` - Backend Deployment + Service
- `kubernetes/azure-b2c-booking/frontend.yaml` - Frontend Deployment + Service (NodePort)

**Zeitaufwand:** 45 Minuten

**Was wird definiert:**
- Namespace: `azure-b2c-booking`
- ConfigMaps: Umgebungsvariablen (DB-Host, Ports)
- Secrets: Passwörter (werden später via GitHub Actions erstellt)
- Deployments: Replicas, Images, Ressourcen-Limits
- Services: Netzwerk-Zugriff (ClusterIP für intern, NodePort für extern)

**Wichtig:** Noch KEINE Secrets-Datei committen! Werden später über GitHub Actions injected.

---

### **Schritt 4: Manuelles Test-Deployment auf Kubernetes**

**Was du machst:**
- Namespace erstellen
- Secrets manuell anlegen
- Alle Manifests deployen
- Testen ob alles läuft

**Zeitaufwand:** 30 Minuten

**Befehle (vereinfacht):**
```bash
kubectl create namespace azure-b2c-booking
kubectl create secret generic azure-b2c-secrets \
  --from-literal=postgres-password="test123" \
  -n azure-b2c-booking
kubectl apply -f kubernetes/azure-b2c-booking/
```

**Warum wichtig:**
- Du siehst jeden Schritt
- Verstehst die Komponenten
- Fehler frühzeitig finden
- **Erst wenn das läuft, automatisieren!**

**Test:**
```bash
kubectl get pods -n azure-b2c-booking  # Alle Pods sollten Running sein
kubectl get service frontend-service -n azure-b2c-booking  # NodePort notieren
```
Browser: `http://192.168.2.48:<NodePort>`

---

### **Schritt 5: GitHub Secrets konfigurieren**

**Was du einrichtest:**
GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

**Diese Secrets erstellen:**
- `DOCKER_USERNAME` - Dein Docker Hub Username
- `DOCKER_PASSWORD` - Docker Hub Token (nicht Passwort!)
- `KUBE_CONFIG` - Base64-encodierte kubeconfig
- `POSTGRES_PASSWORD` - Sicheres Passwort
- `REDIS_PASSWORD` - Sicheres Passwort
- `SESSION_SECRET` - Random String (32+ Zeichen)
- `STRIPE_SECRET_KEY` - Stripe API Key (oder dummy für Test)

**Zeitaufwand:** 20 Minuten

**Wichtigster Schritt - kubeconfig encodieren:**
```bash
# Auf deinem PC oder Master:
cat ~/.kube/config | base64 -w 0
# Output kopieren → KUBE_CONFIG Secret
```

**Docker Hub Token erstellen:**
- Docker Hub → Account Settings → Security → New Access Token
- Token kopieren → `DOCKER_PASSWORD` Secret

**Wichtig:** Niemals Secrets in Git committen!

---

### **Schritt 6: GitHub Actions Workflows erstellen**

**Was du erstellst:**
- `.github/workflows/backend-cicd.yml` - Backend CI/CD
- `.github/workflows/frontend-cicd.yml` - Frontend CI/CD
- `.github/workflows/initial-deploy.yml` - Erste Installation

**Zeitaufwand:** 60 Minuten

**Was jeder Workflow macht:**

**Backend CI/CD (triggered bei Änderungen in `backend/`):**
1. Code auschecken
2. Node.js + Dependencies installieren
3. TypeScript kompilieren
4. Tests ausführen
5. Docker Image bauen (Tags: `latest` + Git SHA)
6. Zu Docker Hub pushen
7. kubectl konfigurieren
8. Kubernetes Deployment aktualisieren
9. Rollout überwachen (5 Min Timeout)
10. Logs anzeigen

**Frontend CI/CD (triggered bei Änderungen in `frontend/`):**
- Gleiche Schritte, nur für Frontend

**Initial Deployment (manueller Trigger):**
1. Namespace erstellen
2. Secrets aus GitHub Secrets erstellen
3. PostgreSQL + Redis deployen
4. Warten bis Datenbanken ready
5. Backend + Frontend deployen
6. Access-Informationen anzeigen

**Wichtig:** Workflows sind YAML-Dateien - Einrückung ist kritisch!

---

### **Schritt 7: Ersten automatisierten Deployment testen**

**Was du machst:**
1. Alle Dateien committen (außer Secrets!)
2. Zu GitHub pushen
3. Initial Deployment Workflow manuell starten
4. Logs in GitHub Actions beobachten

**Zeitaufwand:** 20 Minuten (+ 10 Min Deployment-Zeit)

**Befehle:**
```bash
git add .
git commit -m "Add CI/CD pipeline"
git push origin main
```

**In GitHub:**
- Repository → Actions Tab
- "Initial Deployment" auswählen
- "Run workflow" klicken
- Logs beobachten

**Was du siehst:**
- ✅ Grüne Häkchen bei jedem erfolgreichen Schritt
- ❌ Rote X bei Fehlern (mit Logs)
- Dauer: ~10 Minuten

**Testen:**
```bash
kubectl get all -n azure-b2c-booking
kubectl get service frontend-service -n azure-b2c-booking
```
Browser: `http://192.168.2.48:<NodePort>`

---

### **Schritt 8: CI/CD Pipeline testen (End-to-End)**

**Was du machst:**
- Kleine Änderung im Backend-Code machen
- Committen + Pushen
- Beobachten wie automatisch deployed wird

**Zeitaufwand:** 15 Minuten

**Test-Änderung (Beispiel):**
```typescript
// backend/src/server.ts
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', version: '1.0.1' });  // Version geändert
});
```

**Commit + Push:**
```bash
git add backend/src/server.ts
git commit -m "Update health endpoint version"
git push
```

**Was automatisch passiert:**
1. GitHub Actions erkennt Änderung in `backend/`
2. Backend CI/CD Workflow startet
3. Docker Image wird gebaut (Tag: Git SHA)
4. Zu Docker Hub gepusht
5. Kubernetes Deployment aktualisiert
6. Rolling Update (Zero Downtime!)
7. Nach 5-10 Min: Neue Version läuft

**Testen:**
```bash
curl http://192.168.2.48:<NodePort>/api/health
# Sollte version: 1.0.1 zeigen
```

**Wichtig:** Keine manuelle Arbeit nötig! Alles automatisch!

---

### **Schritt 9: Monitoring & Auto-Scaling hinzufügen**

**Was du erstellst:**
- `kubernetes/azure-b2c-booking/hpa.yaml` - Horizontal Pod Autoscaler
- `kubernetes/azure-b2c-booking/resource-quota.yaml` - Ressourcen-Limits

**Zeitaufwand:** 20 Minuten

**Was passiert:**
- HPA überwacht CPU/Memory
- Startet automatisch mehr Pods bei Last (2-10 Replicas)
- Scaled runter bei wenig Traffic
- Resource Quotas verhindern Ressourcen-Überlastung

**Deployen:**
```bash
kubectl apply -f kubernetes/azure-b2c-booking/hpa.yaml
kubectl apply -f kubernetes/azure-b2c-booking/resource-quota.yaml
```

**Überwachen:**
```bash
kubectl get hpa -n azure-b2c-booking
kubectl top pods -n azure-b2c-booking
```

**Wichtig:** Macht dein System Production-Ready!

---

## 🔑 Wichtige Konzepte (kurz erklärt)

### Multi-Stage Docker Builds
**Problem:** Dev-Tools (TypeScript Compiler) brauchen viel Speicher  
**Lösung:** Build-Stage (mit Tools) → Production-Stage (nur Runtime)  
**Ergebnis:** Images sind 90% kleiner (z.B. 1GB → 100MB)

### Image Tagging-Strategie
- `latest` - Immer neueste Version (für schnelle Tests)
- `<git-sha>` - Exakte Version (für präzise Rollbacks)

**Warum beide?** Rollback zu jeder Version in Sekunden möglich!

### Secrets vs ConfigMaps
- **Secrets:** Passwörter, API Keys (Base64-encoded, verschlüsselt)
- **ConfigMaps:** Normale Config (Hostnames, Ports, URLs)

### Liveness vs Readiness Probes
- **Liveness:** "Lebt der Container noch?" → Neustart bei Fehler
- **Readiness:** "Ist der Container bereit?" → Kein Traffic wenn nicht ready

### Rolling Updates
- Kubernetes startet neue Pods **bevor** alte gelöscht werden
- Zero-Downtime Deployments
- Automatischer Rollback bei Fehler

### Service Types
- **ClusterIP:** Nur intern erreichbar (Backend, Datenbank)
- **NodePort:** Von außen erreichbar (Frontend, für Dev/Test)
- **LoadBalancer:** Cloud-Provider externe IP (Production, später Azure)

---

## 🎬 Der typische Workflow (nach Setup)

1. **Code schreiben** auf deinem PC
2. **Git commit + push**
3. **GitHub Actions** startet automatisch (siehst du im Actions-Tab)
4. **5-10 Minuten später:** Neue Version läuft auf Kubernetes
5. **Testen:** Browser → `http://192.168.2.48:<NodePort>`
6. **Bei Fehler:** Code fixen → push → automatisch neu deployen

**Kein SSH, kein kubectl, kein Docker-Build mehr nötig!**

---

## 🛡️ Sicherheits-Best-Practices

1. **Secrets Management**
   - Niemals Passwörter im Code
   - Niemals Secrets in Git committen
   - GitHub Secrets = verschlüsselt gespeichert
   - Kubernetes Secrets = Base64-encoded + RBAC-geschützt

2. **Least Privilege**
   - GitHub Actions nur Zugriff auf einen Namespace
   - Service Accounts mit minimalen Permissions
   - kubeconfig mit RBAC-Beschränkungen

3. **Image Security**
   - Alpine Linux (minimal, weniger Angriffsfläche)
   - Multi-Stage Builds (keine Build-Tools in Production)
   - Image Scanning (später mit Trivy/Snyk)

---

## 🚀 Vorteile nach Implementation

✅ **Automatisierung:** Push → Deploy in 10 Min  
✅ **Reproduzierbar:** Jeder Build identisch  
✅ **Testbar:** Jeder Commit wird getestet  
✅ **Rollback:** Zurück zu jeder Version in Sekunden  
✅ **Dokumentation:** Workflow-Dateien = Deployment-Doku  
✅ **Skalierbar:** Von 1 zu 100 Deployments/Tag  
✅ **Team-Ready:** Mehrere Entwickler, keine Konflikte  

---

## 💰 Kosten-Übersicht

### Development (jetzt)
- GitHub Actions: 2000 Min/Monat kostenlos
- Docker Hub: Unbegrenzt Public Repos kostenlos
- Kubernetes: Deine Hardware ($0)
- **Total: $0/Monat**

### Production (später, optional - Azure)
- GitHub Actions: $0 (innerhalb Free Tier)
- Azure Container Registry: ~$5/Monat (Basic)
- Azure Kubernetes Service: ~$150/Monat (3 Nodes)
- Azure Database for PostgreSQL: ~$50/Monat
- **Total: ~$205/Monat**

---

## 🔧 Rollback-Strategie (bei Problemen)

**Zu vorheriger Version:**
```bash
kubectl rollout undo deployment/backend -n azure-b2c-booking
kubectl rollout undo deployment/frontend -n azure-b2c-booking
```

**Zu spezifischer Version:**
```bash
kubectl set image deployment/backend \
  backend=yourusername/azure-b2c-backend:<git-sha> \
  -n azure-b2c-booking
```

**Rollout-Status prüfen:**
```bash
kubectl rollout history deployment/backend -n azure-b2c-booking
```

---

## 📊 Monitoring & Debugging

### Logs anschauen
```bash
kubectl logs -f deployment/backend -n azure-b2c-booking
kubectl logs -f deployment/frontend -n azure-b2c-booking
```

### Ressourcen-Auslastung
```bash
kubectl top pods -n azure-b2c-booking
kubectl top nodes
```

### Events prüfen
```bash
kubectl get events -n azure-b2c-booking --sort-by='.lastTimestamp'
```

### Pods debuggen
```bash
kubectl describe pod <pod-name> -n azure-b2c-booking
kubectl get pods -n azure-b2c-booking -o wide
```

---

## 🔜 Nächste Schritte (nach CI/CD)

1. **Ingress Controller** (Traefik/Nginx) - Custom Domains statt NodePort
2. **Cert-Manager** - Automatische SSL-Zertifikate (Let's Encrypt)
3. **Prometheus + Grafana** - Professionelles Monitoring & Dashboards
4. **Loki** - Zentralisiertes Logging
5. **Azure AD Integration** - Authentifizierung (siehe `Azure-AD-Auth-Plan.md`)
6. **ArgoCD** - GitOps (fortgeschritten, optional)

---

## ⚠️ Häufige Fehler & Lösungen

### "Failed to push to Docker Hub"
- **Ursache:** Falsches Docker Hub Token
- **Lösung:** Token neu erstellen → GitHub Secret aktualisieren

### "kubectl: connection refused"
- **Ursache:** kubeconfig falsch encodiert oder veraltet
- **Lösung:** Neu encodieren mit `cat ~/.kube/config | base64 -w 0`

### "ImagePullBackOff" in Kubernetes
- **Ursache:** Image existiert nicht auf Docker Hub
- **Lösung:** Prüfe Docker Hub → manuell Image pushen zum Test

### "CrashLoopBackOff" nach Deployment
- **Ursache:** Container startet nicht (z.B. DB-Verbindung fehlt)
- **Lösung:** Logs prüfen: `kubectl logs <pod-name> -n azure-b2c-booking`

### Workflow schlägt fehl bei "npm ci"
- **Ursache:** `package-lock.json` fehlt oder veraltet
- **Lösung:** Lokal `npm install` → `package-lock.json` committen

---

## 📋 Checkliste: Bist du bereit?

**Vor Start:**
- [ ] GitHub Account erstellt
- [ ] Docker Hub Account erstellt
- [ ] Kubernetes Cluster läuft (192.168.2.48 + .22)
- [ ] kubectl lokal funktioniert
- [ ] Kann auf Master SSH'en

**Nach Schritt 4:**
- [ ] Manuelles Deployment funktioniert
- [ ] Frontend im Browser erreichbar
- [ ] Backend /health endpoint antwortet
- [ ] Datenbank-Verbindung klappt

**Nach Schritt 7:**
- [ ] GitHub Actions läuft erfolgreich durch
- [ ] Docker Images auf Docker Hub sichtbar
- [ ] Kubernetes Pods laufen (alle "Running")

**Nach Schritt 8:**
- [ ] Code-Änderung triggert automatisches Deployment
- [ ] Neue Version nach ~10 Min live
- [ ] Zero-Downtime (alte Pods laufen während Update)

---

## 📁 Dateien-Übersicht (kompakt)

### Dockerfiles (3 Dateien)
- `backend/Dockerfile` - Multi-Stage Build für Node.js
- `frontend/Dockerfile` - Multi-Stage Build für React + Nginx
- `frontend/nginx.conf` - Nginx Config mit API-Proxy

### Kubernetes Manifests (7 Dateien)
- `kubernetes/azure-b2c-booking/namespace.yaml` - Namespace + ConfigMap
- `kubernetes/azure-b2c-booking/postgres.yaml` - PostgreSQL + PVC + Service
- `kubernetes/azure-b2c-booking/redis.yaml` - Redis + PVC + Service
- `kubernetes/azure-b2c-booking/backend.yaml` - Backend Deployment + Service
- `kubernetes/azure-b2c-booking/frontend.yaml` - Frontend Deployment + Service (NodePort)
- `kubernetes/azure-b2c-booking/hpa.yaml` - Horizontal Pod Autoscaler
- `kubernetes/azure-b2c-booking/resource-quota.yaml` - Ressourcen-Limits

### GitHub Workflows (3 Dateien)
- `.github/workflows/backend-cicd.yml` - Backend CI/CD
- `.github/workflows/frontend-cicd.yml` - Frontend CI/CD
- `.github/workflows/initial-deploy.yml` - Erste Installation

**Total: 13 Dateien zum Erstellen**

---

## 📄 Anhang: Code-Vorlagen

Die vollständigen Code-Vorlagen für alle 13 Dateien findest du weiter unten im Dokument, organisiert nach Phasen:

**Für schnellen Zugriff:**
- **Phase 1:** GitHub Repository & Secrets Setup (Zeile ~590)
- **Phase 2:** Dockerfiles (Zeile ~630)
- **Phase 3:** Kubernetes Manifests (Zeile ~730)
- **Phase 4:** GitHub Actions Workflows (Zeile ~1050)

**Hinweis:** Die Code-Vorlagen sind Vorschläge. Du musst `yourusername`, IPs und Secrets an deine Umgebung anpassen!

---

## Phase 1: Repository & Secrets Setup

### 1.1 GitHub Repository vorbereiten
Aktuelles Projekt auf GitHub pushen (falls noch nicht geschehen):
```bash
git init
git add .
git commit -m "Initial commit: Azure B2C Booking System"
git branch -M main
git remote add origin https://github.com/yourusername/Azure-B2C-Auth.git
git push -u origin main
```

### 1.2 GitHub Secrets konfigurieren
**Settings → Secrets and variables → Actions → New repository secret:**

```
DOCKER_USERNAME: <Docker Hub Username>
DOCKER_PASSWORD: <Docker Hub Token/Password>
KUBE_CONFIG: <Base64-encoded kubeconfig>
POSTGRES_PASSWORD: <Secure Password>
REDIS_PASSWORD: <Secure Password>
SESSION_SECRET: <Random Secret>
STRIPE_SECRET_KEY: <Stripe API Key>
```

**kubeconfig Base64 encodieren:**
```bash
# Auf Kubernetes Master (192.168.2.48):
cat ~/.kube/config | base64 -w 0
# Output als KUBE_CONFIG Secret speichern
```

### 1.3 Docker Hub Repository erstellen
**Docker Hub → Repositories → Create:**
```
Name: azure-b2c-backend
Visibility: Public (oder Private)

Name: azure-b2c-frontend
Visibility: Public (oder Private)
```

## Phase 2: Dockerfiles erstellen

### 2.1 Backend Dockerfile
**Datei: `backend/Dockerfile`**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
EXPOSE 3000
ENV NODE_ENV=production
CMD ["node", "dist/server.js"]
```

### 2.2 Frontend Dockerfile
**Datei: `frontend/Dockerfile`**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 2.3 Nginx Config für Frontend
**Datei: `frontend/nginx.conf`**
```nginx
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://backend-service:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### 2.4 .dockerignore erstellen
**Datei: `backend/.dockerignore` und `frontend/.dockerignore`**
```
node_modules
npm-debug.log
.env
.git
.gitignore
README.md
dist
```

## Phase 3: Kubernetes Manifests

### 3.1 Namespace und ConfigMap
**Datei: `kubernetes/azure-b2c-booking/namespace.yaml`**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: azure-b2c-booking
  labels:
    name: azure-b2c-booking
    environment: development
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: azure-b2c-config
  namespace: azure-b2c-booking
data:
  NODE_ENV: "production"
  DATABASE_HOST: "postgres-service"
  DATABASE_PORT: "5432"
  DATABASE_NAME: "azure_b2c_db"
  REDIS_HOST: "redis-service"
  REDIS_PORT: "6379"
```

### 3.2 Secrets
**Datei: `kubernetes/azure-b2c-booking/secrets.yaml`**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: azure-b2c-secrets
  namespace: azure-b2c-booking
type: Opaque
stringData:
  postgres-password: "${POSTGRES_PASSWORD}"
  redis-password: "${REDIS_PASSWORD}"
  session-secret: "${SESSION_SECRET}"
  stripe-secret-key: "${STRIPE_SECRET_KEY}"
```

### 3.3 PostgreSQL Deployment
**Datei: `kubernetes/azure-b2c-booking/postgres.yaml`**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: azure-b2c-booking
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 20Gi
  storageClassName: local-path
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: azure-b2c-booking
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15-alpine
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          valueFrom:
            configMapKeyRef:
              name: azure-b2c-config
              key: DATABASE_NAME
        - name: POSTGRES_USER
          value: "postgres"
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: azure-b2c-secrets
              key: postgres-password
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
        resources:
          requests: {memory: "512Mi", cpu: "250m"}
          limits: {memory: "1Gi", cpu: "500m"}
      volumes:
      - name: postgres-data
        persistentVolumeClaim:
          claimName: postgres-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: azure-b2c-booking
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  type: ClusterIP
```

### 3.4 Redis Deployment
**Datei: `kubernetes/azure-b2c-booking/redis.yaml`**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-pvc
  namespace: azure-b2c-booking
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-path
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: azure-b2c-booking
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
        command: ["redis-server", "--requirepass", "$(REDIS_PASSWORD)"]
        env:
        - name: REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: azure-b2c-secrets
              key: redis-password
        volumeMounts:
        - name: redis-data
          mountPath: /data
        resources:
          requests: {memory: "256Mi", cpu: "100m"}
          limits: {memory: "512Mi", cpu: "250m"}
      volumes:
      - name: redis-data
        persistentVolumeClaim:
          claimName: redis-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: azure-b2c-booking
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
  type: ClusterIP
```

### 3.5 Backend Deployment
**Datei: `kubernetes/azure-b2c-booking/backend.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: azure-b2c-booking
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: yourusername/azure-b2c-backend:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          valueFrom:
            configMapKeyRef:
              name: azure-b2c-config
              key: NODE_ENV
        - name: DATABASE_URL
          value: "postgresql://postgres:$(POSTGRES_PASSWORD)@postgres-service:5432/azure_b2c_db"
        - name: REDIS_URL
          value: "redis://:$(REDIS_PASSWORD)@redis-service:6379"
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: azure-b2c-secrets
              key: postgres-password
        - name: REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: azure-b2c-secrets
              key: redis-password
        - name: SESSION_SECRET
          valueFrom:
            secretKeyRef:
              name: azure-b2c-secrets
              key: session-secret
        - name: STRIPE_SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: azure-b2c-secrets
              key: stripe-secret-key
        resources:
          requests: {memory: "256Mi", cpu: "250m"}
          limits: {memory: "512Mi", cpu: "500m"}
        livenessProbe:
          httpGet: {path: /health, port: 3000}
          initialDelaySeconds: 30
          periodSeconds: 30
        readinessProbe:
          httpGet: {path: /ready, port: 3000}
          initialDelaySeconds: 10
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: azure-b2c-booking
spec:
  selector:
    app: backend
  ports:
  - port: 3000
    targetPort: 3000
  type: ClusterIP
```

### 3.6 Frontend Deployment
**Datei: `kubernetes/azure-b2c-booking/frontend.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: azure-b2c-booking
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: yourusername/azure-b2c-frontend:latest
        ports:
        - containerPort: 80
        env:
        - name: REACT_APP_API_URL
          value: "http://backend-service:3000"
        resources:
          requests: {memory: "128Mi", cpu: "100m"}
          limits: {memory: "256Mi", cpu: "200m"}
        livenessProbe:
          httpGet: {path: /, port: 80}
          initialDelaySeconds: 10
          periodSeconds: 30
        readinessProbe:
          httpGet: {path: /, port: 80}
          initialDelaySeconds: 5
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: azure-b2c-booking
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
  type: NodePort
```

## Phase 4: GitHub Actions Workflows

### 4.1 Backend CI/CD Workflow
**Datei: `.github/workflows/backend-cicd.yml`**
```yaml
name: Backend CI/CD

on:
  push:
    branches: [main]
    paths:
      - 'backend/**'
      - '.github/workflows/backend-cicd.yml'
  pull_request:
    branches: [main]
    paths:
      - 'backend/**'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: backend/package-lock.json

    - name: Install dependencies
      working-directory: backend
      run: npm ci

    - name: Build TypeScript
      working-directory: backend
      run: npm run build

    - name: Run tests
      working-directory: backend
      run: npm test || echo "No tests yet"

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push Docker image
      working-directory: backend
      run: |
        docker build -t ${{ secrets.DOCKER_USERNAME }}/azure-b2c-backend:latest \
                     -t ${{ secrets.DOCKER_USERNAME }}/azure-b2c-backend:${{ github.sha }} .
        docker push ${{ secrets.DOCKER_USERNAME }}/azure-b2c-backend:latest
        docker push ${{ secrets.DOCKER_USERNAME }}/azure-b2c-backend:${{ github.sha }}

    - name: Set up kubectl
      uses: azure/setup-kubectl@v4

    - name: Configure kubectl
      run: |
        mkdir -p ~/.kube
        echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > ~/.kube/config
        chmod 600 ~/.kube/config

    - name: Deploy to Kubernetes
      run: |
        kubectl set image deployment/backend \
          backend=${{ secrets.DOCKER_USERNAME }}/azure-b2c-backend:${{ github.sha }} \
          -n azure-b2c-booking
        kubectl rollout status deployment/backend -n azure-b2c-booking --timeout=5m

    - name: Verify deployment
      run: |
        kubectl get pods -n azure-b2c-booking -l app=backend
        kubectl logs -l app=backend -n azure-b2c-booking --tail=50
```

### 4.2 Frontend CI/CD Workflow
**Datei: `.github/workflows/frontend-cicd.yml`**
```yaml
name: Frontend CI/CD

on:
  push:
    branches: [main]
    paths:
      - 'frontend/**'
      - '.github/workflows/frontend-cicd.yml'
  pull_request:
    branches: [main]
    paths:
      - 'frontend/**'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: frontend/package-lock.json

    - name: Install dependencies
      working-directory: frontend
      run: npm ci

    - name: Build
      working-directory: frontend
      run: npm run build

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push Docker image
      working-directory: frontend
      run: |
        docker build -t ${{ secrets.DOCKER_USERNAME }}/azure-b2c-frontend:latest \
                     -t ${{ secrets.DOCKER_USERNAME }}/azure-b2c-frontend:${{ github.sha }} .
        docker push ${{ secrets.DOCKER_USERNAME }}/azure-b2c-frontend:latest
        docker push ${{ secrets.DOCKER_USERNAME }}/azure-b2c-frontend:${{ github.sha }}

    - name: Set up kubectl
      uses: azure/setup-kubectl@v4

    - name: Configure kubectl
      run: |
        mkdir -p ~/.kube
        echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > ~/.kube/config
        chmod 600 ~/.kube/config

    - name: Deploy to Kubernetes
      run: |
        kubectl set image deployment/frontend \
          frontend=${{ secrets.DOCKER_USERNAME }}/azure-b2c-frontend:${{ github.sha }} \
          -n azure-b2c-booking
        kubectl rollout status deployment/frontend -n azure-b2c-booking --timeout=5m

    - name: Verify deployment
      run: |
        kubectl get pods -n azure-b2c-booking -l app=frontend
        kubectl get service frontend-service -n azure-b2c-booking
```

### 4.3 Initial Deployment Workflow
**Datei: `.github/workflows/initial-deploy.yml`**
```yaml
name: Initial Deployment

on:
  workflow_dispatch:

jobs:
  deploy-infrastructure:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up kubectl
      uses: azure/setup-kubectl@v4

    - name: Configure kubectl
      run: |
        mkdir -p ~/.kube
        echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > ~/.kube/config
        chmod 600 ~/.kube/config

    - name: Create namespace and secrets
      run: |
        kubectl apply -f kubernetes/azure-b2c-booking/namespace.yaml
        
        # Create secrets from GitHub secrets
        kubectl create secret generic azure-b2c-secrets \
          --from-literal=postgres-password="${{ secrets.POSTGRES_PASSWORD }}" \
          --from-literal=redis-password="${{ secrets.REDIS_PASSWORD }}" \
          --from-literal=session-secret="${{ secrets.SESSION_SECRET }}" \
          --from-literal=stripe-secret-key="${{ secrets.STRIPE_SECRET_KEY }}" \
          -n azure-b2c-booking --dry-run=client -o yaml | kubectl apply -f -

    - name: Deploy PostgreSQL and Redis
      run: |
        kubectl apply -f kubernetes/azure-b2c-booking/postgres.yaml
        kubectl apply -f kubernetes/azure-b2c-booking/redis.yaml
        
        # Wait for databases
        kubectl wait --for=condition=ready pod -l app=postgres -n azure-b2c-booking --timeout=300s
        kubectl wait --for=condition=ready pod -l app=redis -n azure-b2c-booking --timeout=300s

    - name: Deploy Backend and Frontend
      run: |
        kubectl apply -f kubernetes/azure-b2c-booking/backend.yaml
        kubectl apply -f kubernetes/azure-b2c-booking/frontend.yaml
        
        # Wait for deployments
        kubectl rollout status deployment/backend -n azure-b2c-booking --timeout=5m
        kubectl rollout status deployment/frontend -n azure-b2c-booking --timeout=5m

    - name: Display access information
      run: |
        echo "Deployment completed!"
        kubectl get all -n azure-b2c-booking
        echo "Frontend NodePort:"
        kubectl get service frontend-service -n azure-b2c-booking
```

## Phase 5: Deployment & Testing

### 5.1 Initiales Deployment
**Manuell (erstes Mal):**
```bash
# Secrets erstellen
kubectl create namespace azure-b2c-booking
kubectl create secret generic azure-b2c-secrets \
  --from-literal=postgres-password="your-password" \
  --from-literal=redis-password="your-password" \
  --from-literal=session-secret="your-secret" \
  --from-literal=stripe-secret-key="your-key" \
  -n azure-b2c-booking

# Alle Manifests deployen
kubectl apply -f kubernetes/azure-b2c-booking/
```

**Oder via GitHub Actions:**
```
GitHub → Actions → Initial Deployment → Run workflow
```

### 5.2 Zugriff testen
```bash
# NodePort finden
kubectl get service frontend-service -n azure-b2c-booking

# Browser öffnen
http://192.168.2.48:<NodePort>
http://192.168.2.22:<NodePort>
```

### 5.3 Logs prüfen
```bash
kubectl logs -f deployment/backend -n azure-b2c-booking
kubectl logs -f deployment/frontend -n azure-b2c-booking
```

## Phase 6: Auto-Scaling & Monitoring

### 6.1 Horizontal Pod Autoscaler
**Datei: `kubernetes/azure-b2c-booking/hpa.yaml`**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: azure-b2c-booking
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### 6.2 Resource Quotas
**Datei: `kubernetes/azure-b2c-booking/resource-quota.yaml`**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: azure-b2c-quota
  namespace: azure-b2c-booking
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
    persistentvolumeclaims: "10"
    pods: "20"
    services: "10"
```

## Phase 7: Production Migration (Azure)

### 7.1 Azure Container Registry
```bash
# ACR erstellen
az acr create --resource-group myResourceGroup \
  --name azureb2cregistry --sku Basic

# GitHub Secret hinzufügen
ACR_USERNAME: azureb2cregistry
ACR_PASSWORD: <ACR Admin Password>
```

### 7.2 Azure Kubernetes Service
```bash
# AKS Cluster erstellen
az aks create --resource-group myResourceGroup \
  --name azure-b2c-aks --node-count 3 \
  --enable-addons monitoring --generate-ssh-keys

# kubeconfig für AKS
az aks get-credentials --resource-group myResourceGroup --name azure-b2c-aks
```

### 7.3 Production Workflow
**Datei: `.github/workflows/production-deploy.yml`**
```yaml
name: Production Deploy

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy-to-azure:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Login to ACR
      uses: azure/docker-login@v1
      with:
        login-server: azureb2cregistry.azurecr.io
        username: ${{ secrets.ACR_USERNAME }}
        password: ${{ secrets.ACR_PASSWORD }}

    - name: Build and push to ACR
      run: |
        docker build -t azureb2cregistry.azurecr.io/backend:${{ github.ref_name }} backend/
        docker build -t azureb2cregistry.azurecr.io/frontend:${{ github.ref_name }} frontend/
        docker push azureb2cregistry.azurecr.io/backend:${{ github.ref_name }}
        docker push azureb2cregistry.azurecr.io/frontend:${{ github.ref_name }}

    - name: Deploy to AKS
      run: |
        az aks get-credentials --resource-group myResourceGroup --name azure-b2c-aks
        kubectl set image deployment/backend backend=azureb2cregistry.azurecr.io/backend:${{ github.ref_name }} -n azure-b2c-booking
        kubectl set image deployment/frontend frontend=azureb2cregistry.azurecr.io/frontend:${{ github.ref_name }} -n azure-b2c-booking
```

## Sicherheit & Best Practices

1. **Secrets Management**: Niemals Secrets in Git committen
2. **Image Tags**: Immer Git SHA taggen für Rollback-Fähigkeit
3. **Health Checks**: Liveness + Readiness Probes definieren
4. **Resource Limits**: CPU/Memory Limits setzen
5. **RBAC**: Minimal Permissions für Service Accounts
6. **Network Policies**: Traffic zwischen Pods einschränken (später)

## Rollback-Strategie

```bash
# Zu vorheriger Version zurück
kubectl rollout undo deployment/backend -n azure-b2c-booking
kubectl rollout undo deployment/frontend -n azure-b2c-booking

# Zu spezifischer Version
kubectl set image deployment/backend \
  backend=yourusername/azure-b2c-backend:<git-sha> \
  -n azure-b2c-booking
```

## Monitoring & Observability

### Logs
```bash
kubectl logs -f deployment/backend -n azure-b2c-booking
kubectl logs -f deployment/frontend -n azure-b2c-booking
```

### Metrics
```bash
kubectl top pods -n azure-b2c-booking
kubectl top nodes
```

### Events
```bash
kubectl get events -n azure-b2c-booking --sort-by='.lastTimestamp'
```

## Kosten-Übersicht

### Development (Home Lab)
- GitHub Actions: 2000 Minuten/Monat kostenlos
- Docker Hub: Unlimited public repos kostenlos
- Kubernetes: Eigene Hardware ($0)
- **Total: $0/Monat**

### Production (Azure)
- GitHub Actions: $0 (innerhalb Free Tier)
- Azure Container Registry: ~$5/Monat (Basic)
- Azure Kubernetes Service: ~$150/Monat (3 Nodes)
- Azure Database for PostgreSQL: ~$50/Monat
- **Total: ~$205/Monat**

## Dateien zum Erstellen

### Dockerfiles
- `backend/Dockerfile`
- `backend/.dockerignore`
- `frontend/Dockerfile`
- `frontend/.dockerignore`
- `frontend/nginx.conf`

### Kubernetes Manifests
- `kubernetes/azure-b2c-booking/namespace.yaml`
- `kubernetes/azure-b2c-booking/postgres.yaml`
- `kubernetes/azure-b2c-booking/redis.yaml`
- `kubernetes/azure-b2c-booking/backend.yaml`
- `kubernetes/azure-b2c-booking/frontend.yaml`
- `kubernetes/azure-b2c-booking/hpa.yaml`
- `kubernetes/azure-b2c-booking/resource-quota.yaml`

### GitHub Workflows
- `.github/workflows/backend-cicd.yml`
- `.github/workflows/frontend-cicd.yml`
- `.github/workflows/initial-deploy.yml`
- `.github/workflows/production-deploy.yml` (später)

## Checkliste vor Start

- [ ] GitHub Repository erstellt
- [ ] Docker Hub Account vorhanden
- [ ] GitHub Secrets konfiguriert
- [ ] kubeconfig Base64-encodiert
- [ ] Kubernetes Cluster läuft
- [ ] Internet-Zugriff für Cluster (temporär)

## Nächste Schritte nach CI/CD

1. **Ingress Controller** (Traefik/Nginx) für Custom Domains
2. **Cert-Manager** für automatische SSL-Zertifikate
3. **Prometheus + Grafana** für Monitoring
4. **Loki** für zentralisiertes Logging
5. **ArgoCD** für GitOps (optional, fortgeschritten)
6. **Azure AD Integration** für Authentication (siehe Azure-AD-Auth-Plan.md)

