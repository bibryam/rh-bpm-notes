# Install

## Installation 

**Prerequisite**: JDK, JBoss EAP 6.4.8

Launch:

    $ java -jar jboss-brms-6.3.0.GA-installer.jar
    

## Users
jbpm
Admin123$

To add a user use the EAP command line:

    bin/add-user.sh admin admin0

Users are listed in the following file:

    ~/EAP-6.4.0/standalone/configuration/application-users.properties

Roles are defined in the following file:

    ~/EAP-6.4.0/standalone/configuration/application-roles.properties


### Example Roles
    admin=admin,analyst,user,kie-server,rest-all

### Assigning role to projects


https://github.com/jboss-gpe-ref-archs/bpm_deployments/blob/master/doc/multi-tenant-bpm.adoc


## Database / datasource
By default, out-of-the-box it points to a datasource java:jboss/datasources/ExampleDS which is configured to use H2 data store in standalone*.xml files.

The default configuration H2 is configured in mem, so every restart you get a fresh DB.

To be precise, following are the two places where the `java:jboss/datasources/ExampleDS datasource` is used.


    business-central.war/WEB-INF/classes/META-INF/persistence.xml
    dashbuilder.war/WEB-INF/jboss-web.xml
    
Here are the changes that need to be done, in order to configure BPMS 6 to use an external Database.
business-central.war/WEB-INF/classes/META-INF/persistence.xml

    Create a new datasource and install the respective JDBC driver by following the instructions from EAP 6 Documentation .
    The default used H2 database specific datasource configuration looks like the following. Using modular approach to install the JDBC driver would be ideal for ease of configurations later.

Raw

      <subsystem xmlns="urn:jboss:domain:datasources:1.1">
         <datasources>
            <datasource jndi-name="java:jboss/datasources/ExampleDS" pool-name="ExampleDS" enabled="true" use-java-context="true">
               <connection-url>jdbc:h2:mem:test;DB_CLOSE_DELAY=-1</connection-url>
               <driver>h2</driver>
               <security>
                  <user-name>sa</user-name>
                  <password>sa</password>
               </security>
            </datasource>
            <drivers>
               <driver name="h2" module="com.h2database.h2">
                  <xa-datasource-class>org.h2.jdbcx.JdbcDataSource</xa-datasource-class>
               </driver>
            </drivers>
         </datasources>


### Persistent H2 store

    <connection-url>jdbc:h2:h2.filestore;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE</connection-url>

**TCP variant**

    <connection-url>jdbc:h2:tcp://localhost/~/h2tcp-filestore;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE</connection-url>



## Updating / Patching BPM Suite

The topic is well covered in the Installation Guide.

1. Unzip `jboss-bpmsuite-6.3.2-patch.zip`
2. Issue `./apply-updates.sh ~/EAP-6.4.0/ eap6.x`
3. Extract the maven repo update in the previous repo directory

### Updating JBoss EAP

Downlaod the EAP patched and unzip the bundle to get the incremental fix.
E.g. jboss-eap-6.4.8.CP.zip
Lauch the CLI

    $ ./jboss-cli.sh 

In the cli launch the patch command

    [standalone@localhost:9999 /] patch apply /path/to/downloaded-patch.zip

# Clustering

For the Business Central clustering look at Installation Guide.

Here some basic information to cluster the kie-server:

- same database
- setting Quartz

## Clustering JMS Active MQ on EAP 7

???


<!--cluster jms-->

<pooled-connection-factory name="activemq-ra" transaction="xa" entries="java:/JmsXA java:jboss/DefaultJMSConnectionFactory" connectors="http-connector"/>
                <broadcast-group name="my-broadcast-group" connectors="http-connector" socket-binding="messaging-group"/>
                <discovery-group name="my-discovery-group" refresh-timeout="10000" socket-binding="messaging-group"/>


# Configuration for High Performances

- DisabledFollowOnLockOracle10gDialect

# Integrating SSO

[SSO Tison article](https://github.com/jboss-gpe-ref-archs/bpms_rhsso/blob/master/doc/bpms_rhsso.adoc)


# Problems 
## the workbench does not load

http://stackoverflow.com/questions/137212/how-to-solve-performance-problem-with-java-securerandom

add in `standalone.conf` the following line:

    JAVA_OPTS="$JAVA_OPTS -Djava.security.egd=file:/dev/./urandom"


## Internal git is not accessible

Look for the <system-properties> tag and add the following:

    <property name="org.uberfire.nio.git.daemon.host" value="yourserverdomain"/>
    <property name="org.uberfire.nio.git.ssh.host" value="yourserverdomain"/>

## Internal git offer ssh-dss
Issue https://issues.jboss.org/browse/RHBRMS-243

####workaround: 

Add to ~/.ssh/config the followings:
Host *
VerifyHostKeyDNS no
StrictHostKeyChecking no
HostKeyAlgorithms +ssh-dss
UserKnownHostsFile /dev/null