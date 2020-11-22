# Get a build environment
FROM oracle/graalvm-ce:20.3.0-java11 as build-aot
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
FROM openfaas/of-watchdog:0.8.1 as watchdog

# Create new image
FROM registry.access.redhat.com/ubi8/ubi-minimal as base

RUN mkdir -p /newtmp && chmod -R 777 /newtmp

FROM scratch as base_minimal

COPY --from=base /newtmp /tmp

# libraries for quarkus
COPY --from=base "/lib64/libstdc++.so.6" \
    /lib64/libm.so.6 \
    /lib64/libpthread.so.0 \
    /lib64/libdl.so.2 \
    /lib64/libz.so.1 \
    /lib64/librt.so.1 \
    /lib64/libc.so.6 \
    /lib64/ld-linux-x86-64.so.2 \
    /lib64/libgcc_s.so.1 /lib64/


FROM base_minimal

RUN mkdir /config

COPY --from=build-aot /app/function/target/classes/*.properties /config/
COPY --from=build-aot /app/function/target/*-runner /function
COPY --from=watchdog /fwatchdog /fwatchdog

ENV cgi_headers="true"
ENV fprocess="/function"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:8000"

ENV exec_timeout="20s"
ENV write_timeout="25s"
ENV read_timeout="25s"

HEALTHCHECK --interval=2s CMD ["/fwatchdog", "healthcheck"]

CMD ["/fwatchdog"]
