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

# Use a specific version of the .NET SDK for building the application
# https://mcr.microsoft.com/v2/dotnet/sdk/tags/list
FROM mcr.microsoft.com/dotnet/sdk:7.0.302@sha256:5c638e77052b5ae4f6f1da3885035b510fc379d2ce4be274c70679114bcdb936 AS builder
WORKDIR /app

# Copy and restore the project dependencies
COPY cartservice.csproj .
RUN dotnet restore cartservice.csproj -r linux-musl-x64

# Copy the remaining source code and publish the application
COPY . .
RUN dotnet publish cartservice.csproj -p:PublishSingleFile=true -r linux-musl-x64 --self-contained true -p:PublishTrimmed=True -p:TrimMode=Link -c Release -o /cartservice --no-restore

# Use a specific version of the runtime dependencies
# https://mcr.microsoft.com/v2/dotnet/runtime-deps/tags/list
FROM mcr.microsoft.com/dotnet/runtime-deps:7.0.4-alpine3.16-amd64@sha256:7141eea9c7be5f4d2f09df427ba37620e50be150fc93015288b3e26c5071af81 AS without-grpc-health-probe-bin
WORKDIR /app

# Copy the published application from the build stage
COPY --from=builder /cartservice .
EXPOSE 7070

# Set environment variables for .NET
ENV DOTNET_EnableDiagnostics=0 \
    ASPNETCORE_URLS=http://*:7070

# Run the application as a non-root user
USER 1000
ENTRYPOINT ["/app/cartservice"]

# Final stage to add the grpc_health_probe binary
FROM without-grpc-health-probe-bin
USER root

# Renovate: datasource=github-releases depName=grpc-ecosystem/grpc-health-probe
ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18
RUN wget -qO /bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

# Revert back to non-root user
USER 1000
