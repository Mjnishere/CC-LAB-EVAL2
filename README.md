# Basic RBAC with Service Accounts — Kubernetes Lab Evaluation 2

**Course:** Cloud Computing Lab (26ECAC301)  
**Problem:** 18 — Basic RBAC with Service Accounts  
**Environment:** Ubuntu + Visual Studio Code + Docker + Minikube + kubectl  
**Implementation:** Kubernetes RBAC

## 1. Overview

This project demonstrates Kubernetes Role-Based Access Control (RBAC) using a namespace-scoped Role and RoleBinding. A ServiceAccount named `pod-reader` is created in the `rbac-demo` namespace and is granted only the permissions needed to read Pods.

The Role allows:

- `get` on Pods
- `list` on Pods
- `watch` on Pods

The ServiceAccount is then checked using `kubectl auth can-i` to verify which operations are allowed and which are denied.

This follows the principle of least privilege: the ServiceAccount receives only the permissions required for its task.

## 2. Team

| Team member | SRN |
|---|---|
| Mudit Nayak | 01FE24BCI076 |
| Kartik | 01FE24BCI072 |
| Manoj | 01FE24BCI077 |
| Vinayak | ____________________ |

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
│   ├── 01-rbac-demo-setup.png
│   └── 02-rbac-resources-validation.png
├── README.md
├── .gitignore
└── .vscode/
```

## 4. Screenshots

### Screenshot 1 — RBAC setup and resource creation

![Screenshot 1 — RBAC setup and resource creation](screenshots/Screenshot%20from%202026-11-17%2000-01-05.png)

### Screenshot 2 — RBAC validation output

![Screenshot 2 — RBAC validation output](screenshots/Screenshot%20from%202026-11-17%2000-01-14.png)

## 5. RBAC Concepts

### Namespace
A Kubernetes namespace isolates resources within a logical boundary. This project uses `rbac-demo`.

### ServiceAccount
A ServiceAccount provides an identity for workloads or applications running inside Kubernetes. In this project, it is named `pod-reader`.

### Role
A Role defines permissions within a specific namespace. Here it grants access to `pods` for `get`, `list`, and `watch` operations.

### RoleBinding
A RoleBinding links the ServiceAccount to the Role, allowing the ServiceAccount to use the granted permissions only inside the namespace.

## 6. Prerequisites

Install the following on Ubuntu:

- Docker
- Minikube
- `kubectl`
- Visual Studio Code

Check the tools:

```bash
docker --version
kubectl version --client
minikube version
code --version
```

## 7. Start Minikube

```bash
minikube start --driver=docker
```

Verify the cluster:

```bash
kubectl get nodes
kubectl get pods -A
```

## 8. Apply the RBAC Manifests

From the project root:

```bash
kubectl apply -f manifests/01-namespace.yaml
kubectl apply -f manifests/02-serviceaccount.yaml
kubectl apply -f manifests/03-role.yaml
kubectl apply -f manifests/04-rolebinding.yaml
kubectl apply -f manifests/05-test-pod.yaml
```

Or use the helper script:

```bash
bash scripts/apply.sh
```

Wait until the Pod is Ready:

```bash
kubectl wait --for=condition=Ready pod/rbac-test-pod -n rbac-demo --timeout=120s
```

Inspect the created resources:

```bash
kubectl get serviceaccounts,roles,rolebindings,pods -n rbac-demo
```

## 9. Verify Authorization

The following checks should return `yes` because they are permitted by the Role:

```bash
kubectl auth can-i get pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
kubectl auth can-i list pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
kubectl auth can-i watch pods --as=system:serviceaccount:rbac-demo:pod-reader -n rbac-demo
```

The following checks should return `no` because they are not granted by the Role:

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

## 10. Cleanup

To delete the resources created by this lab:

```bash
bash scripts/cleanup.sh
```

This removes the `rbac-demo` namespace and its resources.

To stop the cluster:

```bash
minikube stop
```

To delete the cluster entirely:

```bash
minikube delete
```

## 11. Conclusion

This project demonstrates how RBAC restricts what an identity is allowed to do. By assigning only the minimum required permissions, Kubernetes enforces least-privilege access and improves security for workloads and applications.

## 12. References

- Kubernetes RBAC: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- Kubernetes ServiceAccounts: https://kubernetes.io/docs/concepts/security/service-accounts/
- Minikube: https://minikube.sigs.k8s.io/docs/start/
- kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
- VS Code: https://code.visualstudio.com/

