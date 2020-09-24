# hpcc-istio

## Install
This is tested local on Docker Desktop

```sh
istioctl install -f install-components.yaml
```

To uninstall
```sh
istioctl manifest generate -f install-components.yaml | kubectl delete -f -
```
## Enable istio-inject
Before start HPCC cluster enabble istio-injection
```sh
 kubectl label namespace ${hpcc_namespace} istio-injection=enabled --overwrite
```
The default namespace is "default"


## Side car lifecycle issue
Since default HPCC Systems cloud will use job pod for eclcc, eclagent and thor and job cannot terminate itself due to envoy still running. It is a known issue for any sidecar job pod for current Kubernetes release (1.19).

Kubernetes is planning to provide a fix but not sure when. The fix is not even in the feature gate yet.

JIRA HPCC : HPCC-24548
HPCC Pods hanging with istio envoy (sidecar) running

To work around the problem
### useChileProcesses:true
Set useChileProcesses:true for eclagent and eclccserver. 
This launch normal deployment pod instead dynamic job for compile and hthor/roxie workunits.
This doesn't help thor cases

### special Docker image
There is a special Docker image hpccsystems/platform-core:pilot-agent which has a private fix JIRA HPCC-24548


## Performance
With envoy side-car enabled the job pod start time is 2 to 5 longer than before. We will investigate and search a solution.

Following may helps 
https://medium.com/@marko.luksa/delaying-application-start-until-sidecar-is-ready-2ec2d21a7b74

https://preliminary.istio.io/latest/news/releases/1.7.x/announcing-1.7/change-notes/

Added config option values.global.proxy.holdApplicationUntilProxyStarts, which causes the sidecar injector to inject the sidecar at the start of the pod’s container list and configures it to block the start of all other containers until the proxy is ready. This option is disabled by default. (Issue #11130)


https://github.com/istio/istio/pull/24737


