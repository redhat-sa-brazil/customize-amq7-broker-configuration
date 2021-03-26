# Custom AMQ 7 Broker Configuration

## Description

The following is intended to showcase how to customize **Red Hat AMQ 7** configuration files *(brooker.xml, bootstrap.xml, etc..)* when using [AMQ 7 Application Templates](https://access.redhat.com/documentation/en-us/red_hat_amq/7.6/html-single/deploying_amq_broker_on_openshift/index#deploying_broker-on-ocp-using-templates_broker-ocp)

## Deployment

* Create an **Openshift** Project/Namespace: `oc new-project amq-broker`

* Deploy **Red Hat AMQ 7 Templates**: `oc replace --force  -f https://raw.githubusercontent.com/jboss-container-images/jboss-amq-7-broker-openshift-image/76-7.6.0.GA/amq-broker-7-image-streams.yaml`

* Before moving forward execute the actions described on this reference: [AMQ 7.7 fails to create from templates on OCP 4.5](https://access.redhat.com/solutions/5507671)

* Create an **Red Hat AMQ Broker instance:**

  ```
  oc new-app --template=amq-broker-76-persistence-clustered \
    -p AMQ_PROTOCOL=openwire,amqp,stomp,mqtt,hornetq \
    -p AMQ_QUEUES=demoQueue \
    -p AMQ_ADDRESSES=demoTopic \
    -p VOLUME_CAPACITY=1Gi \
    -p AMQ_CLUSTERED=true \
    -p AMQ_REPLICAS=1 \
    -p AMQ_USER=amq-demo-user \
    -p AMQ_PASSWORD=password
  ```
  
 * Create a [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/) with all **Red Hat AMQ** files, or if you prefer, just create the ones that you want to customize. Example:

  ```
  Option 1 - Individual ConfigMap for each file
  oc create configmap artemis-users \
    --from-file=artemis-users.properties

  Option 2 - A bundle of AMQ configuration files within one single ConfigMap
  oc create configmap artemis-config \
    --from-file=artemis-roles.properties=artemis-roles.properties \
    --from-file=artemis-users.properties=artemis-users.properties \
    --from-file=artemis.profile=artemis.profile \
    --from-file=bootstrap.xml=bootstrap.xml \
    --from-file=broker.xml=broker.xml \
    --from-file=jgroups-ping.xml=jgroups-ping.xml \
    --from-file=jolokia-access.xml=jolokia-access.xml \
    --from-file=logging.properties=logging.properties \
    --from-file=login.config=login.config \
    --from-file=management.xml=management.xml
  ```
  
* Finally apply your [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/) on **Red Hat AMQ**:

  ```
  Option 1 - Individual ConfigMap for each file
  oc set volume statefulset/broker-amq --add \
    --name=artemis-users \
    --type=configmap \
    --configmap-name=artemis-users \
    --mount-path=/opt/amq/conf/artemis-users.properties \
    --sub-path=artemis-users.properties
  
  Option 2 - A bundle of AMQ configuration files within one single ConfigMap
  oc set volume statefulset/broker-amq --add \
    --name=artemis-config \
    --type=configmap \
    --configmap-name=artemis-config \
    --mount-path=/opt/amq/conf
  ```
  
## References
  
- [Deploying AMQ Broker on OpenShift Container Platform using Application Templates](https://access.redhat.com/documentation/en-us/red_hat_amq/7.6/html-single/deploying_amq_broker_on_openshift/index#deploying_broker-on-ocp-using-templates_broker-ocp)
- [Customizing AMQ Broker configuration files for deployment](https://access.redhat.com/documentation/en-us/red_hat_amq/7.6/html-single/deploying_amq_broker_on_openshift/index#proc_br-custom-broker-config-deploy_broker-ocp)
