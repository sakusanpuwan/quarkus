# Quarkus: Supersonice & Subatomic Java

**Quarkus is an open source Java framework built to make applications efficient on the JVM and ready for native execution.** Its build-time approach, curated extensions, and fast development mode make it a strong fit for microservices—especially when startup time and memory per instance matter.

Kubernetes is a common deployment target, not a requirement. You can use Quarkus for long-running services, run it on a standard JVM, and choose native compilation only when it benefits your workload.

**Supersonic** - Describes the fast time to first response when built on Hotspot (JIT) or GraalVM (AOT)
**Subatomic** - Describes the small memory footprint of Quarkus applications

## Benefits
* **Low memory footprint** - Quarkus is designed to be lightweight and efficient, making it suitable for cloud-native applications.
* **Hot reloading (in dev mode)** - If you change a file, Quarkus recompiles and reloads the application in milliseconds.
* **Fast** - Quarkus is optimized for fast startup times and low memory usage, making it ideal for serverless and microservices architectures.
* **Shallow stack traces** (Compared to Spring boot) and comparatively lesser magic than Spring - Quarkus provides a more straightforward and transparent programming model, making it easier to understand and debug applications.
* **Plenty of modules/extensions**. (https://code.quarkus.io) - Quarkus offers a wide range of extensions that provide additional functionality and integrations with popular libraries and frameworks.
* **Compile-time wiring**, (unlike Spring's runtime wiring) - Quarkus performs much of the framework's discovery and wiring at build time, resulting in faster startup times and reduced runtime overhead.
* **Fast startup** (Kubectl deploy restart comes out in less than a sec) - Quarkus is designed to start up quickly, allowing for rapid deployment and scaling of applications in cloud environments. cloud environments.

## How it works
Typical Spring Boot JVM build: The build packages application and dependency bytecode, often in an executable JAR. At runtime, Spring Boot starts on a JVM, discovers and configures components, creates the application context, and then handles requests. Spring’s auto-configuration is convenient, though some of that setup adds startup work.

Quarkus shifts much of the framework's discovery and wiring from startup to **build time**:

1. Add dependencies and Quarkus extensions.
2. During **augmentation**, Quarkus processes annotations, discovers components, and prepares integration code.
3. Package the result as a JVM application or native executable.
4. At runtime, perform the remaining initialization and serve requests.

This build-time work can reduce startup overhead. Extensions connect libraries—REST, databases, messaging, security, and observability—to Quarkus and often supply native-image metadata.

A typical REST request passes through the HTTP layer (based on Vert.x), a Quarkus REST endpoint, application services, and any database or messaging integrations. Quarkus supports both imperative and reactive programming; use blocking work on worker threads, not event-loop threads.

## JVM or native?

| | JVM mode | Native mode |
|---|---|---|
| **What runs** | Java bytecode on a JVM | A platform-specific executable compiled ahead of time |
| **Optimization** | HotSpot can JIT-compile frequently used code while running | Native Image compiles reachable code during the build |
| **Trade-offs** | Portable, familiar tooling, usually faster builds; JVM startup and memory overhead | Often faster startup and lower memory; slower builds, platform-specific output, and possible constraints on reflection or dynamic behavior |
| **Good fit** | General-purpose and long-running services | Workloads where startup, scale-to-zero, or memory limits justify native build costs |

These are separate technologies:

- **Quarkus** is the framework and build tooling.
- **JVM** is the runtime for Java bytecode. **HotSpot** is a widely used JVM implementation.
- **GraalVM** is an extension of the JVM and has the feature to create native binaries for Java application. **Native Image** do not contain bytecode like executable JARs, but instead contain binary with the needed JVM bundled.
- **Mandrel** is a GraalVM-based distribution commonly used for Quarkus native builds.

Quarkus does not require GraalVM: JVM mode works on standard OpenJDK distributions. Native is not automatically faster for every workload; measure startup, memory, throughput, and build time for your application.

## Quarkus and Spring Boot

Both are full-featured Java frameworks with JVM and native deployment options. Choose based on ecosystem, team experience, and workload—not a blanket claim that one is faster.

| | Quarkus | Spring Boot |
|---|---|---|
| **Emphasis** | Build-time augmentation, extensions, containers, and native use cases | Convention-based setup and the broad Spring ecosystem |
| **Programming model** | Jakarta and MicroProfile standards, plus Quarkus APIs | Spring APIs such as Spring DI, MVC/WebFlux, and Data |
| **Development** | Dev mode, live reload, continuous testing | Spring Boot DevTools and Spring tooling |
| **Consider it when** | Its build model, standards, or startup/memory goals fit | Existing Spring skills, integrations, or estate consistency are priorities |

Both frameworks can run on HotSpot and can produce native executables. Real performance depends on application code, configuration, and workload.

## Dependencies and extensions

Use the **Quarkus platform BOM** to keep extension versions compatible. Add only what the application needs, preferably through the Quarkus project generator or Maven/Gradle tooling.

Example Maven dependencies (the BOM version is managed elsewhere in the project):

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>io.quarkus.platform</groupId>
      <artifactId>quarkus-bom</artifactId>
      <version>${quarkus.platform.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest</artifactId>
  </dependency>
  <dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest-jackson</artifactId>
  </dependency>
</dependencies>
```

Artifact names can change across Quarkus releases; use the platform documentation for the version in your project.

## Configuration

The usual configuration file is `src/main/resources/application.properties`. Environment variables and other supported sources can override values for each deployment.

```properties
quarkus.http.port=8080
quarkus.datasource.db-kind=postgresql
quarkus.datasource.username=${DB_USERNAME}
quarkus.datasource.password=${DB_PASSWORD}
quarkus.datasource.jdbc.url=${DB_JDBC_URL}
```

For example, `QUARKUS_HTTP_PORT` sets `quarkus.http.port`. Keep credentials in environment variables or a secret manager, never in source control. Profiles such as `%dev.`, `%test.`, and `%prod.` allow profile-specific values.

Configuration properties have lifecycle phases: **build-time** values affect augmentation and generally require a rebuild; **runtime** values can be set when the application starts; **build-and-run-time-fixed** values must match the build. Check each property's documentation before expecting a packaged application to accept changes.

Common optional extensions include:

- **Health** for liveness and readiness checks.
- **OpenTelemetry** and metrics integrations for observability.
- **Datasource and ORM** integrations for persistence.
- **Security** integrations for authentication and authorization.

Adding a configuration property does not install its extension.

## Develop and package

```bash
./mvnw quarkus:dev       # live development
./mvnw test
./mvnw package           # JVM application
./mvnw package -Dnative  # native executable
```

On Windows, use `mvnw.cmd`. Native builds require a configured native-image toolchain or container build setup. Quarkus also supports Gradle.

**Extensionss** - developed and maintained by Quarkus community, processed at build time.
**Library** - developed and maintained by third party does not natively compile out of the box

## Quick guide

- **Start with JVM mode** for portability, familiar tooling, and general workloads.
- **Choose native mode** when measured startup or memory benefits outweigh build and compatibility costs.
- **Choose Quarkus or Spring Boot** based on the ecosystem and programming model that best fit your service and team.

For exact APIs, extension versions, and configuration behavior, use documentation matching your project's Quarkus release.

## Quarkus CLI

Using the Quarkus CLI is an alternative to Maven or Gradle. It can create a project, add extensions, and run the application in dev mode.

Create, build, and run a Quarkus application:
```bash
quarkus create app
quarkus build
quarkus run
```

Run the JAR directly
```
java -jar target/quarkus-app/quarkus-run.jar
```

Build a native executable and run it
```
quarkus build --native
java -jar target/*-runner.jar
```

Project structure after packaging:
```
/target  
    /quarkus-app
        /lib
            /boot - contains the Quarkus framework libraries
            /main - contains the application code and dependencies
        /app
            code-with-quarkus-1.0.0-SNAPSHOT.jar - contains the application code and dependencies
        quarkus-run.jar - the executable JAR that contains the application and its dependencies
```

`quarkus-run.jar` is an executable JAR that contains the application and its dependencies.

Run tests with:
```
quarkus test
```

Run in dev mode without requiring a rebuild and restart of the application with:
```
quarkus dev
```

In dev mode, view the Dev UI at:
```
http://localhost:8080/q/dev-ui
```