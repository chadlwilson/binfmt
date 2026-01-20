# syntax=docker/dockerfile:1

ARG GO_VERSION=1.25
ARG ALPINE_VERSION=3.23
ARG XX_VERSION=1.9.0

FROM --platform=$BUILDPLATFORM tonistiigi/xx:${XX_VERSION} AS xx

FROM scratch AS src
COPY *.go go.* /

FROM --platform=$BUILDPLATFORM golang:${GO_VERSION}-alpine${ALPINE_VERSION} AS build
COPY --from=xx / /
RUN apk add clang lld file
ARG TARGETPLATFORM
RUN xx-apk add musl-dev linux-headers gcc
WORKDIR /src
RUN XX_CC_PREFER_STATIC_LINKER=1 xx-clang --setup-target-triple
RUN --mount=from=src \
    --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg \
    CGO_ENABLED=1 xx-go test -c -o /out/test -ldflags "-linkmode external -extldflags -static" . && \
    xx-verify --static /out/test

FROM scratch AS binary
COPY --from=build /out/test .

FROM alpine AS run
RUN apk add libcap
COPY --from=binary / /usr/bin
ARG CONFIG_RT_GROUP_SCHED
RUN --security=insecure REEXEC_NAME=/usr/bin/test /usr/bin/test -test.v

FROM binary
