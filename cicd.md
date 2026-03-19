# Full ci/cd

name: CI

on:
push:
branches: ["main"]

permissions:
id-token: write
contents: read

jobs:
build:
name: CI
runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up .NET Core
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: 8.0

      - name: dotnet test
        run: dotnet test --configuration Release

      - name: dotnet publish
        run: dotnet publish src/GitHubActionsDotNet.Api/GitHubActionsDotNet.Api.csproj --configuration Release -o artifacts

      - uses: actions/upload-artifact@v4
        with:
          name: ms-artifact
          path: artifacts/

    deploy_dev:
        name: Deploy Dev
        runs-on: ubuntu latest:
        needs: build
        environment: dev

        steps:
            - uses: actions/donwload-artifact@v4
              with:
                name: ms-artifact
                path: artifacts/

            - name: Azure Login
              uses: azure/login@v2
              with:
                client-id:
                tenant-id:
                subscription-id

            - name: "Deploy to Azure App Service"
              uses: azure/webapps-deploy@v2
              with:
                app-name: app-ms-github-actions-dev
                package: artifacts/

    deploy_prod:
        name: Deploy Prod
        runs-on: ubuntu latest:
        needs: deploy_dev
        environment: prod


        steps:
            - uses: actions/donwload-artifact@v4
              with:
                name: ms-artifact
                path: artifacts/

            - name: Azure Login
              uses: azure/login@v2
              with:
                client-id:
                tenant-id:
                subscription-id

            - name: "Deploy to Azure App Service"
              uses: azure/webapps-deploy@v2
              with:
                app-name: app-ms-github-actions-prod
                package: artifacts/
