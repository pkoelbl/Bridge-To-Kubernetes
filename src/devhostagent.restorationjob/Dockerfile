# Build container
FROM mcr.microsoft.com/dotnet/sdk:7.0-cbl-mariner2.0 AS build

ARG Configuration=Release
ARG TelemetryType=TELEMETRY_DEVELOPMENT
ARG MindaroBuildNumber=0.0

WORKDIR /src/devhostagent.restorationjob
COPY /src/devhostagent.restorationjob/devhostAgent.restorationjob.csproj .
COPY /src/common/common.csproj /src/common/
RUN dotnet restore
COPY /src/devhostagent.restorationjob/ .
COPY /src/common/ /src/common/
COPY /build/ /build/
ENV TelemetryType=${TelemetryType}
ENV MINDARO_BUILD_NUMBER=${MindaroBuildNumber}
ENV PATH="${PATH}:/sbin"

RUN dotnet publish -c ${Configuration} -o /output devhostAgent.restorationjob.csproj

# Final container
FROM mcr.microsoft.com/dotnet/aspnet:7.0-cbl-mariner2.0 as final

# Setup common tools
RUN tdnf clean all && \
  tdnf check-update && \
  tdnf upgrade -y

RUN tdnf install -y \
  procps \
  bind-utils

RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl"
RUN install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

WORKDIR /app
ENTRYPOINT ["/app/restorationjob"]

COPY --from=build /output /app