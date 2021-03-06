# The life of a Pod

Updated: 4/14/2015

This document covers the lifecycle of a Pod.  It is not an exhaustive document, but an introduction to the topic.

## Pod Phase

As consistent with the overall [API convention](api-conventions.md#typical-status-properties), phase is a simple, high-level summary of the phase of the lifecycle of a Pod. It is not intended to be a comprehensive rollup of observations of container-level or even pod-level conditions or other state, nor is it intended to be a comprehensive state machine.

The number and meanings of `PodPhase` values are tightly guarded.  Other than what is documented here, nothing should be assumed about Pods with a given `PodPhase`.

* Created: The Pod has been accepted by the system, but one or more of the container images have not been created.  This includes time before being scheduled as well as time spent downloading images over the network, which could take a while.
* Running: The Pod has been bound to a node, and all of the containers have been created. At least one container is still running, or is in the process of starting or restarting.
* Succeeded: All containers in the Pod have been terminated with success, and will not be restarted.
* Failed: All containers in the Pod have been terminated, at least one container has been terminated with failure (exited with non-zero exit status or was terminated by the system).

## Pod Conditions

A Pod containing containers that specify readiness probes will also report the Ready condition. Condition status values may be `True`, `False`, or `Unknown`.

## Container Statuses

More detailed information about the current (and previous) container statuses can be found in `containerStatuses`. The information reported depends on the current ContainerState, which may be Waiting, Running, or Termination (sic).

## RestartPolicy

The RestartPolicy may be `Always`, `OnFailure`, or `Never`. RestartPolicy applies to all containers in the Pod. RestartPolicy only refers to restarts of the containers by the Kubelet on the same node. As discussed in the [Pods document](pods.md#durability-of-pods-or-lack-thereof), once bound to a node, a Pod may never be rebound to another node. This means that some kind of controller is necessary in order for a Pod to survive node failure, even if just a single Pod at a time is desired.

The only controller we currently have is [`ReplicationController`](replication-controller.md). `ReplicationController` is *only* appropriated for Pods with `RestartPolicy = Always`. `ReplicationController` should refuse to instantiate any Pod with a different restart policy.

There is a legitimate need for a controller which keeps Pods with other policies alive. Both of the other policies (`OnFailure` and `Never`) eventually terminate, at which point the controller should stop recreating them. Because of this fundamental distinction, let's hypothesize a new controller, called [`JobController`](https://github.com/GoogleCloudPlatform/kubernetes/issues/1624) for the sake of this document, which can implement this policy.

## Pod lifetime

In general terms, Pods which are created do not disappear until an action destroys them. This might be a human or a `ReplicationController`. The only exception to this rule is when Pods with a `PodPhase` of `Succeeded` or `Failed` for more than some duration (determined by the master) will expire and be automatically reaped.

If a node dies or is disconnected from the rest of the cluster, some entity within the system (call it the NodeController for now) is responsible for applying a policy (e.g. a timeout), and marking any Pod on the lost node as `Failed`.

## Examples

   * Pod is `Running`, 1 container, container exits success
     * Log completion event
     * If RestartPolicy is:
       * Always: restart container, Pod stays `Running`
       * OnFailure: Pod becomes `Succeeded`
       * Never: Pod becomes `Succeeded`

   * Pod is `Running`, 1 container, container exits failure
     * Log failure event
     * If RestartPolicy is:
       * Always: restart container, Pod stays `Running`
       * OnFailure: restart container, Pod stays `Running`
       * Never: Pod becomes `Failed`

   * Pod is `Running`, 2 containers, container 1 exits failure
     * Log failure event
     * If RestartPolicy is:
       * Always: restart container, Pod stays `Running`
       * OnFailure: restart container, Pod stays `Running`
       * Never: Pod stays `Running`
     * When container 2 exits...
       * Log failure event
       * If RestartPolicy is:
         * Always: restart container, Pod stays `Running`
         * OnFailure: restart container, Pod stays `Running`
         * Never: Pod becomes `Failed`

   * Pod is `Running`, container becomes OOM
     * Container terminates in failure
     * Log OOM event
     * If RestartPolicy is:
       * Always: restart container, Pod stays `Running`
       * OnFailure: restart container, Pod stays `Running`
       * Never: log failure event, Pod becomes `Failed`

   * Pod is `Running`, a disk dies
     * All containers are killed
     * Log appropriate event
     * Pod becomes `Failed`
     * If running under a controller, Pod will be recreated elsewhere

   * Pod is `Running`, its node is segmented out
     * NodeController waits for timeout
     * NodeController marks pod `Failed`
     * If running under a controller, Pod will be recreated elsewhere


[![Analytics](https://kubernetes-site.appspot.com/UA-36037335-10/GitHub/docs/pod-states.md?pixel)]()
