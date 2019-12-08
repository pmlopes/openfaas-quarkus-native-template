# Get a build environment
FROM oracle/graalvm-ce:19.2.1 as build-aot
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


FROM openfaas/of-watchdog:0.7.2 as watchdog


# At this point the native image is built

# Create new image from alpine
FROM frolvlad/alpine-glibc:alpine-3.10
RUN apk add --no-cache ca-certificates

RUN addgroup -S app && adduser -S -g app app

RUN apk --no-cache add ca-certificates curl
# Alternatively use ADD https:// (which will not be cached by Docker builder)



WORKDIR /root/

RUN mkdir -p /home/app

# Wrapper/boot-strapper
WORKDIR /home/app
# Copy the SUNEC lib from graal to the final image
COPY --from=build-aot /opt/graalvm-ce-19.2.1/jre/lib/amd64/libsunec.so /usr/lib/libsunec.so
# Copy generated native executable from build-aot
COPY --from=build-aot /app/function/target/*-runner /usr/bin/function
COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
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