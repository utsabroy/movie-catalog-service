# Project API Cloud

This is Helm chart created as recommended in [epic-app with substrate](https://confluence.it.epicgames.com/display/CDE/Using+epic-app+with+Substrate#UsingepicappwithSubstrate-Whyuseepic-appandwhatvaluedoesitprovide)

## Prerequisites

The list of tools can be found [here](https://confluence.it.epicgames.com/display/CDE/Prerequisites)

More or less you need.

-   Helm
-   int128/kubelogin/kubelogin
-   kubectl
-   docker
-   [access to substrate cluster](https://confluence.it.epicgames.com/display/CDE/Accessing+Substrate+Infrastructure)

AOP is not so required, but is helpful. Without it you need to get at least `AWS Credentials` and get configuration from your substrate cluster using kubectl.

## Configuration

Before you will deploy the chart make sure that serviceAccount.annotations.eks.amazonaws.com/role-arn in [./values.dev.yaml](./values.dev.yaml) is set. The Role should be deployed prior installing the Helm Chart. Current version of [terraform module](../../../../../../terraform/) does not deploy it. The terraform module to run deploy it is part of [substrate-role module](../../../../../../terraform/common/substrate-role/) and needs to be deployed manually. The role needs to have only trust policy to be service account used by the project-api

```
    "Version" : "2012-10-17",
    "Statement" : [
      {
        "Effect" : "Allow",
        "Principal" : {
          "Federated" : "arn:aws:iam::${data.aws_caller_identity.current.account_id}:oidc-provider/oidc.eks.${data.aws_region.current.name}.amazonaws.com/id/${var.uniqu_hex_id}"
        },
        "Action" : "sts:AssumeRoleWithWebIdentity",
        "Condition" : {
          "StringEquals" : {
            "oidc.eks.${data.aws_region.current.name}.amazonaws.com/id/${var.uniqu_hex_id}:sub" : "system:serviceaccount:${var.substrate_namespace}:${var.service_account_name}",
            "oidc.eks.${data.aws_region.current.name}.amazonaws.com/id/${var.uniqu_hex_id}:aud" : "sts.amazonaws.com"
          }
        }
      }
    ]

```

## Deploy

Currently we are using local version of epic-app localized in [helm/epic](../../../../../../helm/epic-app/). The fork to epic-app has been raised and awaits merge.

Running below might overwrite already existing helm chart. Please make sure that `.env` file in root has been set as required.

To deploy the helm chart

1. You need to [login](https://confluence.it.epicgames.com/display/CDE/Accessing+Substrate+Infrastructure) to substrate cluster.
2. Make sure template is properly set using commands

```bash
make helm/build/project-api-cloud
make helm/template/project-api-cloud
```

3. Then to install (deploy the app)

```bash
make helm/install/project-api-cloud
```

if it was already installed you can upgrade it to apply latest changes

```bash
make helm/upgrade/project-api-cloud
```

## Uninstall

Just run

```bash
make helm/uninstall/project-api-cloud
```
