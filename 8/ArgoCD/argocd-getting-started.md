# Argo CD Getting Started — argocd-cluster (ca-central-1)

Based on [Argo CD Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/).

---

## 1. Install Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

*(Optional)* Use a pinned version for production, e.g. replace `stable` with `v3.2.0` in the URL.

---

## 2. Argo CD CLI

Install if needed:

```bash
brew install argocd
```

---

## 3. Access Argo CD

**Option A — Port forward (easiest for local access):**

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then open **https://localhost:8080** (accept the self-signed cert or use `--insecure` with the CLI).

**Option B — LoadBalancer (if you want an external IP):**

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl get svc argocd-server -n argocd -o=jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

---

## 4. Login (CLI)

Get the initial admin password:

```bash
argocd admin initial-password -n argocd
```

Login (with port-forward running on 8080):

```bash
argocd login localhost:8080 --insecure
```

Change the password:

```bash
argocd account update-password
```

*(Optional)* To use port-forward automatically with the CLI:

```bash
export ARGOCD_OPTS='--port-forward-namespace argocd'
```

---

## 5. Register another cluster (optional)

Only needed if you want Argo CD to deploy to a *different* cluster. For apps on this same cluster, use `https://kubernetes.default.svc` as the destination server.

To add another context from your kubeconfig:

```bash
kubectl config get-contexts -o name
argocd cluster add <CONTEXTNAME>
```

---

## 6. Create an application

**CLI example (guestbook on this cluster):**

```bash
kubectl config set-context --current --namespace=argocd

argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

Or create the app from the UI: **+ New App** → set repo, path, destination cluster/namespace.

---

## 7. Sync (deploy)

```bash
argocd app get guestbook
argocd app sync guestbook
```

Or use the **Sync** button in the UI.

---

## Quick reference

| Step        | Command / action |
|------------|-------------------|
| Set context| `kubectl config set-context --current --namespace=argocd` |
| Port-forward | `kubectl port-forward svc/argocd-server -n argocd 8080:443` |
| Get password | `argocd admin initial-password -n argocd` |
| Login | `argocd login localhost:8080 --insecure` |

---

## Cluster integration with Elastic (logs, metrics, traces)

Summary of what you’re doing to send cluster observability to Elastic using the OpenTelemetry Operator and Elastic’s kube-stack.

### 1. Add OpenTelemetry Helm repo

```bash
helm repo add open-telemetry 'https://open-telemetry.github.io/opentelemetry-helm-charts' --force-update
```

### 2. Install OpenTelemetry Operator (kube-stack)

- Create namespace: `opentelemetry-operator-system`
- Create secret `elastic-secret-otel` in that namespace with:
  - `elastic_endpoint` — your Elastic Cloud URL (e.g. `https://…gcp.elastic-cloud.com:443`)
  - `elastic_api_key` — API key for the Elastic deployment
- Install the kube-stack chart using the local **`kube-stack-values.yaml`** (copy of [elastic-agent v9.3.1](https://raw.githubusercontent.com/elastic/elastic-agent/refs/tags/v9.3.1/deploy/helm/edot-collector/kube-stack/values.yaml)), chart version 0.12.4:  
  `helm upgrade --install opentelemetry-kube-stack open-telemetry/opentelemetry-kube-stack -n opentelemetry-operator-system -f kube-stack-values.yaml --version 0.12.4`  
- Optional: install cert-manager and tune `values.yaml` for production (e.g. cert renewal).

### 3. Instrument applications (optional)

- Use the Operator’s auto-instrumentation for supported runtimes (Node.js, Java, Python, .NET, Go).
- Either annotate specific Deployment pods or annotate the namespace so all pods get instrumentation.
- Restart deployments and confirm annotations/injection: `kubectl rollout restart deployment <name> -n <ns>`, then `kubectl describe pod <pod> -n <ns>`.
- For other languages, use the OpenTelemetry docs for manual instrumentation.

### 4. Visualize in Elastic

- **Kubernetes cluster health:** Kubernetes Cluster Dashboard.
- **Application services:** Service inventory and related assets.
- Use Elastic’s pre-built assets for logs, metrics, and traces.

**Status:** Metrics are appearing in **[OTEL][Metrics Kubernetes] Cluster Overview** — success so far.

**Troubleshooting — cluster-stats collector crash-loop ("node name is empty"):** The cluster collector runs as a Deployment; the EKS resourcedetection processor needs the pod’s node name. Fix: set `clusterName` in `kube-stack-values.yaml` and add `OTEL_K8S_NODE_NAME` to the cluster collector’s `env` from the downward API (`spec.nodeName`). Then re-run the Helm upgrade (see Step 2).

**Troubleshooting — upgrade failed ".spec.env" conflict with manager:** The Operator manages `.spec.env` on the gateway OpenTelemetryCollector, so Helm hits a server-side apply conflict and `--force`/`--force-replace` can't be used with server-side apply. **Uninstall and reinstall** with the updated values:  
`helm uninstall opentelemetry-kube-stack -n opentelemetry-operator-system`  
then re-run the install (Step 2) with the same `helm upgrade --install` command and your `kube-stack-values.yaml`. Recreate the `elastic-secret-otel` secret in that namespace if the uninstall removed it.

---

## Monitor your Kubernetes cluster with standalone Elastic Agent

*In addition to the OpenTelemetry kube-stack above; may overlap in coverage but adds another layer of metrics/logs collection.*

This install configures and collects metrics and logs by deploying a standalone Elastic Agent on the cluster.

### Step 1 — Install standalone Elastic Agent on Kubernetes

- Add Helm repo and install the Elastic Agent (e.g. into `kube-system`).  
- **Note:** The default manifest includes resource limits that may not be suitable for production; see [Scaling Elastic Agent on Kubernetes](https://www.elastic.co/guide/en/fleet/current/elastic-agent-kubernetes.html) before deploying.

```bash
helm repo add elastic https://helm.elastic.co/

helm install elastic-agent elastic/elastic-agent --version 9.3.1 -n kube-system \
  --set outputs.default.url='https://<your-deployment-id>.us-west2.gcp.elastic-cloud.com:443' \
  --set kubernetes.onboardingID=<your-onboarding-id> \
  --set kubernetes.enabled=true \
  --set outputs.default.type=ESPlainAuthAPI \
  --set outputs.default.api_key=$(echo "<base64-encoded-api-key>" | base64 -d)
```

*Replace `<your-deployment-id>`, `<your-onboarding-id>`, and `<base64-encoded-api-key>` with your Elastic Cloud URL, Kubernetes onboarding ID, and API key (or set the API key from an env var / secret instead of inline).*

### Step 2 — Monitor your Kubernetes cluster

*Incomplete — to be filled in as you complete the workflow.*
