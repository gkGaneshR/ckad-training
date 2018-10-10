# Exercise II Solution

Kubernetes exercise deploying a set of WP instances connected to mariadb.

## Features

* Two wordpress instances, exposed through LoadBalancer service.
* MariaDB as wordpress backend (one master and two slaves), not publicly accesible.
* Config variables at `cm.yaml`
* Secret variables defined at `command.bash`
* Liveness/readiness probes implemented.
* Resources requested and limits.

## How to run

With *kubectl configured*, do `./command.bash` to start the app. The script will print the URL to access wordpress.

## After set-up

### Ensure there won't be more than 40% of the replicas unavailable when running rolling updates on slave deployment
```
kubectl patch deployment -n exercise-02  mariadb-slave -p '{"spec":{"strategy":{"rollingUpdate":{"maxUnavailable":"40%"}}}}'
```

### Scale slaves to 5 replicas
```
kubectl scale deployment -n exercise-02  mariadb-slave --replicas=5
```

### Install & Configure Wordpress HyperDB plugin

WARN: Only after both pods are in a ready state.

First, copy the install script to both wordpress containers (pods with only one container):

```
kubectl cp hyperdb_install.sh exercise-02/$(kubectl get pods -lapp=wordpress,tier=frontend -n exercise-02 -ojsonpath="{.items[0].metadata.name}"):/hyperdb_install.sh

kubectl cp hyperdb_install.sh exercise-02/$(kubectl get pods -lapp=wordpress,tier=frontend -n exercise-02 -ojsonpath="{.items[1].metadata.name}"):/hyperdb_install.sh
```

Then, execute the install script on each container:

```
kubectl exec $(kubectl get pods -lapp=wordpress,tier=frontend -n exercise-02 -o jsonpath="{.items[0].metadata.name}") -n exercise-02 -- /bin/bash -c "chmod +x /hyperdb_install.sh && /hyperdb_install.sh"

kubectl exec $(kubectl get pods -lapp=wordpress,tier=frontend -n exercise-02 -o jsonpath="{.items[1].metadata.name}") -n exercise-02 -- /bin/bash -c "chmod +x /hyperdb_install.sh && /hyperdb_install.sh"
```

## Excercise II original description:

### WP + MariaDB with replication

Create the K8s resources you need to deploy a WP site on your K8s cluster using
a MariaDB cluster as database with the characteristics below:

* All the resources should be created under the *exercise-02* namespace.
* The WP site should use a MariaDB database
* MariaDB should be configured with replication with a master-slave model. There
should be 1 master and 2 slaves
* Use ConfigMaps and Secrets to configure both MariaDB and WP
* Every pod should configure the CPU/RAM requested to the cluster, and the limits
for them.
* Every container should have the proper readiness and liveness probes
configured
* Wordpress should be publicly available while MariaDB should only be accessible
internally (you can consider your cluster supports LoadBalancer service type)

Once everything is setup:

* Ensure there won't be more than 40% or the replicas unavailable when running
rolling updates on slave deployment. You can document how to do it or include the
steps in the *commands.bash* file.
* Scale slaves to 5 replicas. You can document how to do it or include the steps
in the *commands.bash* file.
* Document the steps to follow to install HyperDB WP plugin and configure it to
balance SQL request between Master&Slaves services

### What to deliver

* *README.md* file with a description about the solution you developed and how to
use it. Please be as descriptive as possible and user "Markdown syntax"
* YAML/JSON files with the definitions of every requested K8s object. Templates
are provided.
* If you created your resources from the command line, attach a bash script with
the commands used to create them. Sth like:

```
#!/bin/bash

## Create ns
kubectl create ns ...

## Create secrets
kubectl create secret generic ...
```

Use the structure below on your PR To GitHub:

```
|
|-/session-01
|-/session-01/exercise-02/
|-/session-01/exercise-02/README.md
|-/session-01/exercise-02/commands.bash
|-/session-01/exercise-02/*.{json,yaml}
```

#### Tips

* Use a linter to avoid syntax errors on your YAML/JSON files
* RollingUpdate Strategy: `kubectl explain deploy.spec.strategy`

#### Notes

Docs about how to manager resources on pods:

https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/

Interesting links to consult when configuring HyperDB:

https://wordpress.org/plugins/hyperdb/
https://www.digitalocean.com/community/tutorials/how-to-optimize-wordpress-performance-with-mysql-replication-on-ubuntu-14-04
