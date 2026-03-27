## Automated Response

>> Responding to configured, ingested or detected events by triggering actions including remediation using internal or external automation frameworks.

This was done with a Kubernetes cluster managed by ArgoCD.

The ArgoCD [Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/) documentation was great. 
Once thing to note, running workflows *did not* like the unknown host, nor the self-signed certificate. This required:
- Editing the Elasticsearch trusted hosts. Add it here: `Elastic Cloud` > `Manage` > `Actions` > `Edit Deployment` >  `Kibana` > `Edit user settings` > `workflowsExecutionEngine.http.allowedHosts:`
- Modify ArgoCD to be insecure by [disabling TLS](https://argo-cd.readthedocs.io/en/stable/operator-manual/tls/#inbound-tls-options-for-argocd-server). The first time I tested this it worked fine. The second, I ended up having to duplicate the ArgoCD API service, and having one with an insecure/no TLS that I could use inside workflows, and the other that I could use to access the ArgoCD UI. In addition to what had been deployed through the Getting Started doc, I also deployed an [insecure argo](./ArgoCD/argo.yaml).

Must have Elastic Agent installed in the cluster to get ArgoCD logs into Elastic.

### 8.1 - Demonstrate how a DevOps engineer can continuously monitor production environments to detect an unauthorized deployment and trigger a change to rollback

- Create an Out Of Sync event in ArgoCD by manually changing a deployment that is managed by ArgoCD (scaling deployment).
- Go to `Streams` > `Logs` (I had partitioned out ArgoCD logs) > `Significant Events`.
- Ask the [AI Agent](./Workflows%20and%20Agents/argocd-sig-event-investigator.json) about the significant event.
- It will ask you if you want to rollback, which will use [this workflow](./Workflows%20and%20Agents/Unauthorized%20Deployments%20-%20Update%20outside%20of%20ArgoCD.yaml).

### 8.2 - Show how the platform automates incident response by integrating with collaboration and service management tools, expediting triage, resolution, and communication beyond simple ticket creation.

- Container restarts rule/alert based on the metrics query: `resource.attributes.k8s.container.status.last_terminated_reason: "Error" or resource.attributes.k8s.container.status.last_terminated_reason:"OOMKilled"`
- Manually run the [Container Restarts](./Workflows%20and%20Agents/Container%20Restarts%20Case%20Collaboration%20Workflow.yaml) workflow which will take the (cart service) alert input and create a case, etc.
- It will call the agent defined [here](./Workflows%20and%20Agents/argocd-sig-event-investigator.json)
- It will also send to a slack workspace. Create your own Slack workspace and update the connector.

### 8.3 - Show how the platform can automatically initiate a remediation via integration to an orchestration or infrastructure as code platforms such as Ansible or Terraform.

- This uses the [Unauthorized Deployment - Day of the Week](./Workflows%20and%20Agents/Unauthorized%20Deployments%20-%20Day%20of%20the%20Week.yaml) workflow, which triggers based on an alert:
- Alert query: `kibana.alert.rule.name: "Deployment Out of Sync" or kibana.alert.rule.name: "New Deployment"` 
- This triggers a rollback to a prevision revision or a deletion of the app, depending on what you did in ArgoCD and when the app was deployed.