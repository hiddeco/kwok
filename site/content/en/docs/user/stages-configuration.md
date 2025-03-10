# Stages Configuration

{{< hint "info" >}}

This document walks you through how to configure the Stages of Lifecycle

{{< /hint >}}

## What is a Stage?

The Stage API is a [`kwok` Configuration](./configuration) that allows users to define and simulate different stages in the lifecycle of Kubernetes resources, such as nodes and pods.
Each Stage resource specifies a resourceRef field that identifies the type of resource that the stage applies to, and a selector field that determines when the stage should be executed.

A Stage resource has the following fields:

``` yaml
kind: Stage
apiVersion: kwok.x-k8s.io/v1alpha1
metadata:
  name: <string>
spec:
  resourceRef:
    apiGroup: <string>
    kind: <string>
  selector:
    matchLabels:
      <string>: <string>
    matchAnnotations:
      <string>: <string>
    matchExpressions:
      - key: <expressions-string>
        operator: <string>
        values:
          - <string>
  delay:
    durationMilliseconds: <int>
    durationFrom:
      expressionFrom: <expressions-string>
    jitterDurationMilliseconds: <int>
    jitterDurationFrom:
      expressionFrom: <expressions-string>
  next:
    statusTemplate: <string>
    finalizers:
      add:
        - value: <string>
      remove:
        - value: <string>
      empty: <bool>
    delete: <bool>
```

By setting the `selector` and `next` fields in the spec section of a Stage resource,
users can specify the conditions that need to be met for the stage to be applied,
and the changes that will be made to the resource when the stage is applied.
The `next` field allows users to define the new state of the resource using the `statusTemplate` field,
modify the `finalizers` of the resource, and even `delete` the resource.

Additionally, the `delay` field in a Stage resource allows users to specify a delay before the stage is applied,
and introduce jitter to the delay to specify the latest delay time to make the simulation more realistic.
This can be useful for simulating real-world scenarios where events do not always happen at the same time.

By configuring the `delay`, `selector`, and `next` fields in a Stage, you can control when and how the stage is applied,
providing a flexible and scalable way to simulate real-world scenarios in your Kubernetes cluster.

## Expressions string

The `<expressions-string>` is provided by the [Go Implementation](https://github.com/itchyny/gojq) of [JQ Expressions](https://stedolan.github.io/jq/manual/#Basicfilters)

## Examples

### Node Stages

This example shows how to configure the simplest and fastest stages of Node resource, which is also the default Node stages for `kwok`.

[Default Node Stages](https://github.com/kubernetes-sigs/kwok/blob/main/stages/node-fast.yaml)

{{< mermaid >}}

stateDiagram-v2

    state "node-initialize" as node_initialize
    state "node-heartbeat" as node_heartbeat

    [*] --> node_initialize: Node be created that managed by kwok
    node_initialize --> node_heartbeat: Update heartbeat
    node_heartbeat --> node_heartbeat: Update heartbeat

{{< /mermaid >}}

The `node-initialize` Stage will be applied to nodes that do not have any conditions set in their `status.conditions` field.
When applied, this Stage will set the `status.conditions` field for the node, as well as the `status.addresses`, `status.allocatable`,
and `status.capacity` fields.

The `node-heartbeat` Stage will be applied to nodes that have the `Ready` condition set to `True` in their `status.conditions` field.
When applied, maintain the `status.conditions` field for the node.

### Pod Stages

This example shows how to configure the simplest and fastest stages of Pod resource, which is also the default Pod stages for `kwok`.

[Default Pod Stages](https://github.com/kubernetes-sigs/kwok/blob/main/stages/pod-fast.yaml)

{{< mermaid >}}

stateDiagram-v2

    state "pod-create-and-ready" as pod_create_and_ready
    state "pod-completed-for-job" as pod_completed_for_job
    state "pod-delete" as pod_delete
    
    [*] --> pod_create_and_ready: Pod scheduled to Node that managed by kwok
    pod_create_and_ready --> pod_completed_for_job: If pod create by job
    pod_create_and_ready --> pod_delete: .metadata.deletionTimestamp be set
    pod_completed_for_job --> pod_delete: .metadata.deletionTimestamp be set
    pod_delete --> [*] : Pod be deleted

{{< /mermaid >}}

The `pod-create-and-ready` Stage will be applied to pods that do not have a `status.podIP` set and do not have a `metadata.deletionTimestamp` set.
When applied, this Stage will set the `status.conditions`, `status.containerStatuses`, and `status.initContainerStatuses` fields for the pod,
as well as the `status.hostIP` and `status.podIP` fields. It will also set the phase and startTime fields, indicating that the pod is running and has been started.

The `pod-completed-for-job` Stage will be applied to pods that are running, do not have a `metadata.deletionTimestamp` set,
and are owned by a Job. When applied, this Stage will update the `status.containerStatuses` field for the pod,
setting the ready and started fields to true and the `state.terminated` field to indicate that the pod has completed.
It will also set the phase field to Succeeded, indicating that the pod has completed successfully.

The `pod-delete` Stage will be applied to pods that have a `metadata.deletionTimestamp` set.
When applied, this Stage will empty the `metadata.finalizers` field for the pod, allowing it to be deleted, and then delete the pod.

<img width="700px" src="/stages-pod-fast.svg">

### Pod Stages that simulate real behavior as closely as possible

[General Pod Stages](https://github.com/kubernetes-sigs/kwok/blob/main/stages/pod-general.yaml)

This Stages will simulate real Pod behavior as closely as possible in the future,
which is not perfect at the moment, so the refinement of this configuration is still a **Work In Progress**.

<img width="700px" src="/stages-pod-general.svg">
