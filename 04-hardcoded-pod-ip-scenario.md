# A Kubernetes Networking Mistake: Why Hardcoded Pod IPs Can Break Your Application

*DevOps Interview Preparation | Question 4*

A frontend may communicate with a backend successfully in a test cluster, yet still be configured in a way that is fragile in staging or production. One common example is putting a backend Pod's IP address directly into the frontend's environment variables.

## 4. A Developer Wants to Use a Fixed Backend Pod IP in Kubernetes. How Would You Explain the Risks and Recommend a Better Solution?

**Scenario:** A developer runs a frontend in **Pod A** and a backend in **Pod B**. The frontend's environment variable contains Pod B's IP address. The developer wants to keep this approach when moving to staging and does not want to create a Service.

The key concern is that **Pod IP addresses are not stable identities for applications**. A replacement Pod can receive a new address, leaving the frontend pointing at an old destination.

### 4.1. What Could Go Wrong When a Frontend Application Connects Directly to a Backend Pod's IP Address?

Suppose the frontend uses this environment variable:

```yaml
env:
  - name: BACKEND_URL
    value: "http://10.244.1.15:8080"
```

If the backend is later replaced at `10.244.2.20`, the frontend still attempts to contact `10.244.1.15`. Requests may fail, potentially appearing as gateway errors such as HTTP 502 depending on the application's architecture.

### 4.2. Why Can Pod Restarts and Replacements Disrupt Communication Between Applications?

Pods can become unavailable because of application failures, resource pressure, node problems, deletion, or rollout changes. A replacement Pod may receive a different IP address.

A container restarting **inside the same Pod** does not necessarily change the Pod IP. The important risk is relying on an IP that may change when the Pod itself is replaced.

### 4.3. How Can Kubernetes Services Prevent Applications from Depending on Changing Pod IP Addresses?

Create a Service for the backend and configure the frontend to call the Service by name:

```yaml
env:
  - name: BACKEND_URL
    value: "http://backend-service:8080"
```

Kubernetes keeps the Service endpoint stable while updating its eligible backend endpoints as Pods are added or replaced. The frontend does not need to be reconfigured each time the backend's Pod IP changes.

### 4.4. How Do Labels and Selectors Help a Service Find the Correct Backend Pods?

Backend Pods can be labeled:

```yaml
metadata:
  labels:
    app: backend
```

The backend Service can use the matching selector:

```yaml
spec:
  selector:
    app: backend
```

This tells Kubernetes which Pods should be considered as backend endpoints for the Service. Replacement Pods with the correct label can join the Service's endpoint set when eligible.

### 4.5. How Would You Persuade a Developer to Replace Hardcoded Pod IP Addresses with Kubernetes Services?

I would focus on the operational impact rather than just calling the design incorrect:

- **Fragility:** A Pod replacement may invalidate the stored IP.
- **Maintenance:** Every address change could require a frontend configuration update.
- **Better design:** A Service provides a consistent name while Kubernetes tracks eligible backend Pods.

This is especially valuable in staging and production, where deployments and Pod replacements are routine.

### 4.6. How Should You Respond to This Kubernetes Networking Scenario in a DevOps Interview?

> I would explain that a Pod IP is not a dependable long-term address because Pods can be deleted or replaced and the replacement may receive a different IP. If the frontend hardcodes the backend Pod's address, requests can fail after such a change. I would recommend a Kubernetes Service for the backend and configure the frontend to use its Service DNS name. The Service selector identifies the appropriate backend Pods, and Kubernetes updates the eligible endpoints as Pods change. This reduces configuration maintenance and makes communication more reliable, although it cannot prevent every outage.

### 4.7. What Are the Practical Differences Between Using a Pod IP Address and a Kubernetes Service?

| Direct Pod IP | Kubernetes Service |
|---|---|
| Depends on one Pod's address | Provides a stable address or DNS name |
| Can break after Pod replacement | Can target eligible replacement Pods |
| Requires tracking IP changes | Kubernetes maintains endpoint information |
| Fragile for changing workloads | Better suited to dynamic applications |

**Quick revision:** Do not hardcode a backend Pod IP in a frontend configuration. Use a Service name so that Pod replacement does not require updating the frontend's destination.