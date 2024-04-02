This custom workload type 'autoscaling-server' is the same as the existing workload type 'server' with an added HorizontalPodAutoscaler CR and Kapp Config to allow the changes the autoscaler makes to the replica count to stay and not be reverted.

This new workload type has support for min and max replicas using CPU scaling by default and allows configuration through a parameter called 'autoscaling'. Outside of min/max replicas and CPU..you can set your own metrics config to override the default CPU metrics usage.

workload examples

enable the new workload type

``` yaml
apiVersion: carto.run/v1alpha1
kind: Workload
metadata:
  name: hpa-metrics-tanzu-java-web-app
  namespace: dev
  labels:
    apps.tanzu.vmware.com/workload-type: autoscaling-server
```


once enabled, the following is the default HPA you will get
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-metrics-tanzu-java-web-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-metrics-tanzu-java-web-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 65
```

You can adjust these defaults in the `hpa-template.yaml` within this PR

For customizing, you would adjust the workload yaml through the param of autoscaling. Here are a couple examples
```yaml
spec:
  params:
  - name: autoscaling
    value:
      min_replicas: 5
      max_replicas: 15
      cpu_average_utilization: 80
```

Min/Max/Avg Utilization have defaults, and if you do not enter the values, the defaults are taken. (meaning you can enter 1, all 3, or none)


If you do not want to use CPU, you can provide your own metrics block. (sample pulled from k8s doc)
If you provide the metrics in the param, the default of CPU will not be added and it is on you to add that back in if you want it. You cannot use `cpu_average_utilization` in addition to `metrics`, it will error and let you know one or the other.

```yaml
  params:
  - name: autoscaling
    value:
      min_replicas: 2
      max_replicas: 30
      metrics:
      - type: Resource
        resource:
          name: cpu
          target:
            type: Utilization
            averageUtilization: 50
      - type: Pods
        pods:
          metric:
            name: packets-per-second
          target:
            type: AverageValue
            averageValue: 1k
```