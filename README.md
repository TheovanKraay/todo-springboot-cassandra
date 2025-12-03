# Todo Service With Spring Boot

This repository showcases a simple todo application that demonstrates the implementation of a RESTful web service using Spring Boot and Cassandra.

Key technologies and features:

- [**Spring Boot**](https://spring.io/projects/spring-boot): Utilize Spring Boot as the framework for building web applications.
- [**Cassandra**](https://cassandra.apache.org/_/index.html): Utilize Cassandra as data storage.
- [**Cassandra Schema Migration**](https://github.com/patka/cassandra-migration): Automatically manage database schema changes.  
- [**OpenAPI**](https://www.openapis.org/): Define APIs, auto-generate server stubs with bean validation, and produce Swagger documentation.
- [**Docker**](https://docs.docker.com/build/): Containerize the service for deployment.
- [**Docker Compose**](https://docs.docker.com/compose/): Run the todo service docker image together with Cassandra.
- [**Gradle**](https://gradle.org/): Employ Gradle as the build tool.
- [**Lombok**](https://projectlombok.org/): Simplify development with Lombok annotations.

## Local development

### Prerequisites

This application depends on:

- Java 17
- Cassandra (or having docker to run a Cassandra container)

### Run service locally

You need to have Cassandra running locally before you can start the service:

#### Option 1: Using Docker

1. Run cassandra locally in a container `docker run --name cass_cluster -d -p 127.0.0.1:9042:9042 --rm cassandra:4.1.2` 
2. Wait until the container is up and running, then use `cqlsh` to create a Cassandra keyspace for the service:
    ```shell
   docker exec -it cass_cluster cqlsh -e "CREATE KEYSPACE todo
        WITH REPLICATION = { 
         'class' : 'SimpleStrategy', 
         'replication_factor' : 1 
        };"
    ```
3. Create the required tables:
    ```shell
    docker run --rm --network host -v $(pwd)/service/src/main/resources/cassandra/migration/0001_task_tables.cql:/schema.cql cassandra:4.1.2 cqlsh localhost 9042 -f /schema.cql
    ```
4. Start the service with Java 17:
    ```shell
    export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64  # Adjust path for your system
    ./gradlew bootRun
    ```

#### Option 2: Using Local Cassandra Installation

If you have Cassandra installed locally (e.g., in `../cassandra`):

1. Start Cassandra:
    ```shell
    cd ../cassandra
    ./bin/cassandra -f -R > cassandra.log 2>&1 &
    ```
   
2. Wait for Cassandra to start (check with `ss -tuln | grep 9042` or `netstat -tuln | grep 9042`)

3. Create the keyspace using cqlsh via Docker:
    ```shell
    docker run --rm --network host cassandra:4.1.2 cqlsh localhost 9042 -e "CREATE KEYSPACE IF NOT EXISTS todo WITH REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor' : 1};"
    ```

4. Create the tables:
    ```shell
    docker run --rm --network host -v $(pwd)/service/src/main/resources/cassandra/migration/0001_task_tables.cql:/schema.cql cassandra:4.1.2 cqlsh localhost 9042 -f /schema.cql
    ```

5. Start the application with Java 17:
    ```shell
    export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64  # Adjust path for your system
    ./gradlew bootRun
    ```

6. Access the application:
   - **Frontend UI**: http://localhost:8944/
   - **Swagger UI**: http://localhost:8944/swagger-ui/index.html
   - **API**: http://localhost:8944/tasks

### Run service using docker compose

Use this command to run the service together with its dependencies using docker compose `docker compose -f service/docker-compose.yml up`.

### Build docker image

- Run `docker build -t todo-service service` to build the docker image. 
- Run the image `docker run --name todo-service -d -p 127.0.0.1:8944:8944 todo-service:latest`

### Other useful commands 

- Start the service in debug mode to attach a debugger `./gradlew bootRun --debug-jvm`
- List all gradle tasks `./gradlew tasks`
- Generate server stub `./gradlew openApiGenerate`
- Visit [Swagger UI](http://localhost:8944/swagger-ui/index.html) for API documentation
