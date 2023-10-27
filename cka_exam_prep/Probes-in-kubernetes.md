# Container probes

A probe is a diagnostic performed periodically by the kubelet on a container. To perform a diagnostic, the kubelet either executes code within the container, or makes a network request.

## Check mechanisms

There are four different ways to check a container using a probe. Each probe must define exactly one of these four mechanisms:

**exec**</br>
Executes a specified command inside the container. The diagnostic is considered successful if the command exits with a status code of 0.</br>
**grpc**</br>
Performs a remote procedure call using gRPC. The target should implement gRPC health checks. The diagnostic is considered successful if the status of the response is SERVING.</br>
**httpGet**</br>
Performs an HTTP GET request against the Pod's IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400.</br>
**tcpSocket**</br>
Performs a TCP check against the Pod's IP address on a specified port. The diagnostic is considered successful if the port is open. If the remote system (the container) closes the connection immediately after it opens, this counts as healthy.</br>

## Probe outcome

Each probe has one of three results:

**Success**</br>
The container passed the diagnostic.</br>
**Failure**</br>
The container failed the diagnostic.</br>
**Unknown**</br>
The diagnostic failed (no action should be taken, and the kubelet will make further checks).</br>

## Types of probe

The kubelet can optionally perform and react to three kinds of probes on running containers:

**livenessProbe**</br>
Indicates whether the container is running. If the liveness probe fails, the kubelet kills the container, and the container is subjected to its restart policy. If a container does not provide a liveness probe, the default state is Success.</br>
Liveness probes let Kubernetes know if your app is alive or dead. If your app is alive, then Kubernetes leaves it alone. If your app is dead, Kubernetes removes the Pod and starts a new one to replace it.</br>
**readinessProbe**</br>
Indicates whether the container is ready to respond to requests. If the readiness probe fails, the endpoints controller removes the Pod's IP address from the endpoints of all Services that match the Pod. The default state of readiness before the initial delay is Failure. If a container does not provide a readiness probe, the default state is Success.</br>
Readiness probes are designed to let Kubernetes know when your app is ready to serve traffic. Kubernetes makes sure the readiness probe passes before allowing a service to send traffic to the pod. If a readiness probe starts to fail, Kubernetes stops sending traffic to the pod until it passes.</br>
**startupProbe**</br>
Indicates whether the application within the container is started. All other probes are disabled if a startup probe is provided, until it succeeds. If the startup probe fails, the kubelet kills the container, and the container is subjected to its restart policy. If a container does not provide a startup probe, the default state is Success.</br>

For more information about how to set up a liveness, readiness, or startup probe, see [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/).</br>

- **initialDelaySeconds:** Number of seconds after the container has started before liveness or readiness probes are initiated.
- **periodSeconds:** How often (in seconds) to perform the probe.
- **timeoutSeconds:** Number of seconds after which the probe times out.
- **failureThreshold:** When a probe fails, Kubernetes will try failureThreshold times before giving up. Giving up in case of liveness probe means restarting the container. In case of readiness probe the Pod will be marked Unready.
