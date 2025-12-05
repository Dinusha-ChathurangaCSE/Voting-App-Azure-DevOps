# -------------------------------------------------------
# BUILD STAGE
# -------------------------------------------------------

# If buildx is not used, BUILDPLATFORM will be empty.
# Setting a safe default prevents errors.
ARG BUILDPLATFORM=linux/amd64

FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:7.0 AS build

ARG TARGETPLATFORM=linux/amd64
ARG TARGETARCH=amd64

RUN echo "BUILD PLATFORM : $BUILDPLATFORM"
RUN echo "TARGET PLATFORM: $TARGETPLATFORM ($TARGETARCH)"

WORKDIR /source

COPY *.csproj ./
RUN dotnet restore -a $TARGETARCH

COPY . .
RUN dotnet publish -c Release -a $TARGETARCH \
    --self-contained false --no-restore -o /app

# -------------------------------------------------------
# RUNTIME STAGE
# -------------------------------------------------------
FROM mcr.microsoft.com/dotnet/runtime:7.0

WORKDIR /app
COPY --from=build /app .

ENTRYPOINT ["dotnet", "Worker.dll"]
