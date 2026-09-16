#
# Build stage
#
FROM maven:3.6.0-jdk-8-slim AS build

WORKDIR /build

COPY pom.xml .
COPY src ./src

RUN mvn clean package


#
# Package/runtime stage
#
FROM eclipse-temurin:8-jre-alpine

WORKDIR /app

# Install the Application Insights Java agent
ARG APPLICATIONINSIGHTS_AGENT_VERSION=3.7.9

ADD https://github.com/microsoft/ApplicationInsights-Java/releases/download/${APPLICATIONINSIGHTS_AGENT_VERSION}/applicationinsights-agent-${APPLICATIONINSIGHTS_AGENT_VERSION}.jar /applicationinsights-agent.jar

# Automatically attach Application Insights whenever Java starts
ENV JAVA_TOOL_OPTIONS="-javaagent:/applicationinsights-agent.jar"

# Copy the application and Application Insights configuration
COPY --from=build /build/target/*.jar /app/petstorepetservice.jar
COPY applicationinsights.json /app/applicationinsights.json

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app/petstorepetservice.jar"]