# Backup Controller base configuration

This repository offers a configuration for a Backup Controller that is built on top of the Cluster-as-a-Service configuration.

[Velero](https://velero.io/) is a widely used open-source solution in the Kubernetes ecosystem for performing backup and restore operations on Kubernetes cluster resources. It is strongly recommended to incorporate Velero into any disaster recovery plan for your Crossplane control planes. This guide provides best practices for using Velero in the context of Crossplane.

[Disaster Recovery When Using Crossplane for Infrastructure Provision on AWS](https://aws.amazon.com/blogs/opensource/disaster-recovery-when-using-crossplane-for-infrastructure-provisioning-on-aws/) in this blog post, we discuss different backup considerations when employing GitOps and the use of Kubernetes-native infrastructure provisioning tools on Amazon Elastic Kubernetes Service (Amazon EKS).

## In the Context of Crossplane

It's important to understand that disaster recovery for Crossplane does not equate to disaster recovery for your business-critical state, such as database or cloud storage data. Here are key points to keep in mind:

1. Crossplane ensures the configuration of your resources.
2. Crossplane does not concern itself with the internal state of your resources.

Crossplane enforces two primary aspects:

1. Ensuring that the declared resources exist.
2. Ensuring that the configuration of those resources matches what you declared. If there is a mismatch, Crossplane attempts to reconcile it to the desired state.

Crossplane does not delve into the internal state of the resource.

Likewise, if your control plane fails and goes offline, it does not necessarily mean that the resources managed by that control plane also fail. It simply means that the configuration of those resources stops reconciling while the control plane is offline, which could result in configuration drift.

It is advisable to configure Velero to store captured backup states for all your control planes in a single global bucket or blob.

For each control plane you create, this configuration creates three Velero schedules with the following intervals:

- Hourly
- Daily
- Monthly

You can adjust the backup frequency based on your platform's Recovery Point Objective (RPO). If you need to restore state more frequently, you must take more frequent snapshots.

## Restoring the Control Plane from State

To restore the backup state into a new cluster, follow these steps:

1. Ensure that Crossplane and providers are not running in the new cluster. Otherwise, the order of managed resources, composites, and claims becomes critical. Failure to do this may result in race conditions and, for example, duplication of managed resources.
2. Install Velero and configure it to use the backup data source.
3. Restore the backed-up resources to the new cluster.
4. Scale up the provider deployments to one in the new cluster.
5. During the backup and restoration process, Velero preserves owner references, resource states, and external names of managed resources. The reconciliation process should proceed as if there were no changes.

## Contributing

If you encounter issues or want to request improvements, review the
[Contributing Guides](https://docs.crossplane.io/contribute/).
