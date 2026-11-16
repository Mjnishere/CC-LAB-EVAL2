# Basic RBAC with Service Accounts

<p align="center">
  <img src="https://img.shields.io/badge/Kubernetes-RBAC-326ce5?style=for-the-badge&logo=kubernetes" alt="Kubernetes RBAC" />
  <img src="https://img.shields.io/badge/Ubuntu-22.04%2B-E95420?style=for-the-badge&logo=ubuntu" alt="Ubuntu" />
  <img src="https://img.shields.io/badge/Minikube-Local%20Cluster-00C7B7?style=for-the-badge&logo=docker" alt="Minikube" />
</p>

## Lab Evaluation 2

**Course:** Cloud Computing Lab (26ECAC301)  
**Problem:** 18 — Basic RBAC with Service Accounts  
**Environment:** Ubuntu + VS Code + Docker + Minikube + kubectl  
**Implementation:** Kubernetes RBAC

---

## 1. Objective

This project demonstrates how Kubernetes Role-Based Access Control (RBAC) restricts access to resources using a namespace-scoped `Role` and `RoleBinding`.

A `ServiceAccount` named `pod-reader` is created in the `rbac-demo` namespace and is granted only the permissions necessary to read Pods. The project verifies the difference between:

- allowed actions (`get`, `list`, `watch`)
- denied actions (`create`, `update`, `delete`, `list secrets`)

This follows the principle of least privilege and helps explain the core idea of authorization in Kubernetes.

---

## 2. Team

| Team member | SRN |
|---|---|
| Mudit Nayak | 01FE24BCI076 |
| Kartik | 01FE24BCI072 |
| Manoj | 01FE24BCI077 |
| Vinayak | ____________________ |

---

## 3. Project Structure

```text
basic-rbac-vscode/
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
│   ├── Screenshot from 2026-11-17 00-01-05.png
│   └── Screenshot from 2026-11-17 00-01-14.png
├── README.md
├── .gitignore
└── .vscode/
```

---

## 4. Key RBAC Concepts

### Namespace
A namespace provides a logical scope for resources. In this project, the namespace is `rbac-demo`.

### ServiceAccount
A `ServiceAccount` represents an identity for workloads running inside the cluster. Here it is used as `pod-reader`.

### Role
A `Role` grants permissions within a single namespace. This Role allows read-only Pod access.

### RoleBinding
A `RoleBinding` associates the ServiceAccount with the Role, granting access only within the chosen namespace.

---

## 5. Screenshots

### Screenshot 1 — RBAC Setup and Resource Creation

![Screenshot 1 — RBAC setup and resource creation](screenshots/Screenshot%20from%202026-11-17%2000-01-05.png)

### Screenshot 2 — RBAC Validation Output

![Screenshot 2 — RBAC validation output](screenshots/Screenshot%20from%202026-11-17%2000-01-14.png)

---

## 6. Prerequisites

Install the required tools on Ubuntu:

- Docker
- Minikube
- `kubectl`
- Visual Studio Code

Verify installation:

```bash
docker --version
kubectl version --client
minikube version
code --version
```

---

## 7. Start the Cluster

```bash
minikube start --driver=docker
```

Check the cluster state:

```bash
kubectl get nodes
kubectl get pods -A
```

---

## 8. Apply the RBAC Manifests

From the project root, run:

```bash
kubectl apply -f manifests/01-namespace.yaml
kubectl apply -f manifests/02-serviceaccount.yaml
kubectl apply -f manifests/03-role.yaml
kubectl apply -f manifests/04-rolebinding.yaml
kubectl apply -f manifests/05-test-pod.yaml
```

Or use the script:

```bash
bash scripts/apply.sh
```

Wait for the Pod to become Ready:

```bash
kubectl wait --for=condition=Ready pod/rbac-test-pod -n rbac-demo --timeout=120s
```

Inspect created resources:

```bash
kubectl get serviceaccounts,roles,rolebindings,pods -n rbac-demo
```

---

## 9. Authorization Checks

Allowed operations:

```bash
kubectl auth can-i get pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
kubectl auth can-i list pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
kubectl auth can-i watch pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
```

Denied operations:

```bash
kubectl auth can-i create pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
kubectl auth can-i update pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
kubectl auth can-i delete pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
kubectl auth can-i list secrets --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
```

Namespace restriction check:

```bash
kubectl auth can-i list pods --as=system:serviceaccount:rbac-demo:pod-reader -n default
```

Expected result: `no`.

---

## 10. Cleanup

Delete the project resources:

```bash
bash scripts/cleanup.sh
```

Stop the Minikube cluster:

```bash
minikube stop
```

Delete the cluster completely:

```bash
minikube delete
```

---

## 11. Conclusion

This project shows that RBAC enforces least-privilege access by granting only the minimum permissions needed for a workload. The ServiceAccount can read Pods in its namespace but cannot perform unauthorized operations beyond the defined Role.

---

## 12. References

- Kubernetes RBAC: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- Kubernetes ServiceAccounts: https://kubernetes.io/docs/concepts/security/service-accounts/
- Minikube: https://minikube.sigs.k8s.io/docs/start/
- kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
- VS Code: https://code.visualstudio.com/

