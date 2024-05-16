## Failed Node Scale up: Insufficient nvidia.com/gpu
Context:
- to request and limit the number of GPUs on an instance, we add a node resource limit,
```yaml
resources:
    limits:
         nvidia.com/gpu: `xx` #how many gpus you want to use  
```
and since we use CA to handle node group scaling, we need to define certain labels on the node groups that allow the CA to find the node the pending pods can fit in. (node labels, capacity).
Ex:
```json
{
            "k8s-io-cluster-autoscaler-node-template-label-machine-type": "g2-standard-8",
            "k8s-io-cluster-autoscaler-default": "true",
            "k8s-io-cluster-autoscaler-enabled": "true",
            "k8s-io-cluster-name": "default",
            "k8s-io-cluster-autoscaler-node-template-label-gpu-type": "nvidia-l4",
            "k8s-io-cluster-autoscaler-node-template-label-gpu-count": "1",
            "k8s-io-cluster-autoscaler-node-template-label-gpu": "true",
            "k8s-io-cluster-autoscaler-node-template-taint-nvidia-com-gpu": "exists-noschedule"
}
```
 
Issue:
The label that defines the nvidia-com/gpu resource capacity should be defined like this:
 ```
   "k8s-io-cluster-autoscaler-node-template-resources-nvidia-com-gpu": "1",
 ```
But the key is longer than 63 characters, so it is rejected by GCP.
thus, pods using this resource limit dont get scheduled. 
```
I0307 11:01:42.041800       1 orchestrator.go:546] Pod XXX can't be scheduled on xxx, predicate checking error: Insufficient nvidia.com/gpu; predicateName=NodeResourcesFit; reasons: Insufficient nvidia.com/gpu; debugInfo=
```

I am looking at https://github.com/kubernetes/autoscaler/issues/5278.
possible solution, add it in `tags` : https://github.com/kubernetes/autoscaler/pull/5214
