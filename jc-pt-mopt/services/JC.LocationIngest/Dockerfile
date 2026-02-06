# Build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copiamos csproj primero (cache restore)
COPY services/JC.LocationIngest/JC.LocationIngest.csproj ./services/JC.LocationIngest/
RUN dotnet restore ./services/JC.LocationIngest/JC.LocationIngest.csproj

# Copiamos el resto del código
COPY . .

# Publicamos desde el folder del proyecto (más estable)
WORKDIR /src/services/JC.LocationIngest
RUN dotnet publish JC.LocationIngest.csproj -c Release -o /app/publish

# Runtime
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .

ENV ASPNETCORE_URLS=http://0.0.0.0:8080
EXPOSE 8080

ENTRYPOINT ["dotnet", "JC.LocationIngest.dll"]
