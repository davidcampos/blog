---
layout: post
title:  "Cassandra with JPA: Achilles vs. Datastax vs. Kundera"
subtitle:  "Comparison of Cassandra JPA libraries usage, performance and resources usage."
date:   2018-11-02 10:00:00 +0100
author: david_campos
tags: cassandra jpa datastax kundera achilles java docker
comments: true
read_time: true
background: '/assets/cassandra-jpa-example/background.jpg'
math: true
---

# TL;DR
Project using JPA to communicate with Cassandra comparing Achilles, Datastax and Kundera libraries. Kundera presents better processing speeds also with lower computational resources usage.

**Source code is available on [Github](https://github.com/hands-on-tech/cassandra-jpa-example){:target="_blank"}** with detailed documentation on how to build and run the tests using Docker.

# Goal
With the overwhelming amounts of data being generated in nowadays technological solutions, one of the main challenges is to find the best solutions to properly store, manage and serve huge amounts of data. Apache Cassandra is one of such solutions, which is a NoSQL database designed for large-scale data management with high availability, consistency and performance. When performing millions of operations per day on top of such databases, every millisecond counts with significant impact on overall system behaviour.

**The main goal of this project is to use different Java libraries to communicate with Cassandra, comparing usage complexity, processing speeds and resources usage**. The following architecture is proposed to achieve the aforementioned goal, which contains the following components and interfaces:
- **Cassandra**: database for large-scale data management;
- **Datastax Native**: Java library to communicate with Cassandra;
- **Datastax ORM**: Java library to communicate with Cassandra;
- **Kundera**: Java library to communicate with Cassandra;
- **Achilles**: Java library to communicate with Cassandra.

![Architecture](/assets/cassandra-jpa-example/architecture.svg){: .image-center}
***Figure:** Illustration of the implementation architecture of the Cassandra and JPA.*

The aforementioned architecture is provided using following technologies:
- **Jave 8**: main programming language for experiment;
- **Maven**: dependency management and package building;
- **Docker**: components containerization, orchestration and setup;
- **Docker Compose**: simplify running multi-container solutions with dependencies.

# Apache Cassandra
- Key concepts
- Performance comparison with other NoSQL databases


![NoSQL Popularity](/assets/cassandra-jpa-example/nosql_popularity.png){: .image-center .image-rounded-corners}
***Figure:** Popularity of several NoSQL databases from [DB-Engines](https://db-engines.com){:target="_blank"}.*

Not straighforward to find isent and transparent comparisons of NoSQL solutions.

Performance comparisons:
-  Çankaya University: https://www.researchgate.net/profile/Murat_Saran/publication/321622083_A_Comparison_of_NoSQL_Database_Systems_A_Study_on_MongoDB_Apache_Hbase_and_Apache_Cassandra/links/5a29173a4585155dd42796db/A-Comparison-of-NoSQL-Database-Systems-A-Study-on-MongoDB-Apache-Hbase-and-Apache-Cassandra.pdf


- Cassandra Frieds: (End Point company)http://www.datastax.com/wp-content/themes/datastax-2014-08/files/NoSQL_Benchmarks_EndPoint.pdf
- Couchbase friends: https://info.couchbase.com/rs/302-GJY-034/images/2018Altoros_NoSQL_Performance_Benchmark.pdf

![NoSQL Performance](/assets/cassandra-jpa-example/nosql_performance.png){: .image-center .image-rounded-corners}
***Figure:** Performance of several NoSQL databases from [EndPoint](http://www.datastax.com/wp-content/themes/datastax-2014-08/files/NoSQL_Benchmarks_EndPoint.pdf){:target="_blank"}.*

Overall, Cassandra better at large-scale and more write than read use cases.






# JPA libraries for Cassandra

The [official Cassandra documentation page](http://cassandra.apache.org/doc/latest/getting_started/drivers.html){:target="_blank"} presents a comprehensive list of available libraries to communicate with Cassandra using Java. A brief analysis shows that only some projects are active and have significant community support:
- **[Achilles](https://github.com/doanduyhai/Achilles){:target="_blank"}**: active project with small community;
- **[Astyanax](https://github.com/Netflix/astyanax){:target="_blank"}**: deprecated and is no longer supported;
- **[Casser](https://github.com/noorq/casser){:target="_blank"}**: very small community and is not clear if project is still active;
- **[Datastax](https://github.com/datastax/java-driver){:target="_blank"}**: active project with large community and enterprise interest;
- **[Kundera](https://github.com/Impetus/Kundera){:target="_blank"}**: active project with large community;
- **[PlayORM](https://github.com/deanhiller/playorm){:target="_blank"}**: specific for Play Framework and project does not look active.

Based on such analysis, **Achilles**, **Datastax** and **Kundera** are the JPA libraries that will be considered during this analysis.

# How to compare JPA libraries?

In order to have a fair performance and resources usage comparison of the several JPA libraries for Cassandra, it is important to consider and analyse several questions in detail, such as:
- Which database operations should be executed and compared?
- What type of data should be considered?
- What is the data complexity?
- What are the relevant performance indicators?
- How to measure and collect the performance indicators?
- How to collect consistent results without interferences and outliers?

Taking the previous topics into consideration, the following testing guidelines were defined:
- **Operation types**: write, read, update and delete;
- **Data type**: single table with simple fields and without relations;
- **Data singularity**: all operations should be performed with unique data values to avoid caching;
- **Performance measurement**: elapsed time to perform each operation;
- **Resources usage measurement**: CPU and RAM usage of client and server applications;
- **Repetition factor**: all tests should be repeated several times to collect average values instead of results from single executions.

A simplistic approach will be followed for the data definition. The following Figure illustrates the "User" class that will be used during the tests, which contains only four textual attributes (unique identifier, first name, last name and city). In summary, everytime an operation is performed, an instance of the User class is being written, read, updated or deleted on Cassandra.

![User](/assets/cassandra-jpa-example/user.svg){: .image-center}
***Figure:** Illustration of the simple "User" class and respective attributes.*

The following pseudocode presents the algorithm applied to collect the processing times for each library and operation types, using a set of users with different attributes. For each library and test cycle, each operation type (write, read, update and delete) will be executed $$O$$ times (TOTAL_OPERATIONS), which is repeated $$R$$ times (TOTAL_REPETITIONS) to calculate the average of total processing times. If multiple cycles are defined, the previous process is repeated $$C$$ times (TOTAL_CYCLES) to collect average values of all repetitions. In the end, average times of all cycles and repetitions are collected per library and operation type. That way, all tasks are repeated to make sure external interferences have no impact on compared processing times.

```ruby
FOR EACH library in [datastax_native, datastax, kundera, achilles]
	FOR EACH cycle in TOTAL_CYCLES (C)
		SET users of size TOTAL_REPETITIONS*TOTAL_OPERATIONS
		FOR EACH operation type in [write, read, update, delete]
			FOR EACH repetition in TOTAL_REPETITIONS (R)
				FOR EACH operation in TOTAL_OPERATIONS (O)
				    GET unique user from users
					CALL operation with unique user instance
					GET operation processing time
				END FOR
				GET total time of all operations
			END FOR
			GET average of repeated total times
		END FOR
		GET average times per operation type
	END FOR
	GET average times per library and operation type
END FOR
```
***Pseudocode:** Algorithm defined to perform JPA libraries tests.*

While executing the operations in the Java application, CPU and RAM resources usage should be collected of both client and server applications. By doing this we are able to evaluate if there is any significant impact of each JPA library on the Java application and Cassandra server resources usage.

# Implementation

To implement 

The Java application implementation was performed minimizing code replication as much as possible.

![Architecture](/assets/cassandra-jpa-example/code_user.svg){: .image-center}
***Figure:** Illustration of the "User" implementation.*

![Architecture](/assets/cassandra-jpa-example/code_run.svg){: .image-center}
***Figure:** Illustration of the "Run" implementation.*


![Architecture](/assets/cassandra-jpa-example/code_main.svg){: .image-center}
***Figure:** Illustration of the "Main" implementation.*

## Cassandra Server

Before starting with implementation details, it is crucial to have a Cassandra server running, towards developing and testing the code. The following Docker Compose YML file is provided to run the Cassandra server with an attached network.

```yml
version: '3.6'

networks:
  bridge:
    driver: bridge

services:
  cassandra:
    image: cassandra:3.11
    environment:
      CASSANDRA_START_RPC: "true"
      CASSANDRA_CLUSTER_NAME: cassandra
    networks:
      bridge:
        aliases:
        - cassandra
```
***Code:** `docker-compose.yml` file for running Cassandra.*

Unfortunately it was not possible to find any good web-based tool to access and manage Cassandra. In order to validate if operations were performed properly, a [RazorSQL](https://razorsql.com){:target="_blank"} trial license was used instead. Let me know if you know any good web-based alternative :blush:.

Finally, the Cassandra server can be started using the `docker compose` tool as following:
```bash
docker-compose up -d
```

## Datastax Native implementation
- Maven Dependency
- Initialization
- Write, Read, Update and Delete operations example

```xml
<dependency>
	<groupId>com.datastax.cassandra</groupId>
	<artifactId>cassandra-driver-core</artifactId>
	<version>3.6.0</version>
</dependency>
```
***Code:** Maven dependency for Datastax Native implementation.*

```java
// Connect
Cluster cluster = Cluster.builder()
		.addContactPoint(Commons.EXAMPLE_CASSANDRA_HOST)
		.build();
Session session = cluster.connect();

// Write
Insert insert = QueryBuilder
		.insertInto("example", "user")
		.value("id", uuid)
		.value("first_name", "John")
		.value("last_name", "Smith")
		.value("city", "London");
session.execute(insert);

// Read
Select.Where select = QueryBuilder
		.select("id", "first_name", "last_name", "city")
		.from("example", "user")
		.where(QueryBuilder.eq("id", uuid));
ResultSet rs = session.execute(select);

// Update
Update.Where update = QueryBuilder
		.update("example", "user")
		.with(QueryBuilder.set("first_name", "___u"))
		.and(QueryBuilder.set("last_name", "___u"))
		.and(QueryBuilder.set("city", "___u"))
		.where(QueryBuilder.eq("id", uuid));
ResultSet rs = session.execute(update);

// Delete
Delete.Where delete = QueryBuilder
		.delete()
		.from("example", "user")
		.where(QueryBuilder.eq("id", uuid));
session.execute(delete);

```
***Code:** Example code to perform connect, write, read, update and delete operations using Datastax Native.*


## Datastax ORM implementation
- Maven Dependency
- User
- Initialization
- Write, Read, Update and Delete operations example

```xml
<dependency>
	<groupId>com.datastax.cassandra</groupId>
	<artifactId>cassandra-driver-mapping</artifactId>
	<version>3.5.1</version>
</dependency>
```
***Code:** Maven dependency for Datastax ORM implementation.*

```java
@Table(keyspace = "example", name = "user",
        readConsistency = "QUORUM",
        writeConsistency = "QUORUM",
        caseSensitiveKeyspace = false,
        caseSensitiveTable = false)
public class UserDatastax implements User {
    @Column(name = "id")
    @PartitionKey
    private UUID id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @Column(name = "city")
    private String city;
...
}
```
***Code:** Datastax User implementation.*

```java
// Connect
Cluster cluster = Cluster.builder()
		.addContactPoint(Commons.EXAMPLE_CASSANDRA_HOST)
		.build();
Session session = cluster.connect();

// Write
UserDatastax user = new UserDatastax(uuid, "John", "Smith", "London");
mapper.save(user);

// Read
UserDatastax user = (UserDatastax) mapper.get(uuid);

// Update
UserDatastax user = users.get(uuid);
user.setFirstName(user.getFirstName() + "___u");
user.setLastName(user.getLastName() + "___u");
user.setCity(user.getCity() + "___u");
mapper.save(user);

// Delete
UserDatastax user = users.get(uuid);
mapper.delete(user);
```
***Code:** Example code to perform connect, write, read, update and delete operations using Datastax ORM.*

## Kundera implementation
- Maven Dependency
- User
- Initialization
- Write, Read, Update and Delete operations example

```xml
<dependency>
	<groupId>com.impetus.kundera.core</groupId>
	<artifactId>kundera-core</artifactId>
	<version>3.13</version>
</dependency>
<dependency>
	<groupId>com.impetus.kundera.client</groupId>
	<artifactId>kundera-cassandra</artifactId>
	<version>3.13</version>
</dependency>
```
***Code:** Maven dependency for Kundera implementation.*

```xml
<persistence xmlns="http://java.sun.com/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
	http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
             version="2.0">
    <persistence-unit name="cassandra_pu">
        <provider>com.impetus.kundera.KunderaPersistence</provider>
        <class>org.davidcampos.cassandra.kundera.UserKundera</class>
        <exclude-unlisted-classes>true</exclude-unlisted-classes>
        <properties>
            <property name="kundera.nodes" value="cassandra"/>
            <property name="kundera.port" value="9160"/>
            <property name="kundera.keyspace" value="example"/>
            <property name="kundera.dialect" value="cassandra"/>
            <property name="kundera.ddl.auto.prepare" value="update"/>
            <property name="kundera.client.lookup.class"
                      value="com.impetus.client.cassandra.thrift.ThriftClientFactory"/>
        </properties>
    </persistence-unit>
</persistence>
```
***Code:** Kundera persistence configuration.*


```java
@Entity
@Table(name = "user", schema = "example@cassandra_pu")
public class UserKundera implements User {
    @Id
    @Column(name = "id")
    private UUID id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @Column(name = "city")
    private String city;
...
}
```
***Code:** Kundera User implementation.*


```java
// Connect
Map<String, String> props = new HashMap<>();
props.put(CassandraConstants.CQL_VERSION, CassandraConstants.CQL_VERSION_3_0);

EntityManagerFactory emf = Persistence.createEntityManagerFactory("cassandra_pu", props);
EntityManager em = emf.createEntityManager();

// Write
UserKundera user = new UserKundera(uuid, "John", "Smith", "London");
em.persist(user);

// Read
UserKundera user = em.find(UserKundera.class, uuid);

// Update
UserKundera user = users.get(uuid);
user.setFirstName(user.getFirstName() + "___u");
user.setLastName(user.getLastName() + "___u");
user.setCity(user.getCity() + "___u");
em.merge(user);

// Delete
UserKundera user = users.get(uuid);
em.remove(user);
```
***Code:** Example code to perform connect, write, read, update and delete operations using Kundera.*

- To turn Kundera logging of, logback.xml on resources folder:

```xml 
<configuration>
    <root level="ERROR"></root>
</configuration>
```

- To create database: `<property name="kundera.ddl.auto.prepare" value="create"/>`
- To work with stored values, remove previous line


Error: RPC true on cassandra:

```shell
Exception in thread "main" com.impetus.kundera.configure.schema.SchemaGenerationException: org.apache.thrift.transport.TTransportException
	at com.impetus.client.cassandra.schemamanager.CassandraSchemaManager.create(CassandraSchemaManager.java:264)
	at com.impetus.kundera.configure.schema.api.AbstractSchemaManager.handleOperations(AbstractSchemaManager.java:264)
	at com.impetus.kundera.configure.schema.api.AbstractSchemaManager.exportSchema(AbstractSchemaManager.java:115)
	at com.impetus.client.cassandra.schemamanager.CassandraSchemaManager.exportSchema(CassandraSchemaManager.java:166)
	at com.impetus.kundera.configure.SchemaConfiguration.configure(SchemaConfiguration.java:191)
	at com.impetus.kundera.configure.ClientMetadataBuilder.buildClientFactoryMetadata(ClientMetadataBuilder.java:48)
	at com.impetus.kundera.persistence.EntityManagerFactoryImpl.configureClientFactories(EntityManagerFactoryImpl.java:408)
	at com.impetus.kundera.persistence.EntityManagerFactoryImpl.configure(EntityManagerFactoryImpl.java:161)
	at com.impetus.kundera.persistence.EntityManagerFactoryImpl.<init>(EntityManagerFactoryImpl.java:135)
	at com.impetus.kundera.KunderaPersistence.createEntityManagerFactory(KunderaPersistence.java:85)
	at javax.persistence.Persistence.createEntityManagerFactory(Persistence.java:79)
	at org.davidcampos.cassandra.Main.main(Main.java:23)
Caused by: org.apache.thrift.transport.TTransportException
	at org.apache.thrift.transport.TIOStreamTransport.read(TIOStreamTransport.java:132)
	at org.apache.thrift.transport.TTransport.readAll(TTransport.java:86)
	at org.apache.thrift.transport.TFramedTransport.readFrame(TFramedTransport.java:129)
	at org.apache.thrift.transport.TFramedTransport.read(TFramedTransport.java:101)
	at org.apache.thrift.transport.TTransport.readAll(TTransport.java:86)
	at org.apache.thrift.protocol.TBinaryProtocol.readAll(TBinaryProtocol.java:429)
	at org.apache.thrift.protocol.TBinaryProtocol.readI32(TBinaryProtocol.java:318)
	at org.apache.thrift.protocol.TBinaryProtocol.readMessageBegin(TBinaryProtocol.java:219)
	at org.apache.thrift.TServiceClient.receiveBase(TServiceClient.java:69)
	at org.apache.cassandra.thrift.Cassandra$Client.recv_execute_cql3_query(Cassandra.java:1734)
	at org.apache.cassandra.thrift.Cassandra$Client.execute_cql3_query(Cassandra.java:1719)
	at com.impetus.client.cassandra.schemamanager.CassandraSchemaManager.onCql3CreateKeyspace(CassandraSchemaManager.java:410)
	at com.impetus.client.cassandra.schemamanager.CassandraSchemaManager.createKeyspace(CassandraSchemaManager.java:317)
	at com.impetus.client.cassandra.schemamanager.CassandraSchemaManager.onCreateKeyspace(CassandraSchemaManager.java:294)
	at com.impetus.client.cassandra.schemamanager.CassandraSchemaManager.createOrUpdateKeyspace(CassandraSchemaManager.java:278)
	at com.impetus.client.cassandra.schemamanager.CassandraSchemaManager.create(CassandraSchemaManager.java:260)
	... 11 more
```


## Achilles implementation
- Maven Dependency
- User
- Initialization
- Write, Read, Update and Delete operations example

```xml
<dependency>
	<groupId>info.archinnov</groupId>
	<artifactId>achilles-core</artifactId>
	<version>6.0.0</version>
</dependency>
```
***Code:** Maven dependency for Achilles implementation.*


```java
@Table(table = "user")
public class UserAchilles implements User {
    @Column(value = "id")
    @PartitionKey
    private UUID id;

    @Column(value = "first_name")
    private String firstName;

    @Column(value = "last_name")
    private String lastName;

    @Column(value = "city")
    private String city;
...
}
```
***Code:** Achilles User implementation.*


```java
// Connect
Cluster cluster = Cluster.builder()
		.addContactPoint(Commons.EXAMPLE_CASSANDRA_HOST)
		.build();

Session session = cluster.connect();

ManagerFactory managerFactory = ManagerFactoryBuilder
		.builder(cluster)
		.withDefaultKeyspaceName("example")
		.doForceSchemaCreation(true)
		.build();

UserAchilles_Manager manager = managerFactory.forUserAchilles();

// Write
UserAchilles user = new UserAchilles(uuid, "John", "Smith", "London");
manager.crud().insert(user).execute();

// Read
UserAchilles user = manager.crud().findById(uuid).get();

// Update
UserAchilles user = users.get(uuid);
user.setFirstName(user.getFirstName() + "___u");
user.setLastName(user.getLastName() + "___u");
user.setCity(user.getCity() + "___u");
manager.crud().update(user).execute();

// Delete
UserAchilles user = users.get(uuid);
manager.crud().delete(user).execute();
```
***Code:** Example code to perform connect, write, read, update and delete operations using Achilles.*

- Achilles integration with IDE and generating mapping sources sucks
	- Reminds me of SOAP times
	- Everytime that you change the POJO classes mapping with the database, you need to rebuild the project to rebuild the classes

## Elapsed time measurement
The measurement of the elapsed time is performed to check the execution of the atomic operation only. This means that the time required to create or get `User` objects will not be considered. In the following code example we can check that a `Stopwatch` is used to measure the elapsed time of the `persist` operation only.

```java
// Get UUID
UUID uuid = Commons.uuids.get(repetition * Commons.OPERATIONS + i);

// Create user
UserKundera user = new UserKundera(
		uuid,
		"John" + i,
		"Smith" + i,
		"London" + i
);
users.put(uuid, user);

// Store user
Commons.resumeOrStartStopWatch(stopwatch);
em.persist(user);
stopwatch.suspend();
```
***Code:** Example code to measure the operation elapsed time.*


## Main implementation

To get everything together, the `Main` application is created to run the tests for each JPA library, considering the configurations provided in environment variables.

```java
public class Main {
    public static void main(final String... args) throws InterruptedException {
        RunDatastaxNative runDatastaxNative = new RunDatastaxNative();
        runDatastaxNative.run();

        RunDatastax runDatastax = new RunDatastax();
        runDatastax.run();

        RunKundera runKundera = new RunKundera();
        runKundera.run();

        RunAchilles runAchilles = new RunAchilles();
        runAchilles.run();
    }
}
```
***Code:** Main program to run the tests for each JPA library.*

## Configurations implementation

The following configurations are required to connect with the Apache Cassandra server and configure the tests properly:
- "EXAMPLE_CASSANDRA_HOST" and "EXAMPLE_CASSANDRA_PORT": Cassandra host and port;
- "EXAMPLE_OPERATIONS": number of operations to run;
- "EXAMPLE_REPETITIONS": number of times to repeat operations execution and average values;
- "EXAMPLE_CYCLES": number of times to repeat tests execution and average values.

Such configurations will be loaded from environment variables using the Commons class, which assumes default values if no environment variables are defined. Moreover, unique identifiers are also generated to perform each operation using an UUID that was never used before, creating `EXAMPLE_OPERATIONS*EXAMPLE_REPETITIONS` unique identifiers.

```java
public final static int OPERATIONS = System.getenv("EXAMPLE_OPERATIONS") != null ?
		Integer.parseInt(System.getenv("EXAMPLE_OPERATIONS")) : 1000;

public final static int REPETITIONS = System.getenv("EXAMPLE_REPETITIONS") != null ?
		Integer.parseInt(System.getenv("EXAMPLE_REPETITIONS")) : 5;

public final static int CYCLES = System.getenv("EXAMPLE_CYCLES") != null ?
		Integer.parseInt(System.getenv("EXAMPLE_CYCLES")) : 5;

public static List<UUID> uuids = generateUUIDs();

public final static String EXAMPLE_CASSANDRA_HOST = System.getenv("EXAMPLE_CASSANDRA_HOST") != null ?
		System.getenv("EXAMPLE_CASSANDRA_HOST") : "cassandra";

public final static String EXAMPLE_CASSANDRA_PORT = System.getenv("EXAMPLE_CASSANDRA_PORT") != null ?
		System.getenv("EXAMPLE_CASSANDRA_PORT") : "9160";
```
***Code:** Commons class to load project configurations from environment variables.*

# Packaging

To build fat JAR file with all dependencies included, the Maven Assembly Plugin was used with the following configurations:

```xml
<build>
	<plugins>
		<plugin>
			<artifactId>maven-assembly-plugin</artifactId>
			<version>3.1.0</version>
			<configuration>
				<descriptorRefs>
					<descriptorRef>jar-with-dependencies</descriptorRef>
				</descriptorRefs>
				<archive>
					<manifest>
						<mainClass>org.davidcampos.cassandra.main.Main</mainClass>
					</manifest>
				</archive>
			</configuration>
			<executions>
				<execution>
					<id>make-assembly</id>
					<phase>package</phase>
					<goals>
						<goal>single</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

Since several classes have the `@Table` Java annotation in the same JAR package, Kundera will consider all annotated classes as persistence entities, which will cause an error similar to:

```
Exception in thread "main" com.impetus.kundera.loader.MetamodelLoaderException: Error while retrieving and storing entity metadata
	at com.impetus.kundera.configure.MetamodelConfiguration.loadEntityMetadata(MetamodelConfiguration.java:238)
	at com.impetus.kundera.configure.MetamodelConfiguration.configure(MetamodelConfiguration.java:112)
	at com.impetus.kundera.persistence.EntityManagerFactoryImpl.configure(EntityManagerFactoryImpl.java:158)
	at com.impetus.kundera.persistence.EntityManagerFactoryImpl.<init>(EntityManagerFactoryImpl.java:135)
	at com.impetus.kundera.KunderaPersistence.createEntityManagerFactory(KunderaPersistence.java:85)
	at javax.persistence.Persistence.createEntityManagerFactory(Persistence.java:79)
	at org.davidcampos.cassandra.kundera.KunderaExample.runWrites(KunderaExample.java:37)
	at org.davidcampos.cassandra.kundera.KunderaExample.main(KunderaExample.java:21)
	at org.davidcampos.cassandra.Main.main(Main.java:12)
```

To fix the error, please make sure Kundera excludes non-expected entity classes, adding the following configuration to the `persistence.xml` file:
```xml
<exclude-unlisted-classes>true</exclude-unlisted-classes>
```

Finally, to build the fat JAR, please run `mvn clean package` in the project folder, which stores the resulting JAR `cassandra-jpa-example-1.0-SNAPSHOT-jar-with-dependencies.jar` in the `target` folder.

# Docker image

To build the Docker Image for the Java application, the following `Dockerfile` was built using the [OpenJDK](https://hub.docker.com/_/openjdk/){:target="_blank"} image as baseline:

```dockerfile
FROM openjdk:8u151-jdk-alpine3.7
MAINTAINER David Campos (david.marques.campos@gmail.com)

# Install Bash
RUN apk add --no-cache bash

# Copy resources
WORKDIR /
COPY wait-for-it.sh wait-for-it.sh
COPY target/cassandra-jpa-example-1.0-SNAPSHOT-jar-with-dependencies.jar cassandra-jpa-example.jar

# Wait for Cassandra and Kafka to be available and run application
CMD ./wait-for-it.sh -s -t 180 $EXAMPLE_CASSANDRA_HOST:$EXAMPLE_CASSANDRA_PORT -- java -Xmx512m -jar cassandra-jpa-example.jar
```
***Code:** Dockerfile to build Java application Docker image.*

[`wait-for-it.sh`](https://github.com/vishnubob/wait-for-it){:target="_blank"} is used to check if a Cassandra host and port is available and only run the Java application when connectivity is established. To **build the docker image**, run the following command in the project folder:

```bash
docker build -t cassandra-jpa-example .
```

# Docker compose
To create the container to run the Java application with the tests, the previous Docker Compose YML file should be extended adding the application configurations. The environment variables that provide the Cassandra and Test configurations are also provided.

```yml
  java:
    image: cassandra-jpa-example
    depends_on:
    - cassandra
    environment:
      EXAMPLE_CASSANDRA_HOST: "cassandra"
      EXAMPLE_CASSANDRA_PORT: "9160"
      EXAMPLE_REQUEST_WAIT: 0
      EXAMPLE_ITERATIONS: 10000
      EXAMPLE_REPETITIONS: 3
    networks:
    - bridge
```
***Code:** Part of `docker-compose.yml` file for running Java application.*


# CPU and RAM usage

To collect resources usage of Cassandra and Java application separately, we decided to take advantage of the [`docker stats`](https://docs.docker.com/engine/reference/commandline/stats/){:target="_blank"} utility, which provides detailed RAM and CPU usage of a target container and also allows to customize the output data format. The following script allows to continuously collect Cassandra's container resources usage and store the results in the TSV file `stats-cassandra.tsv`.

```bash
#!/usr/bin/env bash
{% raw %} while true; do docker stats --no-stream cassandra-jpa-example_cassandra_1 --format "\t{{.MemUsage}}\t{{.MemPerc}}\t{{.CPUPerc}}" | ts >> stats-cassandra.tsv; done {% endraw %}
```
***Code:** Script to collect RAM and CPU usage of a docker container.*

# Run

Now that everything is in place, it is time to start the containers using the `docker-compose` tool, passing the `-d` argument to detach and run the containers in the background:

```bash
docker-compose up -d
```

Such execution will provide detailed feedback regarding the success of creating and running each container and network:

```bash
Creating network "cassandra-jpa-example_bridge" with driver "bridge"
Creating cassandra-jpa-example_cassandra_1 ... done
Creating cassandra-jpa-example_java_1      ... done
```

In order to check if everything is working properly, we can take advantage of the `docker logs` tool to analyse the output being generated on each container.

```shell
docker logs kafka-spark-flink-example_kafka-producer_1 -f
```
    
Output should be similar to the following example:

```bash
wait-for-it.sh: waiting 180 seconds for cassandra:9160
wait-for-it.sh: cassandra:9160 is available after 17 seconds
17:19:24.002 [main] INFO  org.davidcampos.cassandra.datastax_native.RunDatastaxNative - 	WRITE	3	38102	16434	10255	12700.666666666666
17:20:02.617 [main] INFO  org.davidcampos.cassandra.datastax_native.RunDatastaxNative - 	READ	3	35910	14873	10247	11970.0
17:20:35.775 [main] INFO  org.davidcampos.cassandra.datastax_native.RunDatastaxNative - 	UPDATE	3	30508	11592	9240	10169.333333333334
17:21:08.828 [main] INFO  org.davidcampos.cassandra.datastax_native.RunDatastaxNative - 	DELETE	3	30453	10565	9673	10151.0
```

In parallel run `docker-stats-cassandra.sh` and `docker-stats-java.sh` scripts to collect results of CPU and RAM usage on Cassandra and Java application containers. Such measurements are stored in TSV files with the following format:

```bash
Dec 15 16:53:46 	1.06GiB / 1.952GiB	54.29%	138.51%
Dec 15 16:53:48 	1.087GiB / 1.952GiB	55.71%	160.26%
Dec 15 16:53:50 	1.141GiB / 1.952GiB	58.44%	218.72%
Dec 15 16:53:52 	1.137GiB / 1.952GiB	58.25%	180.90%
Dec 15 16:53:54 	1.137GiB / 1.952GiB	58.25%	6.11%
Dec 15 16:53:56 	1.117GiB / 1.952GiB	57.23%	4.69%
```

## Host specifications
The execution of the previously described tests is performed in a computer with the following Hardware and Software specifications:

![Specifications](/assets/cassandra-jpa-example/specifications.png){: .image-center .image-rounded-corners}
***Figure:** Host specifications.*

# Results
Please keep in mind that the results collected are highly related with the pre-conditions previously described, namely:
- Elapsed time measured considering atomic operations only;
- Single thread application to perform operations;
- Tests executed on macOS using a MacBook Pro with 4 cores @ 2,3GHz and 16GB RAM;
- Cassandra and Java Application running on top of Docker;
- CPU and RAM usage collected using `docker stats`.

The results were collected with the following configurations:
- **Number of operations**: 1000, 5000 and 10000;
- **Number of repetitions**: 3;
- **Number of cycles**: 3.

The Figure below presents the average of the measured times for the several libraries and operation types.
Overall, delete operations are the fastest ones, followed by the write tasks. As expected by Cassandra architecture and functionality, read operations are the ones that take longer execution time.
When comparing the used JPA libraries, **Kundera presents the fastest performance times in write, read, update and delete operation types**. On the other hand, Achilles presents the worst results. Comparing the best with the worst library, for 10K operations we have an average difference of 3.2 seconds. If we extrapolate for **10M operations**, this execution **time difference can reach almost 1 hour**.
In average, Kundera performance is 28% better than Achilles, 19% better than Datastax, and 24% better than Native. It is quite interesting to see that Datastax ORM presents similar or better time measurements than Datastax Native. Keep in mind that the low complexity of the `User` data is not adding significant complexity on top of native and ORM solutions.

![Comparison of Cassandra JPA libraries time](/assets/cassandra-jpa-example/results_time.png){: .image-center .image-width-justify-90}
***Figure:** Comparison of Cassandra JPA libraries processing time for the different operation types.*

Jumping into the resources usage analysis, the Figure below presents CPU and RAM usage of the Java application and Cassandra while performing the tests that provided the previously presented results. Overall, **Kundera presents significant lower CPU usages on both Cassandra and Java application**. Regarding RAM usage, there is no significant difference or impact on Cassandra when all JPA libraries are being used. However, Kundera and Achilles seem to use more RAM than Datastax libraries.
For instance, on the 10K operations test, Kundera presents up to 78% less CPU usage on Cassandra, and up to 41% less CPU usage on Java application. Regarding RAM usage, Kundera and Achilles have more 7% of RAM usage than Datastax libraries.
Such differences might related with the fact that **Kundera holds operations on RAM before submitting them to Cassandra**, which has a minor impact on RAM usage but a very significant impact on low CPU usage both on client and server applications. However, it is still open to clarify if a higher complexity on the stored data will have a higher impact on RAM usage.

![Comparison of Cassandra JPA libraries resources](/assets/cassandra-jpa-example/results_resources.png){: .image-center .image-width-justify-90}
***Figure:** Comparison of Cassandra JPA libraries resources usage.*

# Conclusion

In conclusion, Kundera presents up to 28% faster performance results with significant lower CPU impact on both client application and Cassandra server. Such interesting results are significant and should be considered while designing your next Cassandra and Java project, in order to reduce resources usage and increase processing throughput. Nevertheless, do not forget to validate the behavior of Kundera with your specific data characteristics, requirements and complexity.

![Nice](/assets/cassandra-jpa-example/nice.gif){: .image-center}

Please tell me if you had different results using this or other JPA libraries. Your comments, suggestions and contributions are more than welcome. 

**Happy new and techy 2019! :smile: :fireworks:**