# How Do You Roll Out a New Application Version Safely? Canary Deployment Explained

> **Kubernetes Interview Preparation — Question 9**  
> Topic: Production release strategies

## Which Deployment Strategy Would You Choose for a Production Application?

Production releases carry risk. A defect that escapes development and staging can affect many users if a new version is released to everyone at once. A **canary deployment** reduces this risk by exposing a small share of traffic to the new version first.

Two widely discussed approaches are **canary deployments** and **blue-green deployments**. In a canary release, old and new versions run together while traffic gradually shifts. In a blue-green release, traffic switches between two separate environments or versions.

## How Does a Canary Deployment Work?

A typical rollout might follow these stages:

1. Keep the existing application version serving most users.
2. Deploy the new version alongside it.
3. Send approximately **10%** of traffic to the new version and **90%** to the old version.
4. Monitor errors, latency, performance, and feedback.
5. If results are healthy, increase the new version's traffic share to **20%**, then **50%**, and eventually **100%**.
6. After the rollout is verified, retire the old version.

The percentages are illustrative, not mandatory. If the new version performs poorly, stop the rollout and direct traffic back to the previous version.

## How Can You Implement a Canary Release with Kubernetes Ingress?

One approach is to deploy separate old and new application backends and configure an Ingress controller to split traffic between them. Some controllers support canary-specific annotations or weighted routing.

For example, **ingress-nginx** supports controller-specific canary annotations such as:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
```

This snippet is **not a complete Ingress manifest**. It illustrates one controller-specific method; other Ingress controllers and Gateway API implementations use different configuration. The underlying setup still needs the relevant Deployments, Services, routing rules, and monitoring.

## What Should You Monitor During a Canary Rollout?

- Application errors and failed requests
- Response times and resource usage
- Load-test results and production health metrics
- User feedback or other signs of regressions

Traffic should only be increased when the new version meets the team's release criteria.

## What Is a Strong Interview Answer?

> For a production service, I would consider a canary deployment to reduce release risk. I would run the new version alongside the existing version and initially direct around 10% of traffic to it. After checking error rates, performance, and user impact, I would gradually increase traffic to 20%, 50%, and finally 100%. In Kubernetes, this can be implemented with an Ingress controller or another traffic-management solution that supports weighted routing. If the canary fails, I would route traffic back to the stable version.

**Interview integrity note:** Describe this as a strategy you would use unless you have actually implemented it in your organization. Do not claim experience you do not have.

## Key Takeaway

Canary deployments make production releases safer by exposing a new version gradually, observing its behavior, and retaining a path to roll back.