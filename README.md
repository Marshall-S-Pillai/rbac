# rbac
### KUBERNETES LAB – 3 HANDS ON: NETWORK POLICY IN KUBERNETES

---

### **1. Create Network Policies for App1 and App2 Pods**
#### **Objective:**
Allow traffic from pod `app1` to pod `app2`.

#### **Steps:**
1. **Create Pod `app1`:**
   - Define `app1.yml` manifest file.
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: app1
     labels:
       app: app1
   spec:
     containers:
     - name: app1-container
       image: nginx
   ```
   Apply using `kubectl apply -f app1.yml`.

2. **Create Pod `app2`:**
   - Define `app2.yml` manifest file.
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: app2
     labels:
       app: app2
   spec:
     containers:
     - name: app2-container
       image: nginx
   ```
   Apply using `kubectl apply -f app2.yml`.

3. **Create Network Policy:**
   - Define `app-network-policy.yml` to allow traffic from `app1` to `app2`.
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: app1-to-app2
   spec:
     podSelector:
       matchLabels:
         app: app2
     ingress:
     - from:
       - podSelector:
           matchLabels:
             app: app1
   ```
   Apply using `kubectl apply -f app-network-policy.yml`.

4. **Validate Network Policy:**
   - Test communication between Pods using `kubectl exec`:
     ```bash
     kubectl exec -it app1 -- curl http://app2
     ```

---

### **2. Create Network Policies for Frontend and Backend Pods**
#### **Objective:**
Allow communication from Pods labeled `role=frontend` to Pods labeled `role=backend`, deny all other traffic.

#### **Steps:**
1. **Create Pods `frontend.yml` & `backend.yml`:**
   - `frontend.yml`:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: frontend
     labels:
       role: frontend
   spec:
     containers:
     - name: frontend-container
       image: nginx
   ```
   - `backend.yml`:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: backend
     labels:
       role: backend
   spec:
     containers:
     - name: backend-container
       image: nginx
   ```
   Apply using `kubectl apply -f frontend.yml` and `kubectl apply -f backend.yml`.

2. **Create Network Policy:**
   - Define `frontend-to-backend-policy.yml` to allow traffic only from Pods with label `role=frontend` to Pods with label `role=backend`.
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: frontend-to-backend
   spec:
     podSelector:
       matchLabels:
         role: backend
     ingress:
     - from:
       - podSelector:
           matchLabels:
             role: frontend
   ```
   Apply using `kubectl apply -f frontend-to-backend-policy.yml`.

3. **Validate Network Policy:**
   - Test allowed communication:
     ```bash
     kubectl exec -it frontend -- curl http://backend
     ```
   - Test denied communication from other Pods (e.g., admin or database).

#### **Explanation:**
- **Pod Selector:** Policy applies to Pods labeled with `role: backend`.
- **Ingress Rule:** Traffic allowed only from Pods with `role: frontend`.
- **Effect:** Blocks all other traffic by default.

---

### **3. Pod Troubleshooting Steps**
#### **When Unable to Communicate:**
1. **Check Pod Status:**
   ```bash
   kubectl get pods
   ```
2. **Inspect Logs:**
   ```bash
   kubectl logs <pod-name>
   ```
3. **Verify Network Policies:**
   ```bash
   kubectl get networkpolicies -n <namespace>
   ```
4. **Test DNS Resolution:**
   ```bash
   kubectl exec -it <pod-name> -- nslookup <service-name>
   ```
5. **Ping Pods:**
   ```bash
   kubectl exec -it <pod-name> -- ping <destination-pod-ip>
   ```
6. **Test Connectivity with Curl:**
   ```bash
   kubectl exec -it <pod-name> -- curl http://<destination-pod-ip>:<port>
   ```
7. **Check Network Interface:**
   ```bash
   kubectl exec -it <pod-name> -- ip addr
   ```
8. **Inspect CNI Configuration:**
   Check `kube-system` namespace for CNI plugin issues.
9. **Review Firewall/Security Groups:**
   Verify external firewalls or cloud security groups.
10. **Verify Service Configuration:**
    ```bash
    kubectl get svc <service-name> -o yaml
    ```
11. **Check Resource Limits:** Ensure Pods have sufficient resources.
12. **Inspect PodSecurityPolicy:** Verify PodSecurityPolicies.
13. **Check Node Network:** Confirm node-level networking.
14. **Restart/Recreate Pods:**
    ```bash
    kubectl delete pod <pod-name>
    ```

---

### **4. Implementing RBAC in Kubernetes**
#### **Objective:**
Create a Role `read-only` and bind it to user `jane_doe`.

#### **Steps:**
1. **Create Role Manifest:**
   - `read-only-role.yml`:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     namespace: development
     name: read-only
   rules:
   - apiGroups: [""]
     resources: ["pods"]
     verbs: ["get", "list"]
   ```
   Apply using `kubectl apply -f read-only-role.yml`.

2. **Create RoleBinding Manifest:**
   - `role-binding.yml`:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: read-only-binding
     namespace: development
   subjects:
   - kind: User
     name: jane_doe
     apiGroup: rbac.authorization.k8s.io
   roleRef:
     kind: Role
     name: read-only
     apiGroup: rbac.authorization.k8s.io
   ```
   Apply using `kubectl apply -f role-binding.yml`.

3. **Verify Role Binding:**
   - Check permissions by switching to `jane_doe` and testing access:
     ```bash
     kubectl auth can-i list pods -n development
     ```

---

