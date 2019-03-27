## Template: openfaas-quarkus-native-template

The [Quarkus](https://quarkus.io) template uses maven as a build system.

The template makes use of the OpenFaaS incubator project [of-watchdog](https://github.com/openfaas-incubator/of-watchdog).

### Structure

This a template showcasing the use of Native Quarkus as a viable runtime for Serverless functions.

Functions are pure `JAX-RS` routes: 

```java
@Path("/function")
public class GreetingResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "hello OpenFaaS";
    }
}
```

### Trying the template

```
$ faas template pull https://github.com/pmlopes/openfaas-quarkus-native-template
$ faas new --lang quarkus-native <fn-name>
```

### Building

When working in development mode, the java application is build as usual:

```
mvn clean package
```

When going to OpenFAAS, the build is run inside a Docker container using the provided `Dockerfile`.

The Dockerfile performs the following actions:

1. Builds the Java code
2. Generates a native image from the fat jar
3. Creates a new image and install OpenFAAS of-watchdog
4. Copies the native image to the new container
5. Configures the watchdog

You can also run the Dockerfile locally:

```
docker build -t my-quarkus-fn .
docker run --rm -p 8080:8080 my-quarkus-fn
```

You can observe instant startup times and low memory usage:

```
Forking - function []
2019/03/27 12:01:45 Started logging stderr from function.
2019/03/27 12:01:45 Started logging stdout from function.
2019/03/27 12:01:45 OperationalMode: http
2019/03/27 12:01:45 Writing lock-file to: /tmp/.lock
2019/03/27 12:01:45 stdout: 2019-03-27 12:01:45,759 INFO  [io.quarkus] (main) Quarkus 0.12.0 started in 0.003s. Listening on: http://0.0.0.0:8000

2019/03/27 12:01:45 stdout: 2019-03-27 12:01:45,759 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]
```

```
docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
90b8a84d23a9        suspicious_ganguly   0.00%               5.594MiB / 15.54GiB   0.04%               0B / 0B             0B / 0B             20
```

Remember that the memory usage in this case accounts for both the vert.x application and the watchdog!
