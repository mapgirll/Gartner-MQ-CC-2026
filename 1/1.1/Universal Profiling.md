## Add Universal Profiling to a Kubernetes Cluster

*I* had trouble getting this set upn initial so here is what I did.

In a **Cloud Hosted** version of Elastic, go to `Infrastrucutre` > `Universal Profiling` > `Add Data`.
This will open up the instructions to install the Universal Profiling Agent.

Now, iirc, this didn't work out-of-the-box for me and I had to edit the values/config to find the commented out sections for universal profiling/eBPF. It might have been something like this: [Deploy Elastic Agent using Kubernetes with the Universal Profiling Agent integration](https://www.elastic.co/docs/solutions/observability/infra-and-hosts/get-started-with-universal-profiling#deploy-agent-using-kubernetes-with-the-universal-profiling-agent-integration)

In the elastic-agent-values.yaml look for the line `# For Universal Profiling (eBPF profiler). For cloud-defend only, use runAsUser + BPF/PERFMON/SYS_RESOURCE instead.`

In the securityContext.yaml look for the line `# The following capabilities are needed for Universal Profiling.`

In the end I think I used the `Elastic Agent Integration` and added the `Universal Profiling Agent` integration to my Elastic Agent running in my cluster.