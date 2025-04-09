# ClusterLogForwarder Configuration in OpenShift

This document describes the process of configuring ClusterLogForwarder to forward OpenShift logs to Loki.

## Prerequisites

The following operators must be installed:
- ODF Operator
- OpenShift Logging Operator
- Loki Operator
- Cluster Observability Operator

## Configuration

### 1. Storage Configuration

First, we create an ObjectBucketClaim to store the logs:
- Creates a bucket in ODF/NooBaa
- Automatically generates access credentials

To create the ObjectBucketClaim:
```
oc apply -f bucket-claim.yaml
```

Set up access variables:
```
AWS_ACCESS_KEY_ID="$(oc extract -n openshift-logging secret/obc-loki --keys='AWS_ACCESS_KEY_ID' --to=-)"
AWS_SECRET_ACCESS_KEY="$(oc extract -n openshift-logging secret/obc-loki --keys='AWS_SECRET_ACCESS_KEY' --to=-)"
BUCKET_NAME="$(oc get objectbucketclaim obc-loki -n openshift-logging -o jsonpath='{.spec.bucketName}')"
```

### 2. Loki Configuration

- Creation of a secret with storage credentials
- LokiStack deployment
- UI Plugin configuration for log visualization

### 3. Collector Configuration

Create dedicated ServiceAccount:
```
oc create sa collector -n openshift-logging
```

Assign necessary roles:
```
oc adm policy add-cluster-role-to-user logging-collector-logs-writer -z collector -n openshift-logging
oc adm policy add-cluster-role-to-user collect-application-logs -z collector -n openshift-logging
oc adm policy add-cluster-role-to-user collect-audit-logs -z collector -n openshift-logging
oc adm policy add-cluster-role-to-user collect-infrastructure-logs -z collector -n openshift-logging
```

## Usage

After configuration, logs can be viewed through the OpenShift Console in the Observability section.