FROM openjdk:11-jre-slim-buster

ARG JAR_FILE=target/*.jar
ARG OLTP=opentelemetry/opentelemetry-javaagent.jar

COPY ${JAR_FILE} /usr/src/myapp/api-cursos.jar 
COPY ${OLTP} /usr/src/myapp/opentelemetry-javaagent.jar 

WORKDIR /usr/src/myapp

RUN rm -f /etc/localtime && \
    ln -s /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime

CMD ["java","-Duser.timezone=UTC-3", "-Xms256M","-Xmx256M","-XX:PermSize=128m","-XX:MaxPermSize=256m","-javaagent:opentelemetry-javaagent.jar","-jar","api-cursos.jar"]
