---
date: '2025-02-28T15:49:14+11:00'
draft: false
title: 'Homelab Machine Learning'
tags: ["ml stack", "mlops"]
categories: ["machine-learning", "deployment", "k8s@home"]
---
# ml@home

ML tooling is in a really weird place right now. At $DAYJOB we use Google Cloud's Vertex AI tools which works just okay. It does the job but it doesn't really feel cohesive? So after setting up a homelab kubernetes cluster, I explored the open-source ML tooling landscape. Here are some initial notes.

## Seldon Core

For ML serving. Deployed but unused right now.

## Feast

A feature store. Could be useful for low latency inference with tabular models like recommender systems. Haven't used it yet. The helm chart didn't work for me, I had to point kustomize at an `install.yaml` to deploy the operator.

## Kubeflow (DeployKF)

Undeployed eventually. Not fun to deploy, MySQL and MariaDB operators don't play nice on home grown clusters which are prone to downtime. DeployKF in particular was very inflexible to deploy, it _requires_ ArgoCD but I use FluxCD.

## Flyte

For data and ML workflows. It doesn't seem to work very well, and has weird design with a `flytectl` tool used (how do I CI/CD my pipelines/workflows) and setting up custom tooling in CI is never fun. Scrapping in favour of...

## Prefect

For data and ML workflows

## Kafka

For event-driven flows with prefect. No topics right now.

## Kuberay

For distributed, parallel data processing and model processing. Very flexible and nice integration with other tools in this stack.

## MLFlow

The only good open source model registry and experiment tracker.

## Iceberg

This was not that fun to deploy and usually seems to be part of larger data lake platforms. Eventually undeployed since I'd rather just use dask + s3fs.

## Minio

S3 compatible object storage. Most other services rely on this. I use a NFS backed one on my NAS.

## Cloudnative PG

PostgreSQL operator, battle-tested.
