# Infrastructure and Application Monitoring

## Infrastructure Components Monitoring

### 1. Infrastructure Components

| App Name               | Namespace                   |
|------------------------|-----------------------------|
| Flux                   | flux-system                 |
| Ingress Controller     | ingress-nginx               |
| Karpenter             | Karpenter                  |
| Keycloak               | keycloak                   |
| Krakend                | api-gateway                |
| OAuth2 Proxy           | oauth2-proxy-keycloak      |
| Cert Manager           | cert-manager               |
| External DNS           | aws-external-dns-helm      |

## Application Components Monitoring

### 2. Application Components

| App Name               | Namespace                 |
|------------------------|---------------------------|
| Pygeo API              | pygeoapi                  |
| STAC FastAPI           | stac-fastapi               |
| External Secrets       | external-secrets           |
| DMS Dashboard          | dms-dashboard              |

## Vulnerability Scanning

### 3. Execute Vulnerability Scanner

To maintain the security of our containers, a vulnerability scanner is regularly executed. Critical vulnerabilities will be raised as GitHub issues for immediate attention.

## New Relic Alerts Monitoring

### 4. Monitor Slack Channel for New Relic Alerts

We anticipate setting up New Relic in the coming weeks to monitor system health. L1 support is expected to:

1. **GitHub Issues Creation:**
   - Upon detecting an issue, create a GitHub issue with relevant details.

2. **Troubleshooting:**
   - Log in to the cluster or AWS Console to investigate the issue.
   - Check logs or use New Relic dashboard for detailed information.

3. **Escalation:**
   - If the issue persists, escalate it to level-2 support.
   - Reference the GitHub issue number for effective tracking.

4. **Communication:**
   - Post updates in the designated TLC_Channel for wider visibility.

This process ensures a systematic approach to issue resolution and tracking for L1 support resources.
