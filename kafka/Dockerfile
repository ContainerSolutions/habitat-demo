FROM alpine:3.5

ARG kafa_version=0.10.1.1
ARG scala_version=2.10
ARG kafka_url=http://apache.mirror.triple-it.nl

WORKDIR /opt
RUN apk add --no-cache -t build-deps ca-certificates wget tar \
		&& wget -O - "$kafka_url/kafka/$kafa_version/kafka_${scala_version}-${kafa_version}.tgz" \
		| tar --transform='s|'"kafka_$scala_version-$kafa_version"'/|kafka/|' -xzf - \
		&& rmdir kafka_* \
		&& apk del build-deps

ENV PATH "$PATH:/opt/kafka/bin"

RUN apk add --no-cache openjdk8-jre-base bash
