# Understanding ReplicaSets & Pods

## 1. Cluster Setup Walkthrough
Before diving into controllers, we need a working environment. Here is a recap of setting up a minimal cluster with one master and one worker.

### A. Accessing the Server (SSH)
To manage your cloud VM, you need to log in remotely.
```bash
ssh -i key.pem ubuntu@<Public-IP>
```
*   **Tip:** Always ensure your `.pem` key has the correct permissions (chmod 400).

### B. Bootstrapping the Master
Run the following on the machine you want to be the Control Plane:
```bash
kubeadm init
```
This single command does a lot: generates certificates, starts the API Server, and gives you a `kubeadm join` token.

**Lost the token?** No problem. Generate a new one:
```bash
kubeadm token create --print-join-command
```

### C. The Missing Piece: Networking (CNI)
After initializing, you might notice your nodes are `NotReady`. This is because they can't talk to each other yet. You need a **CNI (Container Network Interface)** plugin like Weave or Flannel.
```bash
kubectl apply -f https://.../weave-daemonset.yaml
```
Once this is installed, the CoreDNS pods will start, and your node status will turn to `Ready`.

---

## 2. Launching Your First Pod
Let's spin up a simple Nginx server.
```bash
kubectl run scalerdemo --image=nginx
```
*   **Note:** In newer versions of Kubernetes, `kubectl run` creates a Pod directly. To be absolutely safe and avoid creating a Deployment by accident, you can add `--restart=Never`.

### Where is it running?
To see which node picked up your Pod:
```bash
kubectl get po -o wide
```
The `-o wide` flag is your best friend here—it shows the Node Name and the Pod IP.

---

## 3. Deleting (and Restoring) Pods
If you delete a standalone Pod:
```bash
kubectl delete po --all
```
...it is gone forever. Kubernetes won't bring it back because a bare Pod has no "controller" watching over it. This is why we need higher-level abstractions like **ReplicaSets**.

---

## 4. Under the Hood: Container Runtime
On the *Worker Node*, you can actually see the containers running if you look 'under' Kubernetes.
```bash
crictl ps
# or
ps -ef | grep runc
```
This confirms that the Kubelet is talking to the Container Runtime (like containerd or CRI-O) to do the actual heavy lifting.

---

## 5. Anatomy of a Pod YAML
Imperative commands (`kubectl run`) are good for testing, but declarative YAML is for pros.
Here is the skeleton of a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: web
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
```

### Quick YAML Refresher
*   **Indentation matters:** Use 2 spaces.
*   **Lists:** Start with a `-` (dash).
*   **Pairs:** `key: value`.

---

## 6. ReplicaSets Explained
A **ReplicaSet (RS)** is a controller with a single job: **Maintain the Count**.
If you tell it "I want 3 Nginx pods," it ensures there are exactly 3.
*   If one crashes? It starts a new one.
*   If you delete one manually? It starts a new one.
*   If you accidentally start 5? It kills 2.

---

## 7. Defining a ReplicaSet
The YAML for a ReplicaSet looks like a Pod YAML wrapped in some management logic.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-replicaset
spec:
  replicas: 3
  selector:
    matchLabels: 
      app: web    # <--- Critical Link
  template:       # <--- The Pod Blueprint
    metadata:
      labels:
        app: web  # <--- Must match the selector above!
    spec:
      containers:
        - name: nginx
          image: nginx
```
**The "Selector" Trap:** The `selector` tells the RS which Pods it owns. The `template` describes the new Pods it creates. These labels **must match**, or the RS will be confused.

---

## 8. The Life of a ReplicaSet Request
When you run `kubectl apply`, a fascinating chain of events kicks off:
1.  **You -> API Server:** You send the manifest.
2.  **API Server -> etcd:** The configuration is saved to the cluster database.
3.  **Controller Manager:** The ReplicaSet Controller process notices a new event. It checks the DB: "User wants 3 pods. I see 0."
4.  **Creation:** The Controller creates 3 Pod objects in the API.
5.  **Scheduler:** Noticed 3 new "Pending" pods. Assigns them to nodes.
6.  **Kubelet:** Sees a pod assigned to its node. Starts the docker container.

---

## 9. Failure Recovery in Action
*   **Pod Failure:** If the Nginx process dies, the Kubelet restarts it locally. The RS doesn't even need to intervene.
*   **Node Failure:** If the whole server smokes, the Kubelet dies. The RS notices the Pods are "Unknown" or gone, and spins up replacements on a healthy node.

---

## 10. Deleting ReplicaSets (Cascading vs Orphan)
When you delete a ReplicaSet, you have a choice:
1.  **Cascade (Default):** Delete the RS *and* all its Pods.
    ```bash
    kubectl delete rs web-replicaset
    ```
2.  **Orphan:** Delete the RS, but leave the Pods running (unmanaged).
    ```bash
    kubectl delete rs web-replicaset --cascade=orphan
    ```

---

## 11. Troubleshooting 101
*   **Pending?** The scheduler can't find a place for your pod. Check `kubectl describe pod`. Do you have enough RAM? Is the node Tainted?
*   **ImagePullBackOff?** You probably made a typo in the image name, or you don't have permission to pull it.
*   **CrashLoopBackOff?** The app is starting but immediately dying. Check logs: `kubectl logs <podname>`.

---

## 12. Final Takeaways
1.  **Pods are ephemeral.** Don't rely on them lasting forever.
2.  **ReplicaSets guarantee availability.** They are the self-healing engine.
3.  **Labels are the glue.** Selectors and Labels bind the RS to its Pods.
