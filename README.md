# dataphos-helm

This repository provides custom HELM charts designed to simplify the deployment and maintenance of Dataphos products. The goal is to streamline Kubernetes manifest configuration for quick cluster deployments, without relying on IaC tools or broader infrastructure frameworks.

## Chart Methodology

The charts have not been designed to serve as deployments of a single component instance, but rather a management system of several instances of a single component.

For instance, while you may deploy only a single Schema Registry instance in a cluster, the Schema Registry Validator and Persistor charts have been structured in a way to enable the user to manage several at the same time through a single `values.yaml` file. This is particularly useful in the case when managing the Persistor, Indexer and Resubmitter components, whose relationship and interconnectedness may vary depending on the use-case.

Individual releases should be managed on a namespace level.

Secrets referenced by the configuration files (service accounts, connection strings, etc.) are created if the appropriate secret data is provided in the chart's `values.yaml` file. They can also be deployed independently.

## Prerequisites

* [Helm](https://helm.sh/)

## Chart List

| Chart Name | Description |
| ---------- | ----------- |
| `dataphos-persistor` | The chart for the Dataphos Persistor. |
| `dataphos-schema-registry` | The chart for the Dataphos Schema Registry database and service. |
| `dataphos-schema-registry-validator` | The chart for the Dataphos Schema Registry Validator. |

## Usage

Each chart has its own configuration settings outlined in its respective subfolder. A `values.yaml` file should be prepared and pass to Helm while performing the installation.

```
helm install <release-name> <path-to-chart>
```

For example, if we want to deploy the `dataphos-persistor` chart, we would run:

```
helm install persistor ./dataphos-persistor
```

This would cause the `values.yaml` filed to be read from the root directory of the `dataphos-persistor` folder. The `--values` flag may be passed in the call to override this behavior.

You can also add a `--dry-run` flag that will simply generate the Kubernetes manifests and check if they are valid (note that this requires `kubectl` to be configured against an actual cluster). For general linting of the Helm templates, run `helm lint`. 

If you happen to be using VS Code make sure to have the Kubernetes and Helm extensions installed to make life a little easier for you.

## Verify Docker Images
For security reasons, all of the Docker images used in the repository are digitally signed using [sigstore's cosign](https://github.com/sigstore/cosign), in a keyless manner.

If you wish to verify the signature on any of the images, be sure to run a version of the following command mentioned [here](https://docs.sigstore.dev/cosign/verifying/verify/#keyless-verification-using-openid-connect):
```linux
cosign verify <image URI> --certificate-identity=name@example.com
                          --certificate-oidc-issuer=https://accounts.example.com
```

Since the keyless signing was performed as a step in a GitHub Actions workflow, the certificate OIDC issuer in that case will be `https://token.actions.githubusercontent.com`, while the certificate identity will have the following format:
```linux
https://github.com/{user-or-org}/{repo}/.github/workflows/{workflow-filename}.yaml@refs/heads/{branch}
```

Considering all of that, the signature verification, e.g the Persistor Docker image `syntioinc/persistor:<tag>` on the `main` GitHub branch, would look something like this:
```linux
cosign verify syntioinc/persistor:<tag> 
--certificate-identity=https://github.com/dataphos/dataphos-helm/.github/workflows/push.yaml@refs/heads/main 
--certificate-oidc-issuer=https://token.actions.githubusercontent.com
```