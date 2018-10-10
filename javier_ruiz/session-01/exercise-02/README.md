# Exercise II

Create the namespace

    $ kubectl apply -f exercise-02-namespace.yaml

Create the mariadb and wordpress deployment, configmap, secret and service.

    $ kubectl apply -f .

Change the default namespace to exercise-02

    $ kubectlconfig set-context $(k config current-context) --namespace=exercise-02

Watch the creation of the wordpress pod till some of then show "Starting wordpress..."

    $ watch 'kubectl get pod -l app=wordpress -o name | while read pod; do
     echo 
     echo --- $pod
     echo
     kubectl logs $pod|tail -15
    done'

Launch a browser to http://localhost:8001

To delete all the resources created, just delete the exercise-01 namespace

    $ kubectl delete ns exercise-02

To configure that it won't be more than 40% of the replicas unavailable add the following to the deployment.spec.strategy section:

    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxUnavailable: 40%

To scale the mariadb-slaves deployment to 5 replicas use the command:

    kubectl scale deployment/mariadb-slave --replicas=5

The steps needed to configure the HyperDB WP plugin are:

    TODO
