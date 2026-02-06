# Build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copiamos csproj primero para cache de restore
COPY services/JC.LocationIngest/JC.LocationIngest.csproj services/JC.LocationIngest/
RUN dotnet restore services/JC.LocationIngest/JC.LocationIngest.csproj

# Copiamos el resto
COPY . .
RUN dotnet publish services/JC.LocationIngest/JC.LocationIngest.csproj -c Release -o /app/publish

# Runtime
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .

# Cloud Run
ENV ASPNETCORE_URLS=http://0.0.0.0:8080
EXPOSE 8080

ENTRYPOINT ["dotnet", "JC.LocationIngest.dll"]
