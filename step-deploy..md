# step deploy (advanced github actions)

    name "Step - Deploy"

    on:
        workflow_call:
            inputs:
                env:
                    required: true
                    type: string
            secrets:
                AZURE_CLIENT_ID:
                    required: true
                AZURE_TENANT_ID:
                    required: true
                AZURE_SUBSCRIPTION_ID:
                    required: true

    jobs:
        deploy:
            name: Deploy ${{inputs.env}}
            runs-on: ubuntu latest
            environment: ${{inputs.env}}

    steps:
        - uses: actions/donwload-artifact@v4
            with:
            name: ms-artifact
            path: artifacts/

        - name: Azure Login
            uses: azure/login@v2
            with:
                client-id: ${{secrets.AZURE_CLIENT_ID}}
                tenant-id: ${{secrets.AZURE_TENANT_ID}}
                subscription-id: ${{secrets.AZURE_SUBSCRIPTION_ID}}

        - name: "Deploy to Azure App Service"
            uses: azure/webapps-deploy@v2
            with:
            app-name: app-ms-github-actions-${{inputs.env}}
            package: artifacts/
