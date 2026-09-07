FROM golang:1.26.2-trixie AS builder

ARG F5GC_MODULE
ARG TAG

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get -y install git gcc cmake autoconf libtool pkg-config libmnl-dev libyaml-dev apt-transport-https ca-certificates

RUN apk add --no-cache tini

RUN mkdir -p /tmp && cd /tmp && \
    git clone --recursive -b ${TAG} -j `nproc` https://github.com/free5gc/free5gc.git && \
    cd free5gc && \ 
    make ${F5GC_MODULE}


# Alpine is used for debug purpose. You can use scratch for a smaller footprint.
FROM alpine:3.15

ARG F5GC_MODULE
ARG DEBUG_TOOLS

# Install debug tools ~ 100MB (if DEBUG_TOOLS is set to true)
RUN if [ "$DEBUG_TOOLS" = "true" ] ; then apk add -U vim strace net-tools curl netcat-openbsd ; fi

# Set working dir
WORKDIR /free5gc
RUN mkdir -p config/ log/ cert/

# Copy executable and default certs
COPY --from=builder /sbin/tini /sbin/tini
COPY --from=builder /tmp/free5gc/${F5GC_MODULE} ./
COPY --from=builder /tmp/free5gc/cert/${F5GC_MODULE}.pem ./cert/
COPY --from=builder /tmp/free5gc/cert/${F5GC_MODULE}.key ./cert/
