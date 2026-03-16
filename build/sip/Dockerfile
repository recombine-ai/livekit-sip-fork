# Copyright 2023 LiveKit, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

ARG GOVERSION=1.24

FROM golang:$GOVERSION AS builder

ARG VERSION=0.0.0-dev

WORKDIR /workspace

# install libopus
RUN apt-get update && apt-get install -y pkg-config libopus-dev libopusfile-dev libsoxr-dev

# download go modules
COPY go.mod .
COPY go.sum .
RUN go mod download

# copy source
COPY res/ res/
COPY cmd/ cmd/
COPY pkg/ pkg/
COPY version/ version/

# build
RUN CGO_ENABLED=1 GOOS=linux GOARCH=arm64 GO111MODULE=on go build \
    -ldflags "-X github.com/livekit/sip/version.Version=${VERSION}" \
    -o livekit-sip ./cmd/livekit-sip

FROM debian:trixie-slim

# install wget for health check
RUN apt-get update && \
    apt-get install -y libopus0 libopusfile0 libsoxr0 ca-certificates wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# copy binary
COPY --from=builder /workspace/livekit-sip /bin/

WORKDIR /sip

# run
ENTRYPOINT ["livekit-sip", "--config=/sip/config.yaml"]
