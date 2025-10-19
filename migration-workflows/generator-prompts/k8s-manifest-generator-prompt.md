# Kubernetes Manifest Generator Prompt

## 🎯 Zweck

Dieser Prompt hilft dem Cursor Agent dabei, vollständige Kubernetes Manifests für ein bestehendes Projekt zu generieren. Der Agent soll alle notwendigen Kubernetes-Ressourcen erstellen.

## 📋 Prompt für den Agent

### Kontext
Du bist ein Experte für Kubernetes und Container-Orchestrierung. Du sollst Kubernetes Manifests für ein bestehendes Projekt erstellen. Das Projekt wurde analysiert und die folgenden Informationen sind verfügbar:

**Projekt-Informationen:**
- App Name: `{APP_NAME}`
- App Port: `{APP_PORT}`
- Image Name: `{IMAGE_NAME}`
- Registry: `{REGISTRY_NAME}`
- Namespace: `{K8S_NAMESPACE}`
- Replicas: `{REPLICAS}`
- Environment: `{ENVIRONMENT}`

**Infrastruktur-Informationen:**
- Kubernetes Master IP: `{K8S_MASTER_IP}`
- Kubernetes Worker IPs: `{K8S_WORKER_IPS}`
- Registry Type: `{REGISTRY_TYPE}` (dockerhub/acr)
- Secrets Required: `{SECRETS_LIST}`

### Aufgabe
Erstelle vollständige Kubernetes Manifests mit folgenden Anforderungen:

#### 1. Namespace und Konfiguration
- **Namespace:** Isolierte Umgebung für die Anwendung
- **ConfigMap:** Environment-spezifische Konfiguration
- **Secrets:** Template für sensitive Daten

#### 2. Deployment und Services
- **Deployment:** Pod-Definition mit Image und Ressourcen
- **Service:** Netzwerk-Zugriff für die Anwendung
- **Ingress:** Optional für externe Zugriffe

#### 3. Security und Best Practices
- **Resource Limits:** CPU und Memory Limits
- **Health Checks:** Liveness und Readiness Probes
- **Security Context:** Non-root User, Read-only Root Filesystem
- **ImagePullSecrets:** Für private Registry-Zugriffe

### Output-Format
Erstelle folgende Dateien:

1. **k8s/namespace.yaml** - Namespace und ConfigMap
2. **k8s/secrets.yaml** - Secrets Template
3. **k8s/deployment.yaml** - Deployment Definition
4. **k8s/service.yaml** - Service Definition
5. **k8s/ingress.yaml** - Ingress Definition (optional)
6. **k8s/hpa.yaml** - Horizontal Pod Autoscaler (optional)

## 🔧 Manifest-Templates

### Namespace und ConfigMap Template
```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {K8S_NAMESPACE}
  labels:
    name: {K8S_NAMESPACE}
    environment: {ENVIRONMENT}
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: {APP_NAME}-config
  namespace: {K8S_NAMESPACE}
data:
  NODE_ENV: "{ENVIRONMENT}"
  PORT: "{APP_PORT}"
  LOG_LEVEL: "info"
  # Add more environment-specific configuration here
```

### Secrets Template
```yaml
# k8s/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: {APP_NAME}-secrets
  namespace: {K8S_NAMESPACE}
type: Opaque
stringData:
  # Database credentials
  DATABASE_URL: "postgresql://user:password@postgres-service:5432/dbname"
  # API keys
  API_KEY: "your-api-key-here"
  # Session secret
  SESSION_SECRET: "your-session-secret-here"
  # Add more secrets as needed
```

### Deployment Template
```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {APP_NAME}
  namespace: {K8S_NAMESPACE}
  labels:
    app: {APP_NAME}
    environment: {ENVIRONMENT}
spec:
  replicas: {REPLICAS}
  selector:
    matchLabels:
      app: {APP_NAME}
  template:
    metadata:
      labels:
        app: {APP_NAME}
        environment: {ENVIRONMENT}
    spec:
      # Image pull secrets for private registry
      imagePullSecrets:
      - name: {REGISTRY_TYPE}-secret
      
      containers:
      - name: {APP_NAME}
        image: {REGISTRY_NAME}/{IMAGE_NAME}:latest
        ports:
        - containerPort: {APP_PORT}
          name: http
        
        # Environment variables from ConfigMap and Secrets
        env:
        - name: NODE_ENV
          valueFrom:
            configMapKeyRef:
              name: {APP_NAME}-config
              key: NODE_ENV
        - name: PORT
          valueFrom:
            configMapKeyRef:
              name: {APP_NAME}-config
              key: PORT
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: {APP_NAME}-secrets
              key: DATABASE_URL
        
        # Resource limits
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        
        # Health checks
        livenessProbe:
          httpGet:
            path: /health
            port: {APP_PORT}
          initialDelaySeconds: 30
          periodSeconds: 30
          timeoutSeconds: 5
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /ready
            port: {APP_PORT}
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        # Security context
        securityContext:
          runAsNonRoot: true
          runAsUser: 1001
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
      
      # Pod security context
      securityContext:
        fsGroup: 1001
        runAsNonRoot: true
        runAsUser: 1001
```

### Service Template
```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {APP_NAME}-service
  namespace: {K8S_NAMESPACE}
  labels:
    app: {APP_NAME}
spec:
  selector:
    app: {APP_NAME}
  ports:
  - name: http
    port: 80
    targetPort: {APP_PORT}
    protocol: TCP
  type: ClusterIP
```

### Ingress Template (Optional)
```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {APP_NAME}-ingress
  namespace: {K8S_NAMESPACE}
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - host: {APP_NAME}.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: {APP_NAME}-service
            port:
              number: 80
```

### Horizontal Pod Autoscaler Template (Optional)
```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {APP_NAME}-hpa
  namespace: {K8S_NAMESPACE}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {APP_NAME}
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

## 🔒 Registry-spezifische ImagePullSecrets

### Docker Hub ImagePullSecret
```yaml
# k8s/dockerhub-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: dockerhub-secret
  namespace: {K8S_NAMESPACE}
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>
```

### Azure ACR ImagePullSecret
```yaml
# k8s/acr-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: acr-secret
  namespace: {K8S_NAMESPACE}
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-acr-config>
```

## 🎯 Agent-Anweisungen

### Für den Cursor Agent:

1. **Analysiere die Anforderungen:** Verstehe App-Typ, Registry, Security-Anforderungen
2. **Wähle passende Templates:** Verwende Registry-spezifische Templates
3. **Passe Variablen an:** Ersetze alle Platzhalter mit den tatsächlichen Werten
4. **Erstelle alle Manifests:** Namespace, ConfigMap, Secrets, Deployment, Service
5. **Füge Security hinzu:** Resource Limits, Health Checks, Security Context
6. **Validiere YAML-Syntax:** Überprüfe auf korrekte YAML-Syntax
7. **Erkläre die Manifests:** Erkläre dem Benutzer, was jeder Manifest macht

### Beispiel-Interaktion:

```
Agent: Ich erstelle Kubernetes Manifests für eure Node.js Express API.
       Ich sehe, ihr verwendet Azure ACR. Ich erstelle:
       - Namespace und ConfigMap für Konfiguration
       - Secrets Template für sensitive Daten
       - Deployment mit Health Checks und Resource Limits
       - Service für Netzwerk-Zugriff

User: Perfekt! Könntest du auch Ingress hinzufügen?

Agent: Natürlich! Ich füge einen Ingress für externe Zugriffe hinzu.
       Das ermöglicht euch, die API über eine Domain zu erreichen.

User: Das ist großartig!

Agent: Alle Manifests sind erstellt! Sie sind bereit für euer Kubernetes Cluster.
       Soll ich sie in euer Projekt einfügen?
```

## 🔍 Validierung

Nach der Generierung soll der Agent folgende Punkte überprüfen:

- [ ] **YAML-Syntax:** Korrekte YAML-Syntax
- [ ] **Resource Limits:** CPU und Memory Limits gesetzt
- [ ] **Health Checks:** Liveness und Readiness Probes konfiguriert
- [ ] **Security Context:** Non-root User und Security Best Practices
- [ ] **ImagePullSecrets:** Registry-spezifische Secrets konfiguriert
- [ ] **Environment Variables:** ConfigMap und Secrets korrekt referenziert
- [ ] **Labels:** Konsistente Labels für alle Ressourcen
- [ ] **Namespace:** Alle Ressourcen im korrekten Namespace

## 📝 Deployment-Befehle

Nach der Generierung soll der Agent folgende Befehle bereitstellen:

### Manuelles Deployment
```bash
# Namespace und ConfigMap erstellen
kubectl apply -f k8s/namespace.yaml

# Secrets erstellen (Werte anpassen!)
kubectl apply -f k8s/secrets.yaml

# Registry Secret erstellen
kubectl create secret docker-registry {REGISTRY_TYPE}-secret \
  --docker-server={REGISTRY_NAME} \
  --docker-username={REGISTRY_USERNAME} \
  --docker-password={REGISTRY_PASSWORD} \
  --namespace={K8S_NAMESPACE}

# Deployment und Service erstellen
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# Optional: Ingress erstellen
kubectl apply -f k8s/ingress.yaml
```

### Deployment-Status prüfen
```bash
# Pods prüfen
kubectl get pods -n {K8S_NAMESPACE}

# Services prüfen
kubectl get services -n {K8S_NAMESPACE}

# Deployment-Status prüfen
kubectl rollout status deployment/{APP_NAME} -n {K8S_NAMESPACE}

# Logs anschauen
kubectl logs -f deployment/{APP_NAME} -n {K8S_NAMESPACE}
```

---

*Dieser Prompt ist darauf ausgelegt, mit dem Cursor Agent verwendet zu werden, um vollständige Kubernetes Manifests für bestehende Projekte zu generieren.*
