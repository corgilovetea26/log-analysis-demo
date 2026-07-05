# Copyright The OpenTelemetry Authors
# SPDX-License-Identifier: Apache-2.0
# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# https://mcr.microsoft.com/v2/dotnet/sdk/tags/list
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:10.0.301@sha256:ea8bde36c11b6e7eec2656d0e59101d4462f6bd630730f2c8201ed0572b295d5 AS builder
ARG TARGETARCH

WORKDIR /usr/src/app/

COPY ./src/cart/ ./
COPY ./pb/ ./pb/

RUN dotnet restore ./src/cart.csproj -r linux-musl-$TARGETARCH

RUN dotnet publish ./src/cart.csproj -r linux-musl-$TARGETARCH --no-restore -o /cart

# -----------------------------------------------------------------------------

# https://mcr.microsoft.com/v2/dotnet/runtime-deps/tags/list
FROM ghcr.io/grpc-ecosystem/grpc-health-probe:v0.4.52@sha256:0303b506a288d544ed99fef17a55c8c5ce872f498f643c396b532b3595089ac9 AS grpc-health-probe

FROM mcr.microsoft.com/dotnet/runtime-deps:10.0.9-alpine3.23@sha256:297e573be253dafcba4eee8ab29e8e05ab3c5f13f966b3537af2256864de5668

WORKDIR /usr/src/app/
COPY --from=builder /cart/ ./
COPY --from=grpc-health-probe /ko-app/grpc-health-probe /bin/grpc_health_probe

ENV DOTNET_HOSTBUILDER__RELOADCONFIGONCHANGE=false

EXPOSE ${CART_PORT}
ENTRYPOINT [ "./cart" ]
