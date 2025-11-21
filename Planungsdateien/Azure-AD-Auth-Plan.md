# Azure AD Authentication Integration Plan

## Ziel
Azure Entra ID (Azure AD) Authentifizierung für OPNsense Firewall WebGUI und Kubernetes API Server via OIDC, für zentralisierte Cloud-basierte Zugriffskontrolle.

## Voraussetzungen
- Azure Tenant mit Entra ID aktiv ✅
- OPNsense 25.7.5-amd64 auf 192.168.1.1
- Kubernetes Cluster (k3s) im 192.168.2.x Netzwerk
- Admin-Zugriff auf Azure Portal und OPNsense
- **Internet-Zugriff für Kubernetes temporär nötig** (Plugin-Installation, Updates)

## Aktueller Stand

### Netzwerk-Architektur
```
Internet → FritzBox (192.168.0.x)
          ↓
    OPNsense WAN (DHCP von FritzBox)
          ↓
    OPNsense LAN (Port 1) → Router → Knuffelnet (192.168.0.x)
          ↓
    OPNsense Bridge0 (Port 2+3) → Kubernetes (192.168.2.x)
```

### Offene Probleme (vor Azure AD Integration)
1. **k3s Cluster-Reset nötig** (alte IP 192.168.1.101 → neue IP 192.168.2.48)
2. **Netzwerk-Routing** (192.168.0.x vs 192.168.1.x Verwirrung klären)
3. **Internet-Zugriff** für Kubernetes temporär öffnen

## Phase 0: Netzwerk-Vorbereitung

### 0.1 Netzwerk-Klarheit schaffen
**Entscheidung nötig:**
- Option A: Knuffelnet auf 192.168.1.x umstellen (einfach)
- Option B: Static Route 192.168.0.0/24 via Router (komplex)

### 0.2 Kubernetes Cluster-Reset
**Auf Master (.48):**
```bash
# k3s stoppen
sudo systemctl stop k3s

# Alte Cluster-Daten löschen
sudo rm -rf /var/lib/rancher/k3s/server/db
sudo rm -rf /var/lib/rancher/k3s/server/tls

# k3s mit neuer IP neu installieren
curl -sfL https://get.k3s.io | sh -s - server \
  --cluster-init \
  --write-kubeconfig-mode 644 \
  --tls-san 192.168.2.48 \
  --bind-address 192.168.2.48

# Status prüfen
sudo systemctl status k3s
kubectl get nodes
```

**Auf Worker (.22):**
```bash
# k3s-agent stoppen
sudo systemctl stop k3s-agent

# Alte Agent-Daten löschen
sudo rm -rf /var/lib/rancher/k3s/agent

# Token vom Master holen
# Auf Master: sudo cat /var/lib/rancher/k3s/server/node-token

# Worker neu joinen
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.2.48:6443 K3S_TOKEN=<TOKEN> sh -

# Status prüfen
sudo systemctl status k3s-agent
```

### 0.3 Firewall-Regel für Updates (temporär)
```
Firewall → Rules → KubernetesCluster
Regel: TEMP - Internet Access
  Action: Pass
  Source: KubernetesCluster net
  Destination: any
  Gateway: default
  → Save → Apply
```

## Phase 1: Azure Entra ID App-Registrierungen

### 1.1 App-Registrierung für OPNsense
**Azure Portal → Entra ID → App registrations → New registration:**

```
Name: OPNsense-Firewall-Auth
Supported account types: Single tenant
Redirect URI: https://192.168.1.1/oauth2/callback

Nach Erstellung notieren:
- Application (client) ID
- Directory (tenant) ID
```

**Certificates & secrets → New client secret:**
```
Description: OPNsense-OIDC
Expires: 24 months
→ Notieren: Client Secret Value (nur einmal sichtbar!)
```

**Token configuration → Add optional claim:**
```
Token type: ID tokens
Claims: email, preferred_username, upn
```

### 1.2 App-Registrierung für Kubernetes
**Neue App-Registrierung:**
```
Name: Kubernetes-API-Auth
Redirect URI: http://localhost:8000
```

**API permissions → Add:**
```
Microsoft Graph → Delegated:
- User.Read
- email
- openid
- profile
```

**Token configuration → Add optional claim:**
```
Token type: ID tokens + Access tokens
Claims: email, groups, preferred_username
```

**Client Secret erstellen** und notieren.

## Phase 2: OPNsense OIDC Integration

### 2.1 OIDC Plugin installieren
**System → Firmware → Plugins:**
```
Suche: auth
Plugin: os-auth-oidc
→ Install
```

### 2.2 OIDC Provider konfigurieren
**System → Access → Servers → Add:**
```
Type: OpenID Connect
Name: Azure-Entra-ID
Issuer URL: https://login.microsoftonline.com/{TENANT_ID}/v2.0
Authorization Endpoint: https://login.microsoftonline.com/{TENANT_ID}/oauth2/v2.0/authorize
Token Endpoint: https://login.microsoftonline.com/{TENANT_ID}/oauth2/v2.0/token
Client ID: {OPNSENSE_CLIENT_ID}
Client Secret: {OPNSENSE_CLIENT_SECRET}
Scope: openid profile email
Username Attribute: preferred_username
→ Save
```

### 2.3 OIDC Authentication aktivieren
**System → Access → Settings:**
```
Authentication Server: Azure-Entra-ID
☑ Allow fallback to local database
→ Save
```

### 2.4 Test OPNsense Login
```
1. Logout aus OPNsense
2. Login-Page sollte "Login with Azure" zeigen
3. Azure-Credentials eingeben
4. Redirect zurück zu OPNsense
```

## Phase 3: Kubernetes OIDC Integration

### 3.1 k3s API Server konfigurieren
**Auf Master (192.168.2.48):**

Datei erstellen: `/etc/rancher/k3s/config.yaml`
```yaml
kube-apiserver-arg:
  - oidc-issuer-url=https://login.microsoftonline.com/{TENANT_ID}/v2.0
  - oidc-client-id={KUBERNETES_CLIENT_ID}
  - oidc-username-claim=preferred_username
  - oidc-groups-claim=groups
```

**k3s neu starten:**
```bash
sudo systemctl restart k3s
```

### 3.2 RBAC-Rollen erstellen
**Datei: `azure-ad-rbac.yaml`**
```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: azure-admin-role
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: azure-admin-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: azure-admin-role
subjects:
- kind: User
  name: "{YOUR_AZURE_EMAIL}"
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: azure-readonly-role
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
```

**Apply:**
```bash
kubectl apply -f azure-ad-rbac.yaml
```

### 3.3 kubectl OIDC konfigurieren
**Windows PowerShell - kubelogin installieren:**
```bash
choco install kubelogin
```

**~/.kube/config anpassen:**
```yaml
users:
- name: azure-user
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: kubelogin
      args:
      - get-token
      - --login
      - devicecode
      - --server-id
      - {KUBERNETES_CLIENT_ID}
      - --client-id
      - {KUBERNETES_CLIENT_ID}
      - --tenant-id
      - {TENANT_ID}
```

### 3.4 Test Kubernetes Access
```bash
kubectl get nodes --user=azure-user
```

Sollte Device-Code-Prompt zeigen, dann Browser-Login, dann Nodes anzeigen.

## Phase 4: Härtung & Isolation

### 4.1 Firewall-Regeln einschränken
**Nach erfolgreicher Installation:**

```
Firewall → Rules → KubernetesCluster
→ TEMP-Regel löschen

Neue Regeln:
1. Kubernetes → Kubernetes (intern)
2. Kubernetes → Firewall (DNS/Management)
3. Block all other traffic
```

### 4.2 Promiscuous Mode deaktivieren
```
Interfaces → OPT1/OPT2
Promiscuous mode: ☐
→ Save → Apply
```

## Sicherheitshinweise

1. **OPNsense Fallback**: Local admin account aktiv lassen (falls Azure AD down)
2. **Token Rotation**: Client Secrets nach 24 Monaten erneuern
3. **Outbound HTTPS**: Firewall braucht Zugriff zu login.microsoftonline.com
4. **RBAC Minimal**: Start mit minimalen Permissions, erweitern bei Bedarf

## Benötigte Informationen

Vor dem Start sammeln:
- [ ] OPNsense App: Client ID, Tenant ID, Client Secret
- [ ] Kubernetes App: Client ID, Tenant ID, Client Secret
- [ ] Ihre Azure E-Mail für RBAC
- [ ] Domain softwar3.pro DNS konfiguriert (optional für Production)

## Kosten

- Entra ID (Free Tier): $0
- App Registrations: $0
- OIDC Authentication: $0
- **Total: $0/Monat** (nur Cloud-Auth, kein Arc/AADDS)

## Dateien zum Erstellen

- `/etc/rancher/k3s/config.yaml` - k3s OIDC Config
- `kubernetes/azure-ad-rbac.yaml` - RBAC Roles/Bindings
- `~/.kube/config` - kubectl OIDC User Config

## Troubleshooting

### OPNsense Login funktioniert nicht
- Plugin installiert? System → Firmware → Plugins
- Redirect URI korrekt? https://192.168.1.1/oauth2/callback
- Client Secret richtig kopiert? (kein Whitespace)

### Kubernetes OIDC funktioniert nicht
- k3s Logs: `journalctl -xeu k3s -f`
- API Server Args: `ps aux | grep kube-apiserver | grep oidc`
- Token-Validation: kubelogin sollte Device-Code zeigen

### Netzwerk-Probleme
- OPNsense braucht Outbound HTTPS zu login.microsoftonline.com
- Firewall → Rules → WAN: Allow HTTPS outbound
- DNS muss funktionieren (login.microsoftonline.com auflösen)

