Spring Music
============

This is a sample application for using database services on [Cloud Foundry](http://cloudfoundry.org) with the [Spring Framework](http://spring.io) and [Spring Boot](http://projects.spring.io/spring-boot/).

This application has been built to store the same domain objects in one of a variety of different persistence technologies - relational, document, and key-value stores. This is not meant to represent a realistic use case for these technologies, since you would typically choose the one most applicable to the type of data you need to store, but it is useful for testing and experimenting with different types of services on Cloud Foundry.

The application use Spring Java configuration and [bean profiles](http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html) to configure the application and the connection objects needed to use the persistence stores. It also uses the [Java CFEnv](https://github.com/pivotal-cf/java-cfenv/) library to inspect the environment when running on Cloud Foundry. See the [Cloud Foundry documentation](http://docs.cloudfoundry.org/buildpacks/java/spring-service-bindings.html) for details on configuring a Spring application for Cloud Foundry.

## Building

This project requires Java version 17 or later to compile.

> [!NOTE]
> If you need to use an earlier Java version, check out the [`spring-boot-2` branch](https://github.com/cloudfoundry-samples/spring-music/tree/spring-boot-2), which can be built with Java 8 and later.  

To build a runnable Spring Boot jar file, run the following command:

~~~
$ ./gradlew clean assemble
~~~

## Running the application locally

One Spring bean profile should be activated to choose the database provider that the application should use. The profile is selected by setting the system property `spring.profiles.active` when starting the app.

The application can be started locally using the following command:

~~~
$ java -jar -Dspring.profiles.active=<profile> build/libs/spring-music-1.0.jar
~~~

where `<profile>` is one of the following values:

* `mysql`
* `postgres`
* `mongodb`
* `redis`

If no profile is provided, an in-memory relational database will be used. If any other profile is provided, the appropriate database server must be started separately. Spring Boot will auto-configure a connection to the database using it's auto-configuration defaults. The connection parameters can be configured by setting the appropriate [Spring Boot properties](http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html).

If more than one of these profiles is provided, the application will throw an exception and fail to start.

## Running the application on Cloud Foundry

When running on Cloud Foundry, the application will detect the type of database service bound to the application (if any). If a service of one of the supported types (MySQL, Postgres, Oracle, MongoDB, or Redis) is bound to the app, the appropriate Spring profile will be configured to use the database service. The connection strings and credentials needed to use the service will be extracted from the Cloud Foundry environment.

If no bound services are found containing any of these values in the name, an in-memory relational database will be used.

If more than one service containing any of these values is bound to the application, the application will throw an exception and fail to start.

After installing the 'cf' [command-line interface for Cloud Foundry](http://docs.cloudfoundry.org/cf-cli/), targeting a Cloud Foundry instance, and logging in, the application can be built and pushed using these commands:

~~~
$ cf push
~~~

The application will be pushed using settings in the provided `manifest.yml` file. The output from the command will show the URL that has been assigned to the application.

### Creating and binding services

Using the provided manifest, the application will be created without an external database (in the `in-memory` profile). You can create and bind database services to the application using the information below.

#### System-managed services

Depending on the Cloud Foundry service provider, persistence services might be offered and managed by the platform. These steps can be used to create and bind a service that is managed by the platform:

~~~
# view the services available
$ cf marketplace
# create a service instance
$ cf create-service <service> <service plan> <service name>
# bind the service instance to the application
$ cf bind-service <app name> <service name>
# restart the application so the new service is detected
$ cf restart
~~~

#### User-provided services

Cloud Foundry also allows service connection information and credentials to be provided by a user. In order for the application to detect and connect to a user-provided service, a single `uri` field should be given in the credentials using the form `<dbtype>://<username>:<password>@<hostname>:<port>/<databasename>`.

These steps use examples for username, password, host name, and database name that should be replaced with real values.

~~~
# create a user-provided Oracle database service instance
$ cf create-user-provided-service oracle-db -p '{"uri":"oracle://root:secret@dbserver.example.com:1521/mydatabase"}'
# create a user-provided MySQL database service instance
$ cf create-user-provided-service mysql-db -p '{"uri":"mysql://root:secret@dbserver.example.com:3306/mydatabase"}'
# bind a service instance to the application
$ cf bind-service <app name> <service name>
# restart the application so the new service is detected
$ cf restart
~~~

#### Changing bound services

To test the application with different services, you can simply stop the app, unbind a service, bind a different database service, and start the app:

~~~
$ cf unbind-service <app name> <service name>
$ cf bind-service <app name> <service name>
$ cf restart
~~~

#### Database drivers

Database drivers for MySQL, Postgres, Microsoft SQL Server, MongoDB, and Redis are included in the project.

To connect to an Oracle database, you will need to download the appropriate driver (e.g. from http://www.oracle.com/technetwork/database/features/jdbc/index-091264.html). Then make a `libs` directory in the `spring-music` project, and move the driver, `ojdbc7.jar` or `ojdbc8.jar`, into the `libs` directory.
In `build.gradle`, uncomment the line `compile files('libs/ojdbc8.jar')` or `compile files('libs/ojdbc7.jar')` and run `./gradle assemble`.


## Alternate Java versions

By default, the application will be built and deployed using Java 17 compatibility.
If you want to use a more recent version of Java, you will need to update two things.

In `build.gradle`, change the `targetCompatibility` Java version from `JavaVersion.VERSION_17` to a different value from `JavaVersion`:

~~~
java {
  ...
  targetCompatibility = JavaVersion.VERSION_17
}
~~~

In `manifest.yml`, change the Java buildpack JRE version from `version: 17.+` to a different value:

~~~
    JBP_CONFIG_OPEN_JDK_JRE: '{ jre: { version: 17.+ } }'
~~~




## Running the application on Tanzu platform on K8(TPK8).

Use the Following `tanzu` commands to deploy app on TPK8.

**Note** : If you are on using broadcom machine , disable `Symantec wss agent temporarily` to allow push app image into your image registry.

**Prerequisite** before deploying `spring-music` app on Tanzu platform space. 
 - Create a space with spring-dev and custom networking profile. Refer to the [create app environment instructions](https://docs.vmware.com/en/VMware-Tanzu-Platform/SaaS/create-manage-apps-tanzu-platform-k8s/getting-started-create-app-envmt.html) 
 - Download `tanzu cli` and its developer plugins.

```
Deployment instructions :

#Checkout spring music app git repo 
git clone https://github.com/sendjainabhi/spring-m-2.git

#switch to spring-music directory 
cd spring-m-2

#login to tanzu platform
tanzu login

#switch to project 
tanzu project use <your tanzu platform project name>

#switch to space 
tanzu space use <your tanzu platform space name>

#Set java jvm env var to 17 , tpk8 default is jvm is 11 
tanzu app config build non-secret-env set BP_JVM_VERSION=17

#tanzu app init , refer following config param.
 tanzu app init
? What is your app's name? spring-m-2
? Would you like to replace all existing content of the current spring-m-2.yml? Yes
? Which directory contains your app's source code? .
? Select container build type to use for this app: buildpacks
? Should your app be accessible through HTTP? Yes

✓ Created tanzu.yml
✓ Recorded app configuration to .tanzu/config/spring-m-2.yml

#set tanzu registry for container app  (used git hub container registry, you can select of your choice )
tanzu build config --containerapp-registry ghcr.io/<git use name>/spring-m-2

#Docker login for app image push
docker login --username <git user name> --password <git token> ghcr.io

#create a directory dev and assign permission to write
mkdir dev 
chmod 777 dev

#tanzu build and push app image command 
tanzu build --output-dir dev

#tanzu deploy from local directory command 
tanzu deploy --from-build dev

#tanzu deploy for build and deploy app together
tanzu deploy 

Note - change the package visibility to public in GitHub registry,otherwise it will block kapp controller to pull your app image on space. 

#to deploy multiple apps on 1 space without overwriting use command 
tanzu deploy --patch 

#Show the services in space
tanzu services list   

# list all the available services that you can dynamically create
tanzu services type list

#create an instance of MySQL named spring-m2-db
tanzu service create MySQLInstance/spring-m2-db

#bind the db to the spring-m-2  app
tanzu service bind MySQLInstance/spring-m2-db ContainerApp/spring-m-2


#show that the app is bound to the database (if deploying, discussing that it's automatically restarting)
tanzu app list 

# show how to scale the app by adding replicas or changing cpu/mem
tanzu app scale 

```