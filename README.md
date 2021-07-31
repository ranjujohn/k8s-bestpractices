# k8s Security & Best Practices

This is prepared based on a standard Kubeadm cluster. 

## Ops/Cluster Operator
  - Enable imit range/resource quota
    - Creation of these defaults can be managed by [Kyverno](https://kyverno.io/policies/best-practices/add_ns_quota/?policytypes=LimitRange)
  - Implement POD Security Standars
    - Details can be found [here](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
    - Can be implemented using [Kyverno](https://kyverno.io/policies/pod-security/) or [Opa Gatekeeper](https://github.com/open-policy-agent/gatekeeper). You can read about OPA gatekeeper [here](https://kubernetes.io/blog/2019/08/06/opa-gatekeeper-policy-and-governance-for-kubernetes/)
  - Enable default network policy
    - By default communication between all pods in a cluster is allowed
    - In a multi tenant/team env, alteast enable NS isolation using default policies. [Examples](https://github.com/ahmetb/kubernetes-network-policy-recipes)
    - Can also have global default policies based on the needs. Eg: [Deny all except external DNS](https://docs.projectcalico.org/security/kubernetes-default-deny)
    - Network policy can be implemened by CNIs like Calico and Cilium.
    - Additional to the pods/workloads, CNIs like Calico and Cilium can manage [host endpoints](https://docs.projectcalico.org/security/kubernetes-nodes) too. 
    - Creation of these defaults can be manged by [Kyverno](https://kyverno.io/docs/writing-policies/generate/#generate-a-networkpolicy)
  - [Enable Audit logging](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/) 
  - [Enable secret encryption at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
  - Use RBAC to manage the [authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)
  - Use OpenID (OIDC) tokens as a user authentication strategy
  - Minimise access to secrets using RBAC
  - Restrict control plane scheduling (Default allowd on a Kubeadm Cluster) - Kyverno can do this
  - Allow only known container registries - Kyverno or OPA can be used for this
  - NodeLocal DNScache - https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/
  - Set image pull policy to Always in a Multi tenant environment. Can be enabled as an adimission controller using API flag
  - default-not-ready/unreachable-toleration-seconds
  - Security Benchmark
    - https://www.cisecurity.org/benchmark/kubernetes/
    - https://github.com/aquasecurity/kube-bench
    

## Dev/Cluster Users
  - Image 
    - single process  
      - https://www.tutorialworks.com/containers-single-or-multiple-processes/
    - Tini or "shareProcessNamespace" option in K8s
      - https://github.com/krallin/tini
      - https://www.back2code.me/2020/02/zombie-processes-back-in-k8s/
      - https://ahmet.im/blog/minimal-init-process-for-containers/
    - Signal Handling
      - https://tasdikrahman.me/2019/04/24/handling-singals-for-applications-in-kubernetes-docker/
    - Reduce the image size as it would even affect the pull time. [Distroless from Google](https://github.com/GoogleContainerTools/distroless) is a nice option as the base image
  - Output logs to stdout and stderr
  - Using Probes 
    - Liveness probe can be avoided if the app can crash on error
    - Readiness probe is executed throughout the pod's lifetime failing which the pod will be removed from endpoint list
    - Make sure probes are not depend on external dependencies
    - Startup Probe can be used with legacy applications that might require an additional startup time
    - Nice links about Probes
      - [Probes](https://blog.colinbreck.com/kubernetes-liveness-and-readiness-probes-how-to-avoid-shooting-yourself-in-the-foot/)
      - [Probes are dangerous](https://srcco.de/posts/kubernetes-liveness-probes-are-dangerous.html)
      - [Another one](https://loft.sh/blog/kubernetes-liveness-probes-examples-common-pitfalls/index-1/)
  - Resources (Cpu/Memory)
    - Use memory requests and limit same
    - CPU limit is implemented using CFS. Limit can cause throttling for the app
    - Requests is used for scheduling and the resources will be blocked even if not used
    - Some links
      - https://sysdig.com/blog/kubernetes-limits-requests/
      - https://medium.com/omio-engineering/cpu-limits-and-aggressive-throttling-in-kubernetes-c5b20bd8a718
      - https://www.youtube.com/watch?v=eBChCFD9hfs
  - QOS Policy
    - Guaranteed
    - Burstable
    - BestEffort
  - Prestop hook & Gracefultermination seconds
    - https://learnk8s.io/graceful-shutdown
  - Avoid using latest tags as a pod re-schedule might pull a new image
  - Use Pod disruption budget to limits the number of Pods of a replicated application that are down simultaneously from voluntary disruptions

## Good to know
  - Init containers
    - Init containers can contain utilities or custom code for setup that are not present in an app image
    - Another usecase can be to clone a Git repository 
  - StatefulSet
    - Statefulset pod will not be migrated to a new node on the node failure like a Deployment
    - https://medium.com/tailwinds-navigator/kubernetes-tip-how-statefulsets-behave-differently-than-deployments-when-node-fails-d29e36bca7d5

## General useful links
  - https://learnk8s.io/production-best-practices
  - https://cloud.google.com/architecture/best-practices-for-building-containers
  - https://cloud.google.com/architecture/best-practices-for-operating-containers
  - https://pracucci.com/kubernetes-dns-resolution-ndots-options-and-why-it-may-affect-application-performances.html
