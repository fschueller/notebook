# Kubernetes Security Best Practices
##### Notes for CNCF Webinar, 11.03.2020

## Kubernetes Hygiene

- Upgrade to the current version (*always*!)
  - only the last 3 versions are officially supported
  - `kubeadm upgrade apply <version>`
  - avoid sticky CVEs
  - rolling update hygiene saves you pain
  - patches are backported but stay in the windows

- Harden node security
  - control network access
  - restrict ports over network
  - minimal admin access to nodes (very traditional)
  - access to Docker? Access to K8s!

- Enable RBAC
  - control API access through permissions
  - subjects, API resources, operations
  - **regularly audit this**
    - have an auditing process in place
    - only then come the tools!

## Workload best Practices

- Contextualize Risk
  - image? library versions? libraries?
  - cluster? Where does it run?
  - privileges? which users/processes?
  - network config? where's the LB?!
  - keys, secrets?
  - **all of these amplify by stackin onto each other!!**
- Leverage namespaces
  - break down workloads
  - better segmentation for RBAC
  - easier on the `kubectl`
- Leverage network policies
  - pod-centric firewalling
  - policies on ingress/egress
  - helps with multi-tenancy
  - visualize your network policies
- Slim down your images
  - lightweight/distro-less
  - avoid network utilities
  - no chmod/chown
  - look at ephemeral containers
  - read-only filesystems
    - helps against dropping cryptominers
    - very simple technique and helps against lots
    - `docker diff` your stuff

**-> DON'T ROLL YOUR OWN SECRETS MANAGEMENT!**

-> Shape after your organization and apps, there is no cookie-cutter

## Summary

> Security is hard.

- start with low-hanging fruits
- anything will contribute.
