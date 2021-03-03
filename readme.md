
# Very Simple Bootable Wildfly with a Basic MDB in an EAR & Hollowed Jar Connected to AMQ 5.x via  Moduled RAR

## Description
Another bootable WF example.  

Part of motivation for this is that the EAR should function same in BWF or WF, right?   I noticed that there was a small difference in WF vs BWF.   This sample maybe can be used to illustrate that.   


## Interesting Things in this POC
* JDK 11
* Gradle 6.8.3; multiple subprojects
* Properties File use on startup
* RAR is deployed as WF Module -- allowing for hollowed jar
* EAR & MDB separated having little to do with BWF or WF for that matter
* Bootable Wildfly 22  & [Galleon/Bootable](https://docs.wildfly.org/22/Bootable_Guide.html)
* Maven Wrapper as called by gradle 
* Can MDB can be deployed in WF too (well, except for issue mentioned above). 

## Building & Running
Maven activities are called from gradle script.  Standard gradle like targets work, so:

	./gradlew clean build

will build everything; the resulting JAR is directly runnable.   Create or edit a properties files as per wildfly/src/main/resources/Silvertip.dev.properties

Running from CLI is simple; set some environment variables and use java -jar.   Example :

	  /opt/jdk11/bin/java -jar wildfly/target/RunnableMDBWithModuledRAR-bootable.jar  --deployment=ear/build/libs/Silvertip.ear  --properties wildfly/src/main/resources/Silvertip.dev.properties 

Toss a text message  at the dev AMQ queue simpleMDBTestQueue and then check server log output.

#### So here is a transcript of what I did to deploy this to a normal WF instance.

```
[fwelland@wellandf3 galleon]$ java -version
openjdk version "11.0.8" 2020-07-14
OpenJDK Runtime Environment AdoptOpenJDK (build 11.0.8+10)
OpenJDK 64-Bit Server VM AdoptOpenJDK (build 11.0.8+10, mixed mode)
[fwelland@wellandf3 galleon]$ ./do.sh run 
[galleon]$ install wildfly:current   --dir=/opt/wf22-clean
Feature pack installed.d. 
======= ============ ============== 
Product Build        Update Channel 
======= ============ ============== 
wildfly 22.0.1.Final current        
#
# FYI:   galleon-cli-4.2.8.Final-SNAPSHOT.jar 
#
cd /opt/wf22-clean/bin
#
# NOTE:  ~/JavaStuff/Projects/fhw/BWF-MDB-EAR-ModuledRAR is this project's root folder on my workstation. 
#
./standalone.sh --properties ~/JavaStuff/Projects/fhw/BWF-MDB-EAR-ModuledRAR/wildfly/src/main/resources/Silvertip.dev.properties 
#
# Opened another terminal window here
#
cd /opt/wf22-clean/modules/system/layers/base/org/apache/
#
# 'installing'  the same exact rar-module from BWF project 
#
cp -a ~/JavaStuff/Projects/fhw/BWF-MDB-EAR-ModuledRAR/wildfly/amq-rar/modules/system/layers/base/org/apache/amq-rar . 
cd /opt/wf22-clean/bin
./jboss-cli.sh --connect
[standalone@localhost:9990 /] reload 
[standalone@localhost:9990 /] run-batch --file=~/JavaStuff/Projects/fhw/BWF-MDB-EAR-ModuledRAR/wildfly/src/main/resources/configure-wildfly.cli 
The batch executed successfully
[standalone@localhost:9990 /] reload 
[standalone@localhost:9990 /] deploy ~/JavaStuff/Projects/fhw/BWF-MDB-EAR-ModuledRAR/ear/build/libs/Silvertip.ear 
{"WFLYCTL0062: Composite operation failed and was rolled back. Steps that failed:" => {"Operation step-2" => {"WFLYCTL008
0: Failed services" => {"jboss.deployment.subunit.\"Silvertip.ear\".\"Silvertip.jar\".POST_MODULE" => "WFLYSRV0153: Failed to proces
s phase POST_MODULE of subdeployment \"Silvertip.jar\" of deployment \"Silvertip.ear\"
    Caused by: java.lang.NoClassDefFoundError: Failed to link fhw/SimpleMDB (Module \"deployment.Silvertip.ear.Silvertip.jar\" from 
Service Module Loader): javax/jms/MessageListener"},"WFLYCTL0412: Required services that are not installed:" => ["jboss.deployment.u
nit.\"Silvertip.ear\".beanmanager","jboss.deployment.unit.\"Silvertip.ear\".WeldStartService"],"WFLYCTL0180: Services with missing/u
navailable dependencies" => ["jboss.deployment.unit.\"Silvertip.ear\".weld.weldClassIntrospector is missing [jboss.deployment.unit.\
"Silvertip.ear\".WeldStartService, jboss.deployment.unit.\"Silvertip.ear\".beanmanager]"]}}}
#
# Go edit mdb.jar file's manifest to include "Dependencies: javax.jms.api"  and then rebuild the mdb.jar and ear (see more below)
#
[standalone@localhost:9990 /] deploy ~/JavaStuff/Projects/fhw/BWF-MDB-EAR-ModuledRAR/ear/build/libs/Silvertip.ear 
# 
#Works!!!!
#
```

Send a message to broker/queue; works that same at the BWF example. 

The jms manifest thing can be achieved by uncommenting a few lines in the build.gradle:  <root>/mdb/build.gradle.    Hopefully pretty straight forward what to do. 

**NOTE:**   BWF works with or without the manifest dependency in place.  

**NOTE 2:**   How do I know about the jms/depedency thing?   I've ported several MDB.ears from Jboss EAP 5/6 to JBOSS EAP 7x.   This was needed.   I suppose running standalone-full  (for jboss EAP 7.x) would change the equation; but I didn't want all the extra stuff that was part of standalone-full. 

