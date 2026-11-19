# How Does CoreDNS Help Pods Find Services in Kubernetes?

*Kubernetes · DevOps Interview Preparation · Question 10*

Topic: Cluster DNS and Service discovery

## What Role Does CoreDNS Play in Kubernetes Networking?

**CoreDNS** provides DNS resolution inside a Kubernetes cluster. It lets applications use stable Service names rather than relying on IP addresses that may change as cluster resources are updated.

For example, a backend application can connect to a payment Service using a DNS name such as `payments.default.svc.cluster.local`. CoreDNS resolves that name to the Service's ClusterIP, allowing the application to reach the Service without storing its IP address.

## What Happens When a Pod Looks Up a Service Name?

Consider an application Pod that needs to contact a Service named `payments` in the `default` namespace.

1. The application sends a request using `payments.default.svc.cluster.local`.
2. The Pod's configured DNS resolver queries the cluster DNS service, normally backed by CoreDNS.
3. CoreDNS looks up the Kubernetes Service DNS record.
4. For a standard ClusterIP Service, DNS returns the Service's cluster IP address.
5. The application connects to that address, and Kubernetes networking directs the traffic to an eligible backend endpoint.

The DNS lookup and the subsequent forwarding of application traffic are different steps: **CoreDNS resolves names; it does not itself load-balance the application requests.**

## How Can an Application Use a Service DNS Name?

A Pod can receive the Service hostname through an environment variable:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: checkout
spec:
  containers:
    - name: checkout
      image: nginx:stable
      env:
        - name: PAYMENTS_HOST
          value: "payments.default.svc.cluster.local"
```

This manifest illustrates how to provide a DNS hostname to an application. The application must actually use the variable for the connection; adding the variable alone does not create a request.

Within the same namespace, a Pod can often use the shorter name `payments`. Across namespaces, use a qualified name such as `payments.default` or the full Service DNS name. The cluster domain is commonly `cluster.local`, but it can be configured differently.

## How Would You Explain CoreDNS in an Interview?

**Interview-ready answer**

> CoreDNS is the DNS server commonly used in Kubernetes clusters for internal name resolution. It translates Service DNS names into the addresses applications need to connect. For example, when a Pod accesses `payments.default.svc.cluster.local`, CoreDNS can resolve that name to the payment Service's ClusterIP. This allows applications to use Service names instead of hardcoded IP addresses.

## Key Takeaways

- CoreDNS handles DNS lookups for Kubernetes Services and other supported cluster records.
- Applications can connect using readable Service hostnames instead of fixed IP addresses.
- A common Service DNS format is `<service>.<namespace>.svc.cluster.local`.
- CoreDNS resolves names; Kubernetes networking handles the traffic path to backend Pods.
- Service DNS names make application configuration easier to maintain as infrastructure changes.
