# Basic RBAC with Service Accounts — Kubernetes Lab Evaluation 2

**Course:** Cloud Computing Lab (26ECAC301)  
**Problem:** 18 — Basic RBAC with Service Accounts  
**Environment:** Ubuntu + Visual Studio Code + Docker + Minikube + kubectl  
**Implementation:** Native Kubernetes RBAC (OPA/Gatekeeper and a frontend are not required)

## 1. What this project demonstrates

This project creates a namespace called `rbac-demo` and a ServiceAccount called `pod-reader`. A namespace-scoped Role grants that ServiceAccount only these Pod permissions:

- `get` — read an individual Pod
- `list` — list Pods
- `watch` — watch for Pod changes

A RoleBinding connects the ServiceAccount to the Role. The implementation then checks allowed and ungranted operations with `kubectl auth can-i`.

The goal is to demonstrate **authorization** and the **Principle of Least Privilege**: an application should receive only the permissions it needs.

> Important: Kubernetes RBAC permissions are additive. RBAC does not have explicit deny rules. A test returns `no` only when no other applicable RoleBinding or ClusterRoleBinding grants that permission.

## 2. Team

| Team member | SRN |
|---|---|
| Mudit Nayak | 01FE24BCI076 |
| Kartik | 01FE24BCI072 |
| Manoj | 01FE24BCI077 |
| Vinayak | ____________________ |

Update Vinayak's SRN before submission.

## 3. Project structure

```text
basic-rbac-vscode/
├── .vscode/
│   └── extensions.json
├── manifests/
│   ├── 01-namespace.yaml
│   ├── 02-serviceaccount.yaml
│   ├── 03-role.yaml
│   ├── 04-rolebinding.yaml
│   └── 05-test-pod.yaml
├── scripts/
│   ├── apply.sh
│   ├── verify.sh
│   └── cleanup.sh
├── screenshots/
│   └── (save your actual evaluation screenshots here)
└── README.md
```

## 4. Concepts and architecture

### Namespace
A namespace groups and scopes Kubernetes resources. This project uses `rbac-demo`.

### ServiceAccount
A ServiceAccount is an identity intended for processes and workloads running in Kubernetes. This project uses `pod-reader`.

### Role
A Role defines permissions within one namespace. Here it allows `get`, `list`, and `watch` on the core API resource `pods`.

### RoleBinding
A RoleBinding associates a subject (the ServiceAccount) with a Role. Since the RoleBinding is in `rbac-demo`, the permissions it grants are limited to that namespace.

### Request flow

```text
ServiceAccount: pod-reader
        |
        v
RoleBinding: pod-reader-binding
        |
        v
Role: pod-reader-role
        |
        | permits get/list/watch on Pods
        v
Kubernetes API Server authorization check
        |
        +--> permitted Pod read request
        |
        +--> ungranted operations are not authorized
```

Authentication answers **who is making the request?** Authorization answers **what is that identity allowed to do?** The ServiceAccount supplies the workload identity; RBAC defines and assigns permissions.

## 5. Prerequisites on Ubuntu

You need:

1. Ubuntu with an internet connection.
2. Docker Engine (Minikube will use Docker as its driver).
3. Minikube.
4. `kubectl`.
5. Visual Studio Code.

Check whether they are installed:

```bash
docker --version
kubectl version --client
minikube version
code --version
```

### Install Docker (Ubuntu package)

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo systemctl status docker
```

To use Docker as your normal user:

```bash
sudo usermod -aG docker "$USER"
```

Then **log out of Ubuntu and log back in** (or restart the session). Docker-group membership grants powerful, effectively root-level access, so only add a trusted account. Verify:

```bash
docker ps
```

If Docker is already installed and working, do not reinstall it.

### Install kubectl

A convenient Ubuntu option is:

```bash
sudo snap install kubectl --classic
```

Verify:

```bash
kubectl version --client
```

### Install Minikube

For an x86-64 Ubuntu system:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64
```

Verify:

```bash
minikube version
```

If your computer uses ARM64, use the matching Minikube download for your architecture rather than the `amd64` binary.

### Install Visual Studio Code

Install VS Code using the official Ubuntu package or the official download page:
https://code.visualstudio.com/

Recommended extensions (optional but useful):
- Kubernetes — Microsoft
- YAML — Red Hat
- Docker — Microsoft

The project works without these extensions; the integrated terminal and YAML files are sufficient.

## 6. Open the project in VS Code

1. Download and extract the ZIP.
2. Open a terminal in the extracted folder.
3. Run:

```bash
code .
```

Alternatively, in VS Code choose **File → Open Folder** and select `basic-rbac-vscode`.

Open the integrated terminal using **Terminal → New Terminal** or `Ctrl+Shift+`` (the exact shortcut can vary by keyboard layout).

All commands below can be run in this integrated terminal.

## 7. Start the local Kubernetes cluster

Start Minikube with Docker:

```bash
minikube start --driver=docker
```

Check the cluster:

```bash
minikube status
kubectl get nodes
kubectl get pods -A
```

The Minikube node should show `Ready`. If Minikube is already running, `minikube start` will reuse the existing cluster.

## 8. Apply the RBAC manifests

From the project root, run:

```bash
kubectl apply -f manifests/01-namespace.yaml
kubectl apply -f manifests/02-serviceaccount.yaml
kubectl apply -f manifests/03-role.yaml
kubectl apply -f manifests/04-rolebinding.yaml
kubectl apply -f manifests/05-test-pod.yaml
```

Or run the helper script:

```bash
bash scripts/apply.sh
```

Wait for the test Pod:

```bash
kubectl wait --for=condition=Ready pod/rbac-test-pod -n rbac-demo --timeout=120s
```

Inspect the resources:

```bash
kubectl get pods,serviceaccounts,roles,rolebindings -n rbac-demo
kubectl describe role pod-reader-role -n rbac-demo
kubectl describe rolebinding pod-reader-binding -n rbac-demo
```

## 9. Verify allowed and ungranted operations

The `--as` option asks the API Server to evaluate the request as the named ServiceAccount. The account running `kubectl` must have permission to impersonate that ServiceAccount. Minikube's usual administrator context permits this; if you get an impersonation error, inspect the current context and permissions.

Run these commands:

### Expected allowed checks

```bash
kubectl auth can-i get pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
kubectl auth can-i list pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
kubectl auth can-i watch pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
```

Expected response for each: `yes`.

### Expected checks that are not granted by this Role

```bash
kubectl auth can-i create pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
kubectl auth can-i update pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
kubectl auth can-i delete pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
kubectl auth can-i list secrets --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
```

Expected response for each: `no`, provided no other applicable binding grants these permissions.

### Namespace-scope check

```bash
kubectl auth can-i list pods --as=system:serviceaccount:rbac-demo:pod-reader -n default
```

Expected response: `no`, provided no other applicable binding grants this ServiceAccount access in `default`.

### Optional real API request demonstration

A successful read:

```bash
kubectl get pods -n rbac-demo --as=system:serviceaccount:rbac-demo:pod-reader
```

An attempted delete (should be rejected with `Forbidden`):

```bash
kubectl delete pod rbac-test-pod -n rbac-demo --as=system:serviceaccount:rbac-demo:pod-reader
```

Do not run the delete command without `--as` if you intend to preserve the test Pod: your normal administrator identity may be allowed to delete it.

## 10. What each command does

| Command | Explanation |
|---|---|
| `minikube start --driver=docker` | Starts a local Kubernetes cluster using Docker. |
| `kubectl get nodes` | Lists cluster nodes and their status. |
| `kubectl apply -f FILE` | Creates or updates resources declared in a YAML file. |
| `kubectl get ... -n rbac-demo` | Lists selected resources in the specified namespace. |
| `kubectl describe role ...` | Displays Role rules and details. |
| `kubectl describe rolebinding ...` | Displays which subject is bound to which Role. |
| `kubectl auth can-i VERB RESOURCE` | Checks whether an identity is authorized for an API operation. |
| `--as=...` | Uses Kubernetes impersonation for the authorization check; caller needs impersonation permission. |
| `-n rbac-demo` | Sets the namespace for the command. |
| `kubectl wait ...` | Waits until the specified Pod condition is met. |

## 11. Troubleshooting

### `minikube start` fails
Check Docker:

```bash
sudo systemctl status docker
docker ps
```

If Docker permission is denied, confirm you logged out and back in after adding your user to the Docker group.

### `kubectl` cannot connect to the server
Check:

```bash
minikube status
kubectl config current-context
```

Start Minikube if it is stopped.

### Pod is not Ready
Inspect:

```bash
kubectl get pods -n rbac-demo
kubectl describe pod rbac-test-pod -n rbac-demo
kubectl get events -n rbac-demo
```

### `kubectl auth can-i` reports an impersonation error
The current identity needs permission to impersonate the ServiceAccount. Check:

```bash
kubectl auth whoami
kubectl auth can-i impersonate serviceaccounts
```

The exact authorization required may depend on cluster configuration. Do not grant broad permissions to solve this casually; use the intended administrator context for the lab.

### A denied check unexpectedly returns `yes`
Kubernetes RBAC permissions are additive. Another RoleBinding or ClusterRoleBinding may grant the permission. Inspect applicable bindings and remember that this project Role does not revoke permissions granted elsewhere.

## 12. Screenshots to capture for evaluation

Save actual screenshots in the `screenshots/` folder:

1. Minikube running and `kubectl get nodes`.
2. Namespace and ServiceAccount.
3. Role definition (`kubectl describe role`).
4. RoleBinding definition (`kubectl describe rolebinding`).
5. `can-i` checks returning `yes` for get/list/watch.
6. `can-i` checks returning `no` for create/update/delete and Secrets.
7. Namespace-scope check against `default`.
8. Optional actual read success and denied delete request.

Use real output from your own cluster. The expected responses in this README are not evidence that the commands have already been executed.

## 13. Cleanup

To remove only the resources created by this project:

```bash
bash scripts/cleanup.sh
```

This deletes the `rbac-demo` namespace and everything in it. It does not delete the Minikube cluster.

To stop Minikube but retain the cluster:

```bash
minikube stop
```

To delete the Minikube cluster entirely:

```bash
minikube delete
```

## 14. Key viva points

- **RBAC:** Role-Based Access Control; Kubernetes authorization mechanism.
- **ServiceAccount:** identity for a workload or process.
- **Role:** namespace-scoped set of permissions.
- **RoleBinding:** associates a subject with a Role or ClusterRole.
- **Namespace:** logical scope for namespaced resources.
- **Authentication:** establishes who the requester is.
- **Authorization:** decides what the requester may do.
- **Least Privilege:** grant only the permissions needed.
- **`get`:** retrieve a resource; **`list`:** retrieve a collection; **`watch`:** observe changes.
- **Role vs ClusterRole:** Role is namespace-scoped; ClusterRole is cluster-scoped in definition and can be bound at cluster or namespace scope.
- **RoleBinding vs ClusterRoleBinding:** RoleBinding grants within its namespace; ClusterRoleBinding grants cluster-wide.
- **OPA:** an additional policy engine; it is not required for this native RBAC assignment.
- **Frontend:** not required. YAML manifests and terminal verification are the deliverable.

## 15. References

- Kubernetes RBAC: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- Kubernetes ServiceAccounts: https://kubernetes.io/docs/concepts/security/service-accounts/
- Minikube start guide: https://minikube.sigs.k8s.io/docs/start/
- kubectl installation: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
- VS Code: https://code.visualstudio.com/
# CC-LAB-EVAL2
