What is a Cilium “identity”?
============================

Think of every pod/workload getting a **membership card** that says _who it is_ based on its **labels** (like `app=api`, `team=payments`, `ns=prod`).  
Cilium turns that set of labels into a small **number** called an **identity**.  
Same labels ⇒ same identity. Different labels ⇒ different identity.

Cilium then writes its firewall rules as **“identity A can talk to identity B on port X”** instead of “pod IP 10.0.1.23 can talk to 10.0.4.19…”. That’s the whole magic.

* * *

Why identities are useful
-------------------------

*   **Stable & scalable**: Pod IPs change; labels don’t (much). So rules don’t need to be rebuilt every time a pod restarts.
*   **Fast datapath**: In eBPF, checking **source-identity → dest-identity** is a quick lookup.
*   **Fewer rules**: 100 similar pods share **one** identity, so Cilium compiles **one** rule for them, not 100.

* * *

How an identity is created (behind the scenes)
----------------------------------------------

1.  A pod starts.
2.  Cilium reads its **labels** (namespace, app, team, etc.).
3.  It looks up: “Do these exact labels already have an identity?”
    *   **Yes:** reuse that identity number.
    *   **No:** allocate a new identity and remember it.
4.  Cilium maps the pod’s **IP → identity** in an internal **IP cache**.
5.  The eBPF programs use identities to allow/deny traffic.

> Tip: Identities are stored in Kubernetes via a `CiliumIdentity` CRD (operator manages them). You can list them with `kubectl get ciliumidentities`.

* * *

What labels count?
------------------

*   Normal **Kubernetes labels** on the pod and its namespace (e.g., `app`, `version`, `k8s:io.kubernetes.pod.namespace`).
*   Cilium ignores some noisy/ephemeral bits so identities don’t churn unnecessarily.

Keep your policy-relevant labels **simple and stable** (e.g., `app`, `role`, `team`, `env`).

* * *

Reserved identities (special cases)
-----------------------------------

Cilium also uses a few built-in identities for “things that aren’t pods”, for example:

*   **host** (the node itself)
*   **world** (external internet / outside the cluster)
*   **health** (Cilium’s health checks)
*   **remote-node** (peering nodes)

You reference these implicitly; Cilium handles the details.

* * *

How policies use identities (simple example)
--------------------------------------------

You write policies with **labels**. Cilium compiles them into **identity → identity** rules.

```yaml
# Allow web to call api on TCP 8080
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: allow-web-to-api
spec:
  endpointSelector:
    matchLabels:
      app: api
  ingress:
    - fromEndpoints:
        - matchLabels:
            app: web
      toPorts:
        - ports:
            - port: "8080"
              protocol: TCP
```

If every `app=web` pod shares identity **W**, and every `app=api` pod shares identity **A**, the datapath rule becomes:

> **W → A** allowed on **TCP/8080**

When packets flow, Cilium looks up **source IP → W**, **dest IP → A**, then decides.

* * *

Where to see identities in practice
-----------------------------------

*   **All identities**
    *   `kubectl get ciliumidentities`
    *   `kubectl describe ciliumidentity <ID>`
*   **Per-pod identity**
    *   On a node: `cilium endpoint list` (shows each endpoint’s identity)
*   **IP → identity mappings (ipcache)**
    *   On a node: `cilium bpf ipcache list` (or `cilium map get cilium_ipcache`)

* * *

Gotchas & tips
--------------

*   **Label hygiene matters**: avoid inventing a unique label per pod (that would create too many identities). Reuse common labels like `app`, `role`, `team`, `env`.
*   **Identity cleanup**: The Cilium operator garbage-collects unused identities; if you see identity “leaks,” check operator logs.
*   **Multi-cluster (Cluster Mesh)**: Identities can be shared across clusters so the same label set means the same “who-are-you” everywhere.

* * *

### One-line mental model

> **Identity = compressed labels number**.  
> Cilium enforces **who-can-talk-to-whom** by checking **numbers**, not every single IP.
