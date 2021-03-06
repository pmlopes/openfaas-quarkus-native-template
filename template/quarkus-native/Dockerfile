# Get a build environment
FROM oracle/graalvm-ce:19.3.1-java11 as build-aot
RUN gu install native-image
# Set working dir
RUN mkdir /app
WORKDIR /app

# Add maven wrapper
ADD mvnw /app/
ADD .mvn /app/.mvn/
# extra config
ENV GRAALVM_HOME=$JAVA_HOME
# build the function
ADD function /app/function
RUN ./mvnw -f /app/function/pom.xml package -Pnative


# use the openfaas image to get the watchdog
FROM openfaas/of-watchdog:0.7.6 as watchdog

# Create new image
FROM registry.access.redhat.com/ubi8/ubi-minimal as base

RUN microdnf install shadow-utils
RUN groupadd app && useradd app -g app -d /home/app && mkdir -p /home/app

WORKDIR /home/app
USER app

FROM base

COPY --from=build-aot /app/function/target/*-runner /usr/bin/function
COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog



ENV cgi_headers="true"
ENV fprocess="function"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:8000"

ENV exec_timeout="20s"
ENV write_timeout="25s"
ENV read_timeout="25s"

HEALTHCHECK --interval=1s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
