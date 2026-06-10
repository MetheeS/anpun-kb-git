# devops — cicd

## [github-actions-swa-oidc-not-supported]
created: 2026-06-10
tags: github-actions, azure, oidc, swa, aca, cicd, federated-identity
symptom/context: Setting up GitHub Actions CI/CD for an Azure monorepo (ACA backend +
  Static Web App frontend) using OIDC federated identity to avoid storing secrets.
finding: azure/static-web-apps-deploy@latest does NOT support OIDC federated identity.
  It requires a deployment token regardless of how the workflow authenticates to Azure.
  Retrieve it with:
    az staticwebapp secrets list --name <swa> --resource-group <rg> --query "properties.apiKey" -o tsv
  Store as a GitHub secret (e.g. SWA_DEPLOYMENT_TOKEN) and pass as --deployment-token.
  ACA + ACR DO support OIDC via azure/login@v2:
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
  Required role assignments: AcrPush on ACR, Contributor on resource group.
  OIDC federated credential subject must match exactly (no wildcards on main):
    issuer:   https://token.actions.githubusercontent.com
    subject:  repo:OWNER/REPO:ref:refs/heads/main
    audiences: [api://AzureADTokenExchange]
  Use on.push.paths filters to scope backend/frontend triggers separately in a monorepo.
recommendation: Use OIDC (azure/login@v2) for ACA and ACR jobs; accept the deployment
  token as a stored secret for the SWA deploy job — there is no OIDC path for SWA.
  Use path filters to avoid redundant CI runs on unrelated file changes.
