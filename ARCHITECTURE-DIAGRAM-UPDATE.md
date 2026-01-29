# Architecture Diagram Update - Legacy → Current

## Problem z poprzednim diagramem (Legacy)

Poprzedni diagram architektury w `index.html` miał poważne luki architektoniczne:

### Zidentyfikowane GAP'y:

1. **Brak tutum-operator-k8s w diagramie**
   - Diagram pokazywał tylko CSI driver path
   - Operator nie był widoczny pomimo że jest wymieniony w komponentach

2. **Brak TutumCertificate CRD**
   - Kluczowy komponent dla operator mode był pominięty
   - Nie pokazano jak użytkownicy tworzą zasoby certyfikatów

3. **Brak rozróżnienia dwóch trybów injection**
   - CSI Driver Mode (legacy, privileged)
   - Operator Mode (recommended, rootless)

4. **Nieprawidłowe opisy**
   - "applications consume certificates through CSI drivers" - nieprawda dla operator mode
   - Brak informacji o Kubernetes Secrets w operator mode
   - Brak informacji o różnicach między trybami

5. **Brak kontekstu bezpieczeństwa**
   - Nie pokazano że CSI wymaga privileged containers
   - Nie pokazano że Operator jest rootless i bardziej secure

## Wprowadzone zmiany

### 1. Zaktualizowany diagram architektury

**Poprzednio:**
```
Administration Layer → tutum-engine
Container Layer → CSI Drivers → tutum-netlib → tutum-engine
```

**Obecnie:**
```
Administration Layer → tutum-engine

Kubernetes (dwa tryby):
├── Operator Mode (Recommended):
│   User creates CRD → Operator watches → Syncs with Engine → Creates/updates Secrets → Pods mount Secrets
│
└── CSI Driver Mode (Legacy):
    Pods → CSI Driver → tutum-netlib → Engine

Docker:
    Containers → CSI Driver → tutum-netlib → Engine
```

### 2. Dodane komponenty w diagramie

- **TutumCertificate CRD** - pokazuje jak użytkownicy deklarują certyfikaty
- **tutum-operator-k8s** - controller obserwujący CRD
- **Kubernetes Secrets** - natywne zasoby tworzone przez operatora
- **Workflow strzałki** - pokazują przepływ danych w operator mode

### 3. Kolorowanie i wizualne rozróżnienie

- **Operator Mode** - zielony (🎯 recommended)
  - fill: `#d1fae5` (light green)
  - CRD: `#10b981` (emerald)
  - Secrets: `#34d399` (green)

- **CSI Driver Mode** - żółty (⚠️ legacy)
  - fill: `#fef3c7` (light yellow/warning)
  - Driver: `#f59e0b` (amber)

- **tutum-engine** - zielony (core)
- **Docker** - niebieski (standard)

### 4. Dodane porównanie trybów

Dwie karty (cards) pokazujące różnice:

**Operator Mode (Recommended):**
- ✅ Rootless - no privileged containers
- ✅ Native - uses Kubernetes Secrets
- ✅ Declarative - TutumCertificate CRD
- ✅ Auto-sync - kubelet updates volumes automatically (~60s)
- ✅ OpenShift - compatible with restricted SCCs

**CSI Driver Mode (Legacy):**
- ⚠️ Privileged - requires escalated permissions
- ⚠️ Direct mount - CSI ephemeral volumes
- ⚠️ Pod restart - required for cert updates
- ⚠️ DaemonSet - runs on every node
- ⚠️ Use case - legacy deployments

### 5. Zaktualizowane opisy tekstowe

**Przed:**
> "applications consume certificates through CSI drivers"

**Po:**
> "For certificate distribution, Kubernetes supports two modes: CSI Driver (volume-based) and Operator (CRD-based). Docker uses CSI Driver only."

### 6. Dodany alert box z wyjaśnieniem

```html
<div class="alert alert-warning">
  <strong>Kubernetes Injection Modes:</strong>
  <ul>
    <li><strong>CSI Driver Mode:</strong> Mounts certificates as ephemeral volumes (requires privileged containers)</li>
    <li><strong>Operator Mode:</strong> Syncs certificates to Kubernetes Secrets via TutumCertificate CRD (rootless, recommended)</li>
  </ul>
</div>
```

### 7. Zaktualizowana sekcja "Seamless Integration"

**Dodano pierwszą pozycję:**
- ✅ Kubernetes Operator with CRD (rootless, recommended)

**Zmieniono:**
- Native Kubernetes CSI driver for ephemeral volumes → + *(legacy)*

### 8. Zaktualizowana sekcja "Operational Efficiency"

**Przed:**
- Automatic pod restart on certificate updates (K8s)

**Po:**
- Automatic Secret sync (Operator) or pod restart (CSI)

### 9. Zaktualizowana sekcja "Getting Started"

**Dodano deployment operatora:**
```bash
# Deploy Kubernetes Operator (Recommended)
cd tutum-operator-k8s
kubectl apply -f deploy/crd.yaml
kubectl apply -f deploy/operator.yaml
```

**Oznaczono CSI jako legacy:**
```bash
# Or deploy CSI Driver (Legacy)
```

## Komunikacja: Operator → Engine (gRPC)

**Pytanie:** Czy operator używa tutum-netlib?

**Odpowiedź:** NIE. Operator komunikuje się bezpośrednio z Engine przez gRPC.

### Różnica w komunikacji:

```
CSI Driver → tutum-netlib.so → gRPC client → Engine:9090
   (system plugin, C API, needs shared library)

Operator → gRPC client (Go native) → Engine:9090
   (K8s controller, Go application, imports gRPC directly)
```

### Dlaczego?

- **tutum-netlib** jest dla **system plugins** (CSI drivers)
  - CSI driver działa na niskim poziomie (C API, systemd)
  - Potrzebuje shared object (.so) dla gRPC funkcjonalności

- **tutum-operator-k8s** jest **Kubernetes controller** (Go)
  - Importuje bezpośrednio: `google.golang.org/grpc`
  - Używa generated Go code z proto definitions
  - Nie potrzebuje external shared library

### Kod operatora (przykład):

```go
import (
    "google.golang.org/grpc"
    pb "tutum/proto/gen/go/v1"
)

// Direct gRPC connection
conn, _ := grpc.Dial("tutum-engine.tutum-system.svc:9090", grpc.WithInsecure())
client := pb.NewCertificateServiceClient(conn)

// Call gRPC methods
resp, _ := client.GetCertificate(ctx, &pb.GetCertificateRequest{
    Name: "my-certificate",
})
```

**Wniosek:** Strzałka `OPERATOR -->|gRPC| GRPC` w diagramie jest **POPRAWNA** ✅

## Architektura Operator Mode (szczegóły)

### Flow dla Operator Mode:

```
1. User creates TutumCertificate CRD:
   apiVersion: tutum.io/v1
   kind: TutumCertificate
   spec:
     certificateName: "my-cert"
     secretName: my-app-tls

2. Operator watches CRD (via Kubernetes API)

3. Operator syncs with tutum-engine (gRPC:9090)
   - GetCertificate(name, version)
   - Returns: certificate, private key, CA

4. Operator creates/updates Kubernetes Secret:
   apiVersion: v1
   kind: Secret
   type: kubernetes.io/tls
   data:
     tls.crt: <base64>
     tls.key: <base64>
     ca.crt: <base64>

5. Kubernetes kubelet auto-updates volume mounts (~60s sync)

6. Application pod sees new certificate files without restart
```

### Porównanie z CSI Mode:

| Aspekt | Operator Mode | CSI Driver Mode |
|--------|---------------|-----------------|
| Privileged containers | ❌ No | ✅ Yes (DaemonSet) |
| Kubernetes resource | Secret | CSI Volume |
| Deklaracja | TutumCertificate CRD | Pod volumeMount |
| Auto-update | ✅ Kubelet sync | ❌ Requires pod restart |
| OpenShift SCC | ✅ restricted-v2 | ❌ privileged |
| RBAC | ClusterRole (Secrets) | ClusterRole (CSI) |
| Deployment | Single Deployment | DaemonSet (per node) |
| Resource usage | Low (1 pod) | Medium (N pods) |
| Hot-reload | ✅ Automatic (~60s) | ❌ Manual restart |

## Wpływ na użytkowników

### Dla nowych deploymentów:
- **Zalecenie**: Operator Mode
- **Powód**: Rootless, OpenShift-compatible, auto-sync
- **Instrukcje**: Updated w Getting Started

### Dla istniejących deploymentów CSI:
- **Status**: Nadal wspierane (legacy)
- **Migracja**: Opcjonalna, nie wymagana
- **Benefit migracji**: Lepsze bezpieczeństwo, mniej zasobów

### Dla OpenShift:
- **Operator Mode**: ✅ Działa out-of-box z restricted-v2 SCC
- **CSI Mode**: ⚠️ Wymaga privileged SCC

## Pliki zmienione

- `tutum-pro-docs/index.html` - zaktualizowany diagram i opisy (główny overview)
- `tutum-pro-docs/architecture.html` - zaktualizowany diagram i opisy (dedykowana strona architektury)

## Weryfikacja

Otwórz `index.html` w przeglądarce i sprawdź:

1. ✅ Diagram pokazuje dwa tryby dla Kubernetes
2. ✅ Operator Mode jest wizualnie wyróżniony (zielony)
3. ✅ CSI Mode jest oznaczony jako legacy (żółty)
4. ✅ TutumCertificate CRD jest widoczny
5. ✅ Kubernetes Secrets są pokazane
6. ✅ Dwie karty porównawcze są widoczne
7. ✅ Alert box wyjaśnia różnice
8. ✅ Opisy tekstowe są zaktualizowane

## Następne kroki

1. Rozważ aktualizację `architecture.html` z podobnymi zmianami
2. Dodaj dedykowaną stronę "Operator vs CSI Comparison"
3. Zaktualizuj `operator-k8s.html` z więcej szczegółami o CRD
4. Dodaj migration guide dla użytkowników CSI → Operator

## Benefity zaktualizowanego diagramu

1. **Przejrzystość** - jasno pokazuje dwa różne podejścia
2. **Aktualność** - odzwierciedla current best practices
3. **Edukacja** - użytkownicy widzą różnice i zalecenia
4. **Bezpieczeństwo** - podkreśla security benefits operatora
5. **OpenShift** - jasno pokazuje który tryb jest kompatybilny
