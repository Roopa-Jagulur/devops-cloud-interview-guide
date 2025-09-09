## How various components of Kubernetes interact when you run `kubectl apply` (Pod)

### Question  
What happens behind the scenes when you run `kubectl apply -f pod.yaml` to deploy a pod in Kubernetes?

### Short explanation of the question  
This question checks whether you understand the full control flow and the interaction between the Kubernetes components — from API request to pod scheduling and execution on a node.

---

### Answer  
When you run `kubectl apply -f pod.yaml`, the request is sent to the API server, which stores the desired state in etcd. The scheduler then selects a node for the pod, and the kubelet on that node pulls the image and starts the container using the container runtime.

---

### Detailed explanation of the answer for readers’ understanding

Let’s break it down step-by-step:

---

## 🧾 Step 1: `kubectl apply -f pod.yaml`

You define a Pod manifest like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: nginx
```

You run:

```bash
kubectl apply -f pod.yaml
```

This sends a REST request to the Kubernetes API server.

---

## 🔐 Step 2: API Server receives and validates the request

- The `kube-apiserver` is the front-end of the control plane.
- It authenticates and validates the request.
- If valid, it stores the desired state of the Pod object in **etcd**, the cluster’s key-value store.

---

## 🧠 Step 3: Controllers monitor etcd state

- The **scheduler** constantly watches for unscheduled pods via the API server.
- It sees that your pod has no assigned node.

---

## ⚖️ Step 4: Scheduler picks a suitable node

- It considers available resources, taints, affinities, and other rules.
- It assigns the pod to a specific node by updating the pod spec with `nodeName`.

---

## 🔧 Step 5: kubelet on the selected node takes over

- The **kubelet** on that node:
  - Sees the updated pod spec.
  - Pulls the `nginx` image from the registry.
  - Uses the **container runtime** (like containerd) to start the container.

---

## 🔌 Step 6: kube-proxy handles networking

- **kube-proxy** sets up networking rules to route traffic to the pod if it's part of a Service.
- DNS resolution (via CoreDNS) also becomes available.

---

## ✅ Step 7: Pod is now running

You can check it using:

```bash
kubectl get pods
kubectl describe pod myapp
```

---

### 🔄 Summary of Interactions
`kubectl apply -f pod.yaml`
| Component        | Role in the Flow |
|------------------|------------------|
| `kubectl`        | Sends request to Control-plane API Server|
| `kube-apiserver`-Control plane | Validates request(check user has right permissions to apply configuartions on the cluster and forwards request to 'Kube-scheduler) & stores the object in 'etcd' |
| `etcd` -Control plane| Stores desired state |
| `kube-scheduler`-Control plane  | Chooses node for pod via `kube-apiserver`.`kube-apiserver` talks to `kubelet'|
| `kubelet`- Data Plane  | Invokes `containerd` (Worker node / data plane 'COntainer Runtime'), pulls image, starts and run the container within a pod|
| `containerd` - Data Plane     | Runs the actual container |
| `kube-proxy`- Data Plane      | Sets up network rules |
| `CoreDNS` - Data Plane         | Resolves internal DNS |
| `Controller Manager`-Control plane  |For some reason if Pod goes down then there is a replica set controller which takes care of the pod and this is managed by 'Controller Manager' |

---

### 🧠 Real-world Insight

> “We faced an issue where pods remained in `Pending` state. On investigating, we found that the scheduler couldn’t place the pod due to node taints. Understanding this internal flow helped us fix the issue quickly.”

---

### Key takeaway

> "Running `kubectl apply` kicks off a coordinated flow involving the API server, etcd, scheduler, kubelet, and container runtime — all working together to ensure your pod reaches its desired state."

<img width="652" height="856" alt="image" src="https://github.com/user-attachments/assets/e936de79-291a-4762-97c3-af84ff7bff01" />
