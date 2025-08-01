# Limit Ranges

Quotas limits resources in a namespace but then there is 
nothing within namespace which can restrict individual to consume entire quota.

Limits ranges define limits per Pod and Container.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
    name: hellolimitrange
spec:
    limits:
        - default:
            memory: 512Mi # this the default limits
          defaultRequest:
            memory: 256Mi
# max and min values below applies to both limits and requests
          max:
            memory: 1Gi
          min:
            memory: 128Mi
    type: Container 

```

LimitsRange applies to
    - pods
    - containers
    - image streams
    - pvc
    - images

## Steps

- First create a LimitRange as specified in the file 101limitrange.yaml, pay special attention to indentation under limits.
    - it is list.
- 'oc create --save-config -f 101limitrange.yaml`
- Second, create a httpd deployment using the `oc new-app limithttpd --image-stream httpd`

```shell
oc describe limitranges limitrangeone 
Name:       limitrangeone
Namespace:  selfservice-ranges
Type        Resource  Min  Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---  ---  ---------------  -------------  -----------------------
Container   memory    -    -    256Mi            512Mi          -

```

- Check if the limits and requests have been applied, use the following command

```shell
oc get pods limithttpd-869d8b4b75-dz89v -o json | jq .spec.containers[].resources
{
  "limits": {
    "memory": "512Mi"
  },
  "requests": {
    "memory": "256Mi"
  }
}
```

Since we have setup a limitrange we do not have to explictly set the limit inside the manifest file
This can be proved by deploying httpd24-app.yaml which has no resources section defined.
defined but when you check

```shell
oc get pods httpd24-app-d7679fc9d-tqhzh -o json | jq .spec.containers[].resources
{
  "limits": {
    "memory": "512Mi"
  },
  "requests": {
    "memory": "256Mi"
  }
}
```

But the moment, you apply quota things changes to default i.e. you cannot deploy resources without
specifying the resources.
