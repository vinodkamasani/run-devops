# See https://aka.ms/customizecontainer to learn how to customize the debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

# Build this image from the project folder with:
# docker build -t shoppingclient:local .
# or from the solution root with:
# docker build -f Shopping.Client/Dockerfile -t shoppingclient:local Shopping.Client

# This stage is used when running from VS in fast mode (Default for Debug configuration)
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 8080

# If you want to run the container as a non-root user, define APP_UID and create the user here.
# Example (uncomment and adapt if needed):
# ARG APP_UID=1000
# RUN addgroup --system appgroup && adduser --system --ingroup appgroup --uid $APP_UID app
# USER app


# This stage is used to build the service project
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
# When building with the project folder as the build context this copies the project file into the container
COPY ["Shopping.Client.csproj", "./"]
RUN dotnet restore "./Shopping.Client.csproj"
# Copy everything else and build
COPY . .
RUN dotnet build "Shopping.Client.csproj" -c $BUILD_CONFIGURATION -o /app/build

# This stage is used to publish the service project to be copied to the final stage
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "Shopping.Client.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# This stage is used in production or when running from VS in regular mode (Default when not using the Debug configuration)
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
# Ensure the app listens on the container port when launched by Docker/VS
ENV ASPNETCORE_URLS=http://+:8080
ENTRYPOINT ["dotnet", "Shopping.Client.dll"]