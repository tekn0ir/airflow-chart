# Airflow chart
Helm chart for deploying Apache Airflow on kubernetes
Read more about Kubernetes Executor and Operator [here](https://airflow.incubator.apache.org/kubernetes.html).

## Guideance
To force you not to end up in performance and/or other issues, this template takes some experience into account.

### Chain of configuration
The chart might seem to have secrets and configmaps that isn´t used, and worker pods might seem to be missing mounts.
The configuration design of airflow is not straight forward. Webservers and scheduler gets config overloaded with a
configmap while a secret populate the configration file, that in turn creates environment variables for the workers(tasks).
All of which is configurable from the values file.

The chart automatically sets the following variables:
AIRFLOW__CORE__SQL_ALCHEMY_CONN,
AIRFLOW__KUBERNETES__AIRFLOW_CONFIGMAP,
AIRFLOW__KUBERNETES__NAMESPACE
and:
AIRFLOW__KUBERNETES__WORKER_SERVICE_ACCOUNT_NAME
if rbac is enabled.

### Database backend
If you want your own DB backend for Airflow, just disable postgresql in the values file and add the sqlAlchemyConn
value in the values file:
```
sqlAlchemyConn: postgresql+psycopg2://username:password@db-hostname:5432/schema
```

### Provision connections
As the DB needs to be at least initialized, but also connections has to be provisioned to Ariflow. There is a provision
job for this. See the provisioner section of the values file for some inspiration.

### Provision dags
Inner workings of airflow seems to work best if you do not share a volume for dags, but rather put your materialized dags
as a layer on your docker image. Not hosting files on a remote filesystem also improves performance a small bit.
It might seem cumbersome to rebuild your docker image each time and you might have dags in many different repos with
different pipelines. I am sorry, the most reliable way is to do it like this.

### Installing the Chart

To install the chart with the release name `my-airflow` in the `my-airflow` namespace:

```
$ helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
$ helm upgrade --install my-airflow --namespace=my-airflow incubator/airflow
```

This chart includes a postgresql chart as a dependency to the Airflow cluster in its `requirement.yaml` by default. 
The chart can be customized using the following configurable parameters:

| Parameter                      | Description                                                                                                                      | Default                                        |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| `airflowImage`                 | Airflow Container image name                                                                                                     | `tekn0ir/airflow-docker`                       |
| `airflowImageTag`              | Airflow Container image tag                                                                                                      | `1.10.1rc2`                                    |
| `imagePullPolicy`              | Airflow Container pull policy                                                                                                    | `IfNotPresent`                                 |
| `fernetKey`                    | Airflow fernet key, for encryption of data                                                                                       | `af7CN0q6ag5U3g08IsPsw3K45U7Xa0axgVFhoh-3zB8=` |
| `ingress.enabled`              | Enables Ingress for Drone                                                                                                        | `true`                                         |
| `ingress.annotations`          | Ingress annotations                                                                                                              | `{}`                                           |
| `ingress.labels`               | Ingress labels                                                                                                                   | `{}`                                           |
| `ingress.hosts`                | Ingress accepted hostnames                                                                                                       | `[airflow.192.168.99.100.xip.io]`              |
| `ingress.tls`                  | Ingress TLS configuration                                                                                                        | `[]`                                           |
| `service.annotations`          | Service annotations                                                                                                              | `{prometheus.io scrape config}`                                           |
| `webserver.replicas`           | Number of webserver replicas                                                                                                     | `2`                                            |
| `webserver.annotations`        | Webserver annotations                                                                                                            | `{}`                                           |
| `scheduler.annotations`        | Scheduler annotations                                                                                                            | `{}`                                           |
| `rbac.enabled`                 | Enable a service account and role for the cluster to use                                                                         | `true`                                         |
| `serviceAccountName`           | ServiceAccount namer to use if it cannot be created with RBAC                                                                    | ``                                             |
| `provisioner.enabled`          | Enable the provisioning job to run arbitrary bash commands in the Airflow cluster, example initiate DB and provision connections | `true`                                         |
| `provisioner.cmds`             | The provisioning commands...                                                                                                     | `...`                                          |
| `config`                       | Set any environment variable, mostly used to set any airflow setting by the scheme AIRFLOW__{SECTION}__{KEY}                     | `...`                                          |
| `postgresql.enabled`           | Configure dependency: https://github.com/helm/charts/tree/master/stable/postgresql                                               | `true`                                         |

Specify parameters using `--set key=value[,key=value]` argument to `helm install`

Alternatively a YAML file that specifies the values for the parameters can be provided like this:

```bash
$ helm install --name my-airflow -f values.yaml incubator/airflow
```
