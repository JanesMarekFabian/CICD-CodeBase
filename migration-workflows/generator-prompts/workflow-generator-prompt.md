# GitHub Actions Workflow Generator Prompt

## 🎯 Zweck

Dieser Prompt hilft dem Cursor Agent dabei, vollständige GitHub Actions Workflows für CI/CD-Pipelines zu generieren. Der Agent soll Workflows für verschiedene CI/CD-Strategien erstellen.

## 📋 Prompt für den Agent

### Kontext
Du bist ein Experte für GitHub Actions und CI/CD-Pipelines. Du sollst GitHub Actions Workflows für ein bestehendes Projekt erstellen. Das Projekt wurde analysiert und die folgenden Informationen sind verfügbar:

**Projekt-Informationen:**
- Programmiersprache: `{LANGUAGE}`
- Framework: `{FRAMEWORK}`
- Registry: `{REGISTRY_TYPE}` (dockerhub/acr)
- Registry Name: `{REGISTRY_NAME}`
- Image Name: `{IMAGE_NAME}`
- Kubernetes Namespace: `{K8S_NAMESPACE}`
- Kubernetes Master IP: `{K8S_MASTER_IP}`

### Aufgabe
Erstelle GitHub Actions Workflows mit folgenden Anforderungen:

#### 1. CI Workflow (Continuous Integration)
- **Trigger:** Push, Pull Request
- **Steps:** Checkout → Setup → Build → Test
- **Kein Deployment:** Nur Build und Test

#### 2. CD Workflow (Continuous Deployment)
- **Trigger:** Push auf main branch
- **Steps:** Build → Test → Docker → Push → Deploy
- **Registry Integration:** Docker Hub oder Azure ACR
- **Kubernetes Deployment:** Rolling Updates

#### 3. Manual Deploy Workflow
- **Trigger:** workflow_dispatch
- **Steps:** Deploy spezifische Version
- **Features:** Rollbacks und Hotfixes

### Output-Format
Erstelle folgende Dateien:

1. **.github/workflows/ci.yml** - CI Workflow
2. **.github/workflows/cd.yml** - CD Workflow
3. **.github/workflows/manual-deploy.yml** - Manual Deploy
4. **.github/workflows/security-scan.yml** - Security Scanning (optional)

### Registry-spezifische Konfiguration

#### Docker Hub
```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}

- name: Build and push Docker image
  run: |
    docker build -t ${{ secrets.DOCKER_USERNAME }}/${{ env.IMAGE_NAME }}:latest \
                 -t ${{ secrets.DOCKER_USERNAME }}/${{ env.IMAGE_NAME }}:${{ github.sha }} .
    docker push ${{ secrets.DOCKER_USERNAME }}/${{ env.IMAGE_NAME }}:latest
    docker push ${{ secrets.DOCKER_USERNAME }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

#### Azure ACR
```yaml
- name: Login to Azure ACR
  uses: azure/docker-login@v1
  with:
    login-server: ${{ env.ACR_LOGIN_SERVER }}
    username: ${{ secrets.AZURE_CLIENT_ID }}
    password: ${{ secrets.AZURE_CLIENT_SECRET }}

- name: Build and push Docker image
  run: |
    docker build -t ${{ env.ACR_LOGIN_SERVER }}/${{ env.IMAGE_NAME }}:latest \
                 -t ${{ env.ACR_LOGIN_SERVER }}/${{ env.IMAGE_NAME }}:${{ github.sha }} .
    docker push ${{ env.ACR_LOGIN_SERVER }}/${{ env.IMAGE_NAME }}:latest
    docker push ${{ env.ACR_LOGIN_SERVER }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

## 🔧 Workflow-Templates

### CI Workflow Template
```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '18'
  IMAGE_NAME: ${{ github.event.repository.name }}

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

    - name: Run linting
      run: npm run lint

    - name: Build application
      run: npm run build

    - name: Upload build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: build-files
        path: dist/
```

### CD Workflow Template
```yaml
name: CD

on:
  push:
    branches: [ main ]

env:
  NODE_VERSION: '18'
  IMAGE_NAME: ${{ github.event.repository.name }}
  REGISTRY: ${{ secrets.REGISTRY_NAME }}
  K8S_NAMESPACE: ${{ secrets.K8S_NAMESPACE }}

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

    - name: Build application
      run: npm run build

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Login to Container Registry
      # Registry-spezifischer Login (siehe oben)

    - name: Build and push Docker image
      # Registry-spezifischer Build (siehe oben)

    - name: Set up kubectl
      uses: azure/setup-kubectl@v4

    - name: Configure kubectl
      run: |
        mkdir -p ~/.kube
        echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > ~/.kube/config
        chmod 600 ~/.kube/config

    - name: Deploy to Kubernetes
      run: |
        kubectl set image deployment/${{ env.IMAGE_NAME }} \
          ${{ env.IMAGE_NAME }}=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
          -n ${{ env.K8S_NAMESPACE }}
        kubectl rollout status deployment/${{ env.IMAGE_NAME }} \
          -n ${{ env.K8S_NAMESPACE }} --timeout=5m

    - name: Verify deployment
      run: |
        kubectl get pods -n ${{ env.K8S_NAMESPACE }} -l app=${{ env.IMAGE_NAME }}
        kubectl logs -l app=${{ env.IMAGE_NAME }} -n ${{ env.K8S_NAMESPACE }} --tail=50
```

### Manual Deploy Workflow Template
```yaml
name: Manual Deploy

on:
  workflow_dispatch:
    inputs:
      image_tag:
        description: 'Image tag to deploy'
        required: true
        default: 'latest'
      environment:
        description: 'Environment to deploy to'
        required: true
        default: 'production'
        type: choice
        options:
        - production
        - staging

env:
  IMAGE_NAME: ${{ github.event.repository.name }}
  REGISTRY: ${{ secrets.REGISTRY_NAME }}
  K8S_NAMESPACE: ${{ inputs.environment }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Set up kubectl
      uses: azure/setup-kubectl@v4

    - name: Configure kubectl
      run: |
        mkdir -p ~/.kube
        echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > ~/.kube/config
        chmod 600 ~/.kube/config

    - name: Deploy to Kubernetes
      run: |
        kubectl set image deployment/${{ env.IMAGE_NAME }} \
          ${{ env.IMAGE_NAME }}=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ inputs.image_tag }} \
          -n ${{ env.K8S_NAMESPACE }}
        kubectl rollout status deployment/${{ env.IMAGE_NAME }} \
          -n ${{ env.K8S_NAMESPACE }} --timeout=5m

    - name: Verify deployment
      run: |
        kubectl get pods -n ${{ env.K8S_NAMESPACE }} -l app=${{ env.IMAGE_NAME }}
        echo "Deployment completed successfully!"
```

## 🔒 Security Scanning Workflow

### Vulnerability Scanning Template
```yaml
name: Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  IMAGE_NAME: ${{ github.event.repository.name }}
  REGISTRY: ${{ secrets.REGISTRY_NAME }}

jobs:
  security-scan:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Build Docker image
      run: docker build -t ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} .

    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: '${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}'
        format: 'sarif'
        output: 'trivy-results.sarif'

    - name: Upload Trivy scan results to GitHub Security tab
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'
```

## 🎯 Agent-Anweisungen

### Für den Cursor Agent:

1. **Analysiere die Konfiguration:** Verstehe Registry-Typ, Kubernetes-Setup und Projekt-Anforderungen
2. **Wähle passende Templates:** Verwende Registry-spezifische Templates
3. **Passe Variablen an:** Ersetze alle Platzhalter mit den tatsächlichen Werten
4. **Erstelle alle Workflows:** CI, CD, Manual Deploy und optional Security Scan
5. **Validiere YAML-Syntax:** Überprüfe auf korrekte YAML-Syntax
6. **Erkläre die Workflows:** Erkläre dem Benutzer, was jeder Workflow macht

### Beispiel-Interaktion:

```
Agent: Ich erstelle GitHub Actions Workflows für euer Projekt.
       Ich sehe, ihr verwendet Azure ACR als Registry. Ich erstelle:
       - CI Workflow für Tests bei jedem Push/PR
       - CD Workflow für automatische Deployments
       - Manual Deploy für Rollbacks

User: Perfekt! Könntest du auch Security Scanning hinzufügen?

Agent: Natürlich! Ich füge einen Trivy Security Scan Workflow hinzu.
       Das scannt eure Docker Images auf Vulnerabilities.

User: Das ist großartig!

Agent: Alle Workflows sind erstellt! Sie sind bereit für euer GitHub Repository.
       Soll ich sie in euer Projekt einfügen?
```

## 🔍 Validierung

Nach der Generierung soll der Agent folgende Punkte überprüfen:

- [ ] **YAML-Syntax:** Korrekte YAML-Syntax
- [ ] **Secrets:** Alle benötigten Secrets referenziert
- [ ] **Environment Variables:** Alle Variablen korrekt gesetzt
- [ ] **Registry Integration:** Registry-spezifische Konfiguration korrekt
- [ ] **Kubernetes Integration:** kubectl und Deployment-Steps korrekt
- [ ] **Security:** Security Best Practices befolgt
- [ ] **Performance:** Workflow-Optimierungen implementiert

## 📝 Benötigte GitHub Secrets

### Für Docker Hub:
- `DOCKER_USERNAME` - Docker Hub Username
- `DOCKER_PASSWORD` - Docker Hub Token
- `KUBE_CONFIG` - Base64-encoded kubeconfig

### Für Azure ACR:
- `AZURE_CLIENT_ID` - Service Principal Client ID
- `AZURE_CLIENT_SECRET` - Service Principal Client Secret
- `AZURE_TENANT_ID` - Azure Tenant ID
- `ACR_LOGIN_SERVER` - ACR Login Server
- `KUBE_CONFIG` - Base64-encoded kubeconfig

### Allgemein:
- `K8S_NAMESPACE` - Kubernetes Namespace
- `REGISTRY_NAME` - Container Registry Name

---

*Dieser Prompt ist darauf ausgelegt, mit dem Cursor Agent verwendet zu werden, um vollständige GitHub Actions Workflows für CI/CD-Pipelines zu generieren.*
