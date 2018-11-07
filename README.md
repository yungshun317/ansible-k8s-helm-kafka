$ helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
"incubator" has been added to your repositories

$ helm install --name kafka -f kafka-values.yml incubator/kafka
NAME:   kafka
E1217 14:12:07.445654   13038 portforward.go:303] error copying from remote stream to local connection: readfrom tcp4 127.0.0.1:35597->127.0.0.1:54046: write tcp4 127.0.0.1:35597->127.0.0.1:54046: write: broken pipe
LAST DEPLOYED: Mon Dec 17 14:12:06 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/PodDisruptionBudget
NAME             MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
kafka-zookeeper  N/A            1                0                    0s

==> v1/Service
NAME                      TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)                     AGE
kafka-zookeeper-headless  ClusterIP  None           <none>       2181/TCP,3888/TCP,2888/TCP  0s
kafka-zookeeper           ClusterIP  10.104.253.83  <none>       2181/TCP                    0s
kafka                     ClusterIP  10.103.120.89  <none>       9092/TCP                    0s
kafka-headless            ClusterIP  None           <none>       9092/TCP                    0s

==> v1beta1/StatefulSet
NAME             DESIRED  CURRENT  AGE
kafka-zookeeper  3        1        0s
kafka            3        1        0s

==> v1/Pod(related)
NAME               READY  STATUS             RESTARTS  AGE
kafka-zookeeper-0  0/1    ContainerCreating  0         0s
kafka-0            0/1    Pending            0         0s


NOTES:
### Connecting to Kafka from inside Kubernetes

You can connect to Kafka by running a simple pod in the K8s cluster like this with a configuration like this:

  apiVersion: v1
  kind: Pod
  metadata:
    name: testclient
    namespace: default
  spec:
    containers:
    - name: kafka
      image: confluentinc/cp-kafka:5.0.1
      command:
        - sh
        - -c
        - "exec tail -f /dev/null"

Once you have the testclient pod above running, you can list all kafka
topics with:

  kubectl -n default exec testclient -- /usr/bin/kafka-topics --zookeeper kafka-zookeeper:2181 --list

To create a new topic:

  kubectl -n default exec testclient -- /usr/bin/kafka-topics --zookeeper kafka-zookeeper:2181 --topic test1 --create --partitions 1 --replication-factor 1

To listen for messages on a topic:

  kubectl -n default exec -ti testclient -- /usr/bin/kafka-console-consumer --bootstrap-server kafka:9092 --topic test1 --from-beginning

To stop the listener session above press: Ctrl+C

To start an interactive message producer session:
  kubectl -n default exec -ti testclient -- /usr/bin/kafka-console-producer --broker-list kafka-headless:9092 --topic test1

To create a message in the above session, simply type the message and press "enter"
To end the producer session try: Ctrl+C
