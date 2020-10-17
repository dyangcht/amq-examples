# Spring-Boot Camel QuickStart

This example demonstrates how you can use Apache Camel with Spring Boot.

The quickstart uses Spring Boot to configure a little application that includes a Camel route that triggers a message every 5th second, and routes the message to a log.

### Building

The example can be built with

    mvn clean install

### Running the example in OpenShift

It is assumed that:
- OpenShift platform is already running, if not you can find details how to [Install OpenShift at your site](https://docs.openshift.com/container-platform/3.3/install_config/index.html).
- Your system is configured for Fabric8 Maven Workflow, if not you can find a [Get Started Guide](https://access.redhat.com/documentation/en/red-hat-jboss-middleware-for-openshift/3/single/red-hat-jboss-fuse-integration-services-20-for-openshift/)

The example can be built and run on OpenShift using a single goal:

    mvn fabric8:deploy

When the example runs in OpenShift, you can use the OpenShift client tool to inspect the status

To list all the running pods:

    oc get pods

Then find the name of the pod that runs this quickstart, and output the logs from the running pods with:

    oc logs <name of pod>

You can also use the OpenShift [web console](https://docs.openshift.com/container-platform/3.3/getting_started/developers_console.html#developers-console-video) to manage the
running pods, and view logs and much more.

### Running via an S2I Application Template

Application templates allow you deploy applications to OpenShift by filling out a form in the OpenShift console that allows you to adjust deployment parameters.  This template uses an S2I source build so that it handle building and deploying the application for you.

First, import the Fuse image streams:

    oc create -f https://raw.githubusercontent.com/jboss-fuse/application-templates/GA/fis-image-streams.json

Then create the quickstart template:

    oc create -f https://raw.githubusercontent.com/jboss-fuse/application-templates/GA/quickstarts/spring-boot-camel-template.json

Now when you use "Add to Project" button in the OpenShift console, you should see a template for this quickstart. 

### Running via command line only

預設已使用 oc login 登入 OpenShift 4.x. 也使用 AMQ Streams 安裝了 Kafka Instances, Kafka Topic and Kafka User.

#### Kafka Instance: my-cluster
#### Kafka Toic: my-topc
#### Kafka User: my-user

匯出 Server 憑證及金鑰

    oc get secret my-cluster-cluster-ca-cert -o jsonpath='{.data.ca\.crt}' | base64 -d > ca.crt
    mv ca.crt src/main/resources

產生 JKS

    keytool -import -trustcacerts -alias root -file src/main/resources/ca.crt -keystore src/main/resources/keystore.jks -storepass password -noprompt


匯出 User 憑證及金鑰

    oc extract secret/my-user --keys=user.crt --to=- > user.crt
    oc extract secret/my-user --keys=user.key --to=- > user.key

P12(PKCS12)，二進位制格式，同時包含憑證和私鑰，一般會加上密碼保護。JKS，二進位制格式，同時包含憑證和私鑰，一般會有密碼保護。
以下是將憑證轉成 PKCS12 格式並將密碼設成 "123456"

    openssl pkcs12 -export -in user.crt -inkey user.key -name my-user -password pass:123456 -out user.p12

Java 一般使用 JKS 格式，所以也可以使用下列指令將 PKCS12 轉成 JKS

    keytool -importkeystore -srckeystore user.p12 -destkeystore user.jks -srcstoretype PKCS12 -deststoretype JKS -deststorepass 123456 -srcstorepass 123456 -noprompt
    mv user.jks src/main/resources

執行以下指令來測試 Producer 和 Consumer

    mvn -Drun.jvmArguments="-Dbootstrap.server=my-cluster-kafka-bootstrap-YOURPROJECTNAME.apps.YOURDOMAINNAME:443" clean package spring-boot:run


