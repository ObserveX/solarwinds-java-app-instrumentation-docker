

## Instrumenting Pet Clinic app with SolarWinds APM
Petclinic is a [Spring Boot](https://spring.io/guides/gs/spring-boot) application built using Maven. This is a [instrumented version of the original app](https://github.com/spring-projects/spring-petclinic) published by Spring Boot community. 


### Getting Started
```
git clone https://github.com/dockersamples/spring-petclinic.git
cd spring-petclinic
```

### Modify the Dockerfile to include SolarWinds APM Instrumentation steps
In below Dockerfile, we added a RUN command to download the SolarWinds APM agent and an ENTRYPOINT command to include the agent and other options during startup
```
FROM eclipse-temurin:17-jdk-jammy as base
WORKDIR /app
COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:resolve
COPY src ./src

# Download SolarWinds APM agent
RUN curl -sSO https://agent-binaries.cloud.solarwinds.com/apm/java/latest/solarwinds-apm-agent.jar

FROM base as development
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments=-javaagent:/app/solarwinds-apm-agent.jar -Dsw.apm.service.key=SERVICE KEY HERE -Dsw.apm.collector=apm.collector.cloud.solarwinds.com -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"]

FROM base as build
RUN ./mvnw package

FROM eclipse-temurin:17-jre-jammy as production
EXPOSE 8080
COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar
CMD ["java", "-javaagent:/app/solarwinds-apm-agent.jar", "-Dsw.apm.service.key=SERVICE KEY HERE", "-Dsw.apm.collector=apm.collector.cloud.solarwinds.com", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-petclinic.jar"]
```

### Using Docker Compose
```
 docker-compose up -d
```
You can then access petclinic here: http://localhost:8080/. Generate load on application and verify APM data in SolarWinds.


### Steps for Enabling RUM(FrontEnd monitoring) via manual injection

1) On SolarWinds —> Add Data —> Website —> Select Real User Monitoring and provide the Name and URL —> Copy the script. 

```bash
<script src="[https://rum-agent.na-01.cloud.solarwinds.com/ra-e-1570705138407104512.js](https://rum-agent.na-01.cloud.solarwinds.com/ra-e-157070ddddd104512.js)" async></script>
```

2) Use the **`docker exec`**command to start a shell inside the container:

```bash
docker exec -it 6828ced931b2 /bin/bash
```

3) To receive RUM data from your website, add the script immediately before the </body> element on below Konakart web pages.

```bash
PetClinic web pages are located in src/main/resources/templates/
src/main/resources/templates/fragments/layout.html
```

4) exit the container 

```bash
exit
```

5) Restart the container to apply the changes:

```bash
docker restart CONTAINER_ID
docker restart 6828ced931b2 
```

Generate load on application and verify RUM data in SolarWinds.

Commit the changes to your source control system. This step is crucial to ensure that the updated webpages with the SolarWinds JavaScript snippet are preserved across different environments and deployments.
