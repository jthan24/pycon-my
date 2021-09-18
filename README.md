# pycon-my

# The role of Python with IaC concept
> How to Python increase productivity over IaC and DevOps teams for deploy infrastructure with best practice and generate new value for the companies!!

> Makes with Python a new tool for your IT teams, and increase the value for generate more ROI for your organization using the IaC concepts with all your IT teams, from develop, QA, operation, Network and IT teams along the organization to reduce the toil across your teams. From SRE practice and DevOps definitions use a tool capable to deploy and broke the paradigm in your organization.

https://www.pulumi.com/docs/get-started/kubernetes/

## Prereqs

For simple purpose we use minikube for that

```bash
minikube start
```

For more easy solution I use asdf

Install pulumi 
```bash
asdf install pulumi latest
asdf local pulumi latest
```

Install python 
```bash
asdf install python latest
asdf local python latest
```

## Start with pulumi

```bash
mkdir pulumisrc
cd pulumisrc
pulumi new kubernetes-python
```
 >> Before that we have a folder with config files and src files, you need install dependencies with this command `pip install -r requirements.txt`

### Check current environment 
```bash
pulumi preview
```

you can see similar like this

```bash
[resource plugin kubernetes-3.7.1] installing
Downloading plugin: 29.38 MiB / 29.38 MiB [=========================] 100.00% 1s
     Type                              Name           Plan       
 +   pulumi:pulumi:Stack               pulumisrc-dev  create     
 +   └─ kubernetes:apps/v1:Deployment  nginx          create     
 
Resources:
    + 2 to create
```

### Install your current infrastructure

```bash
pulumi up
```

You can see this
```bash
Previewing update (dev)

View Live: https://app.pulumi.com/jthan24/pulumisrc/dev/previews/e08660cb-a8a1-4101-b6ef-6661578123cd

     Type                              Name           Plan       
 +   pulumi:pulumi:Stack               pulumisrc-dev  create     
 +   └─ kubernetes:apps/v1:Deployment  nginx          create     
 
Resources:
    + 2 to create

Do you want to perform this update? yes
Updating (dev)

View Live: https://app.pulumi.com/jthan24/pulumisrc/dev/updates/1

     Type                              Name           Status      
 +   pulumi:pulumi:Stack               pulumisrc-dev  created     
 +   └─ kubernetes:apps/v1:Deployment  nginx          created     
 
Outputs:
    name: "nginx-bpmwdoes"

Resources:
    + 2 created

Duration: 13s
```

### Modify source

added current content in `__main__.py`

```python
frontend = Service(
    app_name,
    metadata={
        "labels": deployment.spec["template"]["metadata"]["labels"],
    },
    spec={
        "type": "ClusterIP" if is_minikube else "LoadBalancer",
        "ports": [{ "port": 80, "target_port": 80, "protocol": "TCP" }],
        "selector": app_labels,
    })

# When "done", this will print the public IP.
result = None
if is_minikube:
    result = frontend.spec.apply(lambda v: v["cluster_ip"] if "cluster_ip" in v else None)
else:
    ingress = frontend.status.apply(lambda v: v["load_balancer"]["ingress"][0] if "load_balancer" in v else None)
    if ingress is not None:
        result = ingress.apply(lambda v: v["ip"] if "ip" in v else v["hostname"])

pulumi.export("ip", result)
```

created file `Pulumi.dev.yaml` with this content:

```yaml
config:
  pulumisrc:isMinikube: "true"
```

>> Check currents changes over previous deployment

```bash
pulumi preview
```

output is similiar to
```bash
Previewing update (dev)

View Live: https://app.pulumi.com/jthan24/pulumisrc/dev/previews/fa37363b-0a50-48c9-a55d-040d4c4c1431

     Type                           Name           Plan       
     pulumi:pulumi:Stack            pulumisrc-dev             
 +   └─ kubernetes:core/v1:Service  nginx          create     
 
Resources:
    + 1 to create
    2 unchanged
```

>> Apply change in infrastructure

```bash
pulumi up
```

Output similiar to
```bash
Previewing update (dev)

View Live: https://app.pulumi.com/jthan24/pulumisrc/dev/previews/a65ccb50-e3d2-47b2-94c4-b478b334d137

     Type                           Name           Plan       
     pulumi:pulumi:Stack            pulumisrc-dev             
 +   └─ kubernetes:core/v1:Service  nginx          create     
 
Resources:
    + 1 to create
    2 unchanged

Do you want to perform this update? yes
Updating (dev)

View Live: https://app.pulumi.com/jthan24/pulumisrc/dev/updates/2

     Type                           Name           Status      
     pulumi:pulumi:Stack            pulumisrc-dev              
 +   └─ kubernetes:core/v1:Service  nginx          created     
 
Outputs:
  + ip  : "10.103.24.120"
    name: "nginx-bpmwdoes"

Resources:
    + 1 created
    2 unchanged

Duration: 12s
```

## Check changes over kubernetes

>> Checking current services
```bash
kubectl get service
```

output
```bash
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP   6d23h
nginx-14jmjr6x   ClusterIP   10.103.24.120   <none>        80/TCP    57s
```

>> Checking currents Pods
```bash
kubectl get pods
```

output
```bash
AME                              READY   STATUS    RESTARTS   AGE
nginx-bpmwdoes-6799fc88d8-hrxrq   1/1     Running   0          22m
```

## Clean our work area

>> Delete resources with pulumi are very easy

```bash
pulumi destroy
```

output

```bash
Previewing destroy (dev)

View Live: https://app.pulumi.com/jthan24/pulumisrc/dev/previews/81aaf0da-045d-4dc4-865e-335f783fb7c9

     Type                              Name           Plan       
 -   pulumi:pulumi:Stack               pulumisrc-dev  delete     
 -   ├─ kubernetes:core/v1:Service     nginx          delete     
 -   └─ kubernetes:apps/v1:Deployment  nginx          delete     
 
Outputs:
  - ip  : "10.103.24.120"
  - name: "nginx-bpmwdoes"

Resources:
    - 3 to delete

Do you want to perform this destroy? yes
Destroying (dev)

View Live: https://app.pulumi.com/jthan24/pulumisrc/dev/updates/3

     Type                              Name           Status      
 -   pulumi:pulumi:Stack               pulumisrc-dev  deleted     
 -   ├─ kubernetes:core/v1:Service     nginx          deleted     
 -   └─ kubernetes:apps/v1:Deployment  nginx          deleted     
 
Outputs:
  - ip  : "10.103.24.120"
  - name: "nginx-bpmwdoes"

Resources:
    - 3 deleted

Duration: 2s

The resources in the stack have been deleted, but the history and configuration associated with the stack are still maintained. 
If you want to remove the stack completely, run 'pulumi stack rm dev'.
```

>> Checking objects over kubernetes

```bash
kubectl get services,pods
```

output
```bash
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6d23h
```

