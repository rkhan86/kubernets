[Integrating Kubernetes via the Addon](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/)</br>
[Changing Configuration Options](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-changing-configuration-options)

- additional arguments may be supplied to the Weave router process by adding them to the command: array in the YAML file,
- additional parameters can be set via the environment variables listed above; these can be inserted into the YAML file like this:

```yaml
containers:
  - name: weave
    env:
      - name: IPALLOC_RANGE
        value: 10.0.0.0/16
```
