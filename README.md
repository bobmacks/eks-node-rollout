# eks-node-rollout
![Docker Cloud Automated build](https://img.shields.io/docker/cloud/automated/cmdlabs/eks-node-rollout) ![Docker Cloud Build Status](https://img.shields.io/docker/cloud/build/cmdlabs/eks-node-rollout) ![Docker Pulls](https://img.shields.io/docker/pulls/cmdlabs/eks-node-rollout) ![Docker Stars](https://img.shields.io/docker/stars/cmdlabs/eks-node-rollout)

eks-node-rollout is a CLI tool that assists with rolling updates to your EKS workers.

https://hub.docker.com/r/cmdlabs/eks-node-rollout

## Justification
When updating an Auto Scaling Group with a tool like Terraform, no functionality exists to perform a rolling update to the ASG.

This poses a problem in automated deployments (especially CI/CD pipelines): you will deploy updates, but not receieve any feedback on whether or not those changes actually worked. Only by manually terminating instances or scaling up/down can you find out if your changes are working. Performing blue/green deployments of ASGs does not help either, as you have no way of knowing when the workers in the new ASG have succesfully launched.

## Overview
These are roughly the actions taken by the tool:

  1. Get all ASGs attached to cluster
  2. Suspend cluster-autoscaler on ASG (if present)
  3. Add an EC2 instance to the ASG
  4. Wait for the new instance to be healthy
  5. Drain an instance that is outdated
  6. Terminate the outdated instance
  7. Wait for all pods to be ready in new instance
  8. Repeat until all instances are up to date

## Usage
```
Usage: eks-node-rollout [OPTIONS]

  Retrieve all outdated workers and perform a rolling update on them.

Options:
  --cluster-name TEXT       Cluster name to discover ASGs from  [required]
  --drain-timeout TEXT      Timeout for draining worker nodes
  --dry-run / --no-dry-run  Run with read-only API calls
  --debug / --no-debug      Enable debug logging
  --help                    Show this message and exit.
```

The tool also accepts environment variables with the prefix `EKS_NODE_ROLLOUT_*` e.g. `EKS_NODE_ROLLOUT_ASG_NAME`.

## Docker Image
This tool is available as a Docker image: `cmdlabs/eks-node-rollout:0.2.1`

docker-compose.yml:
```yml
version: '3.7'
services:
  eks-node-rollout:
    image: cmdlabs/eks-node-rollout:0.2.1
    env_file: .env
    volumes:
      - ~/.kube:/root/.kube
      - ~/.aws:/root/.aws
```

Update your `.env` file with the following:

```
EKS_NODE_ROLLOUT_CLUSTER_NAME
EKS_NODE_ROLLOUT_DRY_RUN
AWS_PROFILE=my-named-profile (optional)
```

Then it can be called as such:

```bash
docker-compose run --rm eks-node-rollout eks-node-rollout --cluster-name=my-eks-cluster
```
