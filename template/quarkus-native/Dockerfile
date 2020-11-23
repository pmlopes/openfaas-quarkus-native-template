# Get a build environment
FROM oracle/graalvm-ce:20.3.0-java11 as build-aot
RUN gu install native-image
# Set working dir
RUN mkdir /app && mkdir -p /tmp/tmpapp
WORKDIR /app

# Add maven wrapper
ADD mvnw /app/
ADD .mvn /app/.mvn/
# extra config
ENV GRAALVM_HOME=$JAVA_HOME
# Cache maven dependency
ADD function/pom.xml /tmp/tmpapp/pom.xml
RUN ./mvnw -f /tmp/tmpapp/pom.xml dependency:resolve
# build the function
ADD function /app/function
RUN ./mvnw -f /app/function/pom.xml package -Pnative


# use the openfaas image to get the watchdog
FROM openfaas/of-watchdog:0.8.1 as watchdog

# Create new image
FROM registry.access.redhat.com/ubi8/ubi-minimal as base

RUN microdnf install shadow-utils
RUN groupadd app && useradd app -g app -d /home/app && mkdir -p /home/app/config
USER app

WORKDIR /home/app

FROM base

COPY --from=build-aot /app/function/target/classes/*.properties /home/app/config/
COPY --from=build-aot /app/function/target/*-runner /home/app/function
COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog

USER root
RUN ln -s /home/app/function /usr/bin/function
USER app

ENV cgi_headers="true"
ENV fprocess="function"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:8000"

ENV exec_timeout="20s"
ENV write_timeout="25s"
ENV read_timeout="25s"


HEALTHCHECK --interval=1s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
