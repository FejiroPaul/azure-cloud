#
# Build stage
#
FROM maven:3.8.4-openjdk-17-slim AS build

WORKDIR /build

COPY pom.xml .
COPY src ./src

RUN mvn clean package


#
# Package/runtime stage
#
FROM mcr.microsoft.com/openjdk/jdk:17-ubuntu

WORKDIR /app

# Install SSH and curl
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    openssh-server \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Install the Application Insights Java agent
ARG APPLICATIONINSIGHTS_AGENT_VERSION=3.7.9

ADD https://github.com/microsoft/ApplicationInsights-Java/releases/download/${APPLICATIONINSIGHTS_AGENT_VERSION}/applicationinsights-agent-${APPLICATIONINSIGHTS_AGENT_VERSION}.jar /applicationinsights-agent.jar

# Automatically attach Application Insights whenever Java starts
ENV JAVA_TOOL_OPTIONS="-javaagent:/applicationinsights-agent.jar"

# Configure SSH
RUN mkdir -p /run/sshd \
    && echo "root:Docker!" | chpasswd

COPY sshd_config /etc/ssh/sshd_config

# Copy the compiled application and Application Insights configuration
COPY --from=build /build/target/*.jar /app/petstoreapp.jar
COPY applicationinsights.json /app/applicationinsights.json

EXPOSE 2222 8080

# Start SSH and then run the application
ENTRYPOINT ["/bin/bash", "-c", "/usr/sbin/sshd && exec java -jar /app/petstoreapp.jar"]