# Deployments Deep Dive

## 1. The Deployment Concept
A **Deployment** is your main interface for managing applications in Kubernetes.
While you *can* create Pods and ReplicaSets directly, you shouldn't.
*   **The Hierarchy:** You create a **Deployment**, which manages a **ReplicaSet**, which manages the **Pods**.
*   **The Rule:** In production, you almost always manage Deployments. K8s handles the rest.

---

## 2. Deployments vs. ReplicaSets
If you look at the YAML for both, they are nearly identical. The only real change is `kind: Deployment`.
**So why the extra layer?**
Because a ReplicaSet is dumb. It only knows how to count ("I need 3 pods").
A Deployment is smart. It knows how to:
*   Perform **Rolling Updates** (v1 -> v2 without downtime).
*   **Rollback** if something breaks.
*   **Pause** a rollout in the middle.

---

## 3. The Hierarchy of Power
Understanding how features are inherited is key to understanding K8s:
1.  **Containers:** Give you lightweight isolation.
2.  **Pods:** Give containers a shared IP and localhost.
3.  **ReplicaSets:** Give certain Pods self-healing and scaling powers.
4.  **Deployments:** Give ReplicaSets the ability to change over time (Updates/Versions).

---

## 4. The Case for Deployments
*   **Scenario:** You have 3 pods running version 1.0. You want to upgrade to version 2.0.
*   **With just ReplicaSets:** You'd have to delete the old RS and create a new one manually. Downtime!
*   **With Deployments:** You change one line in the YAML (`image: v2.0`), and the Deployment automatically orchestrates a smooth transition.

---

## 5. Setting Up the Cluster (Again)
*Refresher from previous sections:*
1.  **Hostname:** `hostnamectl set-hostname controlplane`
2.  **Init:** `kubeadm init`
3.  **Network:** Install Weave/Flannel CNI so CoreDNS starts working.
4.  **Verify:** `kubectl get nodes` should show `Ready`.

---

## 6. Recap: ReplicaSets in Action
You can create a standalone ReplicaSet:
```bash
kubectl apply -f replica-set.yaml
```
And scale it:
```bash
kubectl scale rs my-rs --replicas=5
```
But remember, this is the "old way" or "manual way."

---

## 7. Enter the Deployment
To check all your workload resources at once:
```bash
kubectl get all
# or specifically
kubectl get deploy,rs,po
```

When you create a Deployment (`nginx-deploy`), you'll see:
1.  **Deployment:** `nginx-deploy`
2.  **ReplicaSet:** `nginx-deploy-65f8abcd` (Random hash added)
3.  **Pods:** `nginx-deploy-65f8abcd-xyz`

---

## 8. Three Ways to Scale
Scaling isn't just "add more pods."
1.  **HPA (Horizontal):** Add more Pod replicas. (CPU High? -> More Pods).
2.  **VPA (Vertical):** Make the Pods bigger. (CPU High? -> Give Pod more CPU).
3.  **CA (Cluster Autoscaler):** Add more Nodes. (Cluster Full? -> Buy more EC2 instances).

---

## 9. Why Deployments Rule Production
Standalone Pods and ReplicaSets lack **Version History**.
Deployments keep a record of every change you make (`Revision 1`, `Revision 2`). This history is what allows you to "Undo" a bad deployment instantly.

---

## 10. How Deployments Manage ReplicaSets
A Deployment manages releases by creating **New ReplicaSets**.
*   **State V1:** Deployment points to `ReplicaSet-1` (3 replicas).
*   **Update V2:** Deployment creates `ReplicaSet-2` (0 replicas).
*   **Transition:** It slowly scales `ReplicaSet-2` **UP** and `ReplicaSet-1` **DOWN**.

---

## 11. Writing a Deployment YAML
It looks just like a ReplicaSet, but with `kind: Deployment`.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
```

---

## 12. The Deployment Lifecycle
When you `apply` this file:
1.  **API Server:** Saves the definition to etcd.
2.  **Deployment Controller:** Notices the new Deployment. Creates a ReplicaSet.
3.  **ReplicaSet Controller:** Notices the new RS. Creates 3 Pods.
4.  **Scheduler:** Assigns the Pods to Nodes.
5.  **Kubelet:** Starts the containers.

---

## 13. Zero Downtime: Rolling Updates
The default strategy is `RollingUpdate`.
*   **MaxUnavailable (25%):** It ensures at least 75% of your app is up during the update.
*   **MaxSurge (25%):** It allows the app to temporarily grow to 125% capacity so it can start new pods before killing old ones.
*   **Alternative:** `Recreate` (Kill all -> Start all). Keeps things simple but causes downtime.

---

## 14. Triggering an Update
You can edit the YAML and `apply`, or use the CLI:
```bash
kubectl set image deployment/my-web-app nginx=nginx:1.28
```
This single command triggers the entire "Create New RS -> Scale Up/Down" dance.

---

## 15. Managing Rollouts
*   **Watch it happen:** `kubectl rollout status deployment/my-web-app`
*   **See the past:** `kubectl rollout history deployment/my-web-app`
*   **Undo the mistake:** `kubectl rollout undo deployment/my-web-app`
*   **Time travel:** `kubectl rollout undo deployment/my-web-app --to-revision=2`

---

## 16. Pausing and Resuming
Great for "Canary" testing. You can start an update, pause it to check if the new pods are okay, and then resume.
```bash
kubectl rollout pause deployment/my-web-app
# ... manual checks ...
kubectl rollout resume deployment/my-web-app
```

---

## 17. Scaling Up (and Down)
```bash
kubectl scale deployment/my-web-app --replicas=10
```
This is imperative. In a real GitOps workflow, you'd change the number in your YAML file and commit it.

---

## 18. Storage in etcd
Every resource has its place in the database.
*   `/registry/deployments/...`
*   `/registry/replicasets/...`
*   `/registry/pods/...`

The "Revision" history is stored in the ReplicaSets themselves (which are kept inactive even after they scale down to 0).

---

## 19. Wrapping Up
*   **Use Deployments** for almost everything (stateless).
*   They manage **ReplicaSets** for you.
*   They give you **Updates** and **Rollbacks** for free.
*   Using `kubectl apply` (Declarative) is better than `kubectl scale/edit` (Imperative).
