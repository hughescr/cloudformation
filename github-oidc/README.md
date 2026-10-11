GitHub OIDC Provider Template
=============================

Creates the account's single IAM OIDC identity provider for GitHub Actions (`https://token.actions.githubusercontent.com`, audience `sts.amazonaws.com`) and exports its ARN as `GitHubOIDCProvider-Arn`. Site stacks import that export in an IAM role trust policy so GitHub Actions can deploy with short-lived credentials instead of stored AWS keys. See the `Description` in `github-oidc-provider.yml` for the role snippet.

An account can have only one provider for this URL, so deploy this stack once and share it. If the account already has a GitHub provider (created by hand or by another tool), the deploy fails with `EntityAlreadyExists`; delete that provider or import it first.

Region
------

Exports are per-region, so the importing site stack must be in the same region as this one. The rungie.com site stack deploys to `us-west-2` (`provider.region` in `webservices/serverless.yml`), so deploy this there. IAM itself is global, so the provider is usable from every region, but only stacks in `us-west-2` can `!ImportValue` the ARN.

Deploy (one time)
-----------------

```bash
aws cloudformation deploy \
  --template-file github-oidc/github-oidc-provider.yml \
  --stack-name github-oidc-provider \
  --region us-west-2
```

Verify
------

```bash
aws cloudformation list-exports --region us-west-2 \
  --query "Exports[?Name=='GitHubOIDCProvider-Arn']"
```

Who uses it / deleting
----------------------

CloudFormation refuses to delete this stack while any stack imports the export. List the importers with:

```bash
aws cloudformation list-imports --region us-west-2 --export-name GitHubOIDCProvider-Arn
```

Delete the stack only when that list is empty.
