# How Are Liveness and Readiness Probes Different in Kubernetes?

*Kubernetes · DevOps Interview Preparation · Question 12*

Topic: Container health checks and application availability

## What Is the Difference Between a Liveness Probe and a Readiness Probe?

A **liveness probe** checks whether a container is still functioning and should continue running. If its liveness probe repeatedly fails, the kubelet restarts that container according to the Pod's restart policy.

A **readiness probe** checks whether a container is prepared to handle requests. If readiness fails, Kubernetes marks the Pod as not ready and removes it from the ready endpoints used by matching Services; a readiness failure alone does **not** restart the container.

| Probe | Main question | What happens when it fails? |
|---|---|---|
| Liveness | Is the container functioning? | Kubelet may restart the container after the configured failure threshold. |
| Readiness | Can the application serve traffic now? | Pod becomes unready and is excluded from ready Service endpoints. |

## When Would a Liveness Probe Be Helpful?

Imagine an application process that is still running but has become stuck and cannot recover by itself. A liveness check against a health endpoint, such as `/health`, can detect the problem.

If the configured probe fails enough times, Kubernetes restarts the container. This can restore service without requiring an engineer to manually restart every affected workload. The health endpoint should measure whether a restart is appropriate, not simply whether every dependency is reachable.

## Why Might an Application Be Alive but Not Ready?

A backend process may start successfully but still be unable to serve requests because it is initializing or waiting for an important dependency, such as a database.

In that situation:

- Its **liveness** check can succeed because the process is working.
- Its **readiness** check can fail because it cannot yet handle real requests.
- Kubernetes keeps the Pod out of ready Service endpoints until the readiness check succeeds.

This distinction prevents a running-but-unavailable application from receiving normal Service traffic.

## How Do You Configure Both Probes in a Pod?

The following example assumes the application exposes `/health` and `/ready` on port `8080`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  containers:
    - name: backend
      image: example.com/backend:v1
      ports:
        - containerPort: 8080
      livenessProbe:
        httpGet:
          path: /health
          port: 8080
        periodSeconds: 5
        failureThreshold: 3
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        periodSeconds: 5
        failureThreshold: 3
```

The image and endpoints are illustrative; replace them with your application's real image and health routes. For slow-starting applications, a **startup probe** can prevent liveness checks from restarting the container before initialization finishes.

## How Would You Answer This in a DevOps Interview?

**Interview-ready answer**

> Liveness and readiness probes solve different problems in Kubernetes. A liveness probe checks whether a container is healthy enough to keep running; repeated failures can cause Kubernetes to restart it. A readiness probe checks whether the application can accept traffic. If readiness fails, Kubernetes stops treating the Pod as a ready Service endpoint, but does not restart it solely because of that failure. For example, a backend may be alive while waiting for its database and therefore not yet be ready.

## Key Takeaways

- Liveness probes detect containers that need restarting.
- Readiness probes determine whether an application should receive Service traffic.
- A container can be alive while its Pod is not ready.
- Readiness failures do not automatically restart containers.
- Use application-appropriate health endpoints and consider startup probes for slow initialization.
