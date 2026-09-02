# Node.js DevOps portfolio

This project deploys a Node.js application to AWS Elastic Beanstalk, a platform as a service (PaaS). GitHub Actions handles the validation and deployment workflows.

## Architecture and security

The deployment uses GitHub's OpenID Connect (OIDC) integration with AWS instead of long-lived AWS access keys. GitHub Actions presents an OIDC token to AWS Security Token Service (STS). AWS checks the token against the IAM role's trust policy and returns temporary credentials when the repository and branch claims match.

## Deployment flow

1. Pull requests targeting `main` and pushes to `main` run the CI workflow.
2. The CI workflow installs the exact dependency versions from `package-lock.json` with `npm ci`, then runs ESLint, Jest, and a high-severity dependency audit against Node.js 18 and 20.
3. A push to `main` starts the CD workflow. It packages the application source and dependency manifests into a `.zip` file without including `node_modules`.
4. The workflow authenticates to AWS through OIDC and the configured IAM role.
5. Elastic Beanstalk deploys the application package and the workflow sends an HTTP request to the `/health` endpoint as a smoke test.