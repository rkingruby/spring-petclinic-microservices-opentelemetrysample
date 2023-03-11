FROM eclipse-temurin:17 as builder
WORKDIR application
ARG ARTIFACT_NAME
COPY ${ARTIFACT_NAME}.jar application.jar
RUN java -Djarmode=layertools -jar application.jar extract

# Download dockerize and cache that layer
ARG DOCKERIZE_VERSION
RUN wget -O dockerize.tar.gz https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz
RUN tar xzf dockerize.tar.gz
RUN chmod +x dockerize

# Add SkyWalking agent
COPY /home/tamilkumaran/Downloads/skywalking-agent/skywalking-agent.jar /application/skywalking-agent.jar
COPY /home/tamilkumaran/Downloads/skywalking-agent/agent.config /application/agent.config /application/config/agent.config
ENV SW_AGENT_NAME=${ARTIFACT_NAME}
ENV SW_AGENT_COLLECTOR_BACKEND_SERVICES=172.17.0.1:11800
ENV JAVA_OPTS="-javaagent:/application/skywalking-agent.jar -Dskywalking.collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES} ${JAVA_OPTS}"

FROM eclipse-temurin:17

WORKDIR application

# Dockerize
COPY --from=builder application/dockerize ./

ARG EXPOSED_PORT
EXPOSE ${EXPOSED_PORT}

ENV SPRING_PROFILES_ACTIVE docker

COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
