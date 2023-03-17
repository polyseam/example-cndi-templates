# example-cndi-templates

## Wbat is a CNDI Template?

A [cndi](https://github.com/polyseam/cndi) Template is a JSON file that CNDI can parse in order to provide a simplified deployment experience for some cloud-native application. The template declares 3 blocks, each relating to a particular file. Let's dive into those 3 sections of a template file now.

## cndi-config

### Summary

The `cndi-config` block is used to provide a templating interface around the `cndi-config.jsonc` file that acts as the core of each CNDI project. The shape of the object is as follows:

```jsonc
{
  "cndi-config": {
    "prompts": [/*...*/],
    "template": {
      "infrastructure": {/*...*/},
      "cluster_manifests": {/*...*/},
      "applications": {/*...*/}
    }
  }
}
```

The `prompts` array is a list of questions that CNDI will ask the user when they run `cndi init --interactive`, if the user is not in interactive mode the value will fallback to the default provided.

The `template` object is the standard `cndi-config.jsonc` object with one exception: anywhere within the object a user can choose to inject a value retrieved from the prompts.

### Example

Input:

```jsonc
{
    "cndi-config": {
        "prompts": [{
            // each prompts object is created leveraging the `prompts` module from 
            // https://cliffy.io/docs@v0.25.7/prompt#prompt-list
            // all prompt types are supported
            "type": "Input",
            "name": "argocdDomainName",
            "message": "What domain name should argocd be deployed on?",
            "default": "argocd.example.com"
        }],
        "template": {
            "applications": {/*...*/},
            "infrastructure": {/*...*/},
            "cluster_manifests": {
                "argo-ingress": {
                    "apiVersion": "networking.k8s.io/v1",
                    "kind": "Ingress",
                    "metadata": {/*...*/},
                    "spec": {
                        "tls": [{
                            "hosts": ["$.cndi.prompts.argocdDomainName"],
                            "secretName": "lets-encrypt-private-key"
                        }],
                        "rules": [{
                            "host": "$.cndi.prompts.argocdDomainName",
                            "http": {
                                "paths": [/*...*/]
                            }
                        }]
                    }
                }
            }
        }
    }
}
```

Output:

```jsonc
{
    "applications": {/*...*/},
    "infrastructure": {/*...*/},
    "cluster_manifests": {
        "argo-ingress": {
            "apiVersion": "networking.k8s.io/v1",
            "kind": "Ingress",
            "metadata": {/*...*/},
            "spec": {
                "tls": [
                    {
                        "hosts": ["myargocd.creedthoughts.comgov.uks"],
                        "secretName": "lets-encrypt-private-key"
                    }
                ],
                "rules": [
                    {
                        "host": "myargocd.creedthoughts.comgov.uks",
                        "http": {
                            "paths": [/*...*/]
                        }
                    }
                ]
            }
        }
    }
}
```

## env

### Summary

The `env` block is used to provide a templating interface around the `.env` file which is used for values which should not be written directly to version control. The shape of the object is as follows:

```jsonc
{
  "env": {
    "prompts": [/*...*/],
    "extend_basic_env": "aws"
  }
}
```

By specifying a list of prompts, we can again expose a list of questions to the user when they run `cndi init --interactive` or fallback to specified defaults otherwise.

The `extend_basic_env` key is used to extend the `.env` file for a given deployment target. CNDI provides the necessary prompts for each deployment target so that the `.env` file can be extended after those basic values are collected. The value of `extend_basic_env` should be the name of the deployment target you wish to extend, for example `gcp`, `azure` or `aws` as shown above.

### Example

Input:

```jsonc
{
  "extend_basic_env": "aws",
  "prompts": [
    {
      "type": "Comment",
      "comment": "airflow-git-credentials secret values for DAG Storage"
    },
    {
      "name": "GIT_SYNC_USERNAME",
      "message": "Please enter your git username for Airflow DAG Storage:",
      "type": "Secret"
    },
    {
      "name": "GIT_SYNC_PASSWORD",
      "message": "Please enter your git password for Airflow DAG Storage:",
      "type": "Secret"
    }
  ]
}
```

Output:

```bash
# airflow-git-credentials secret values for DAG Storage

GIT_SYNC_USERNAME='johnstonmatt'

GIT_SYNC_PASSWORD='hunter2'

```

## readme

### Summary

The final block of a [cndi](https://github.com/polyseam/cndi) template is the readme block, which behave similarly to the other two sections. We specify a readme block in order to provide a templating interface around the `README.md` file that is generated when a user runs `cndi init`. The shape of the object is as follows:

```jsonc
{
  "readme": {
    "extends_basic_readme": "aws",
    "template": "## $$.cndi.prompts.projectName\n\nThis is a sample readme"
  }
}
```

The `extends_basic_readme` key is used to extend the `README.md` file for a given deployment target. CNDI provides the necessary readme section for each deployment target so that the `README.md` file can be extended with more content after those basic readme sections have been written to the file.

The value of `extends_basic_readme` should be the name of the deployment target you wish to extend, for example `gcp`, `azure` or `aws` as shown above.

### Example

Input:

```jsonc
{
  "extends_basic_readme": "aws",
  "template": "## This is a simple readme section that is template specific\n\nneato!"
}
```

Output:

```markdown
<!-- Lots more content above -->

## aws

This cluster will be deployed on [Amazon Web Services](https://aws.com).
When your cluster is initialized the next step is to go to your domain registrar and create an CNAME record for your Airflow instance, and another for ArgoCD.
Both entries should point to the single load balancer that was created for your cluster.

## This is a simple readme section that is template specific

neato!
```

## all together now!

```jsonc
{
    "cndi-config": {
        "template": {/*...*/},
        "prompts": [/*...*/]
    },
    "env": {
        "prompts": [/*...*/],
        "extend_basic_env": "aws"
    },
    "readme": {
        "extends_basic_readme": "aws",
        "template": "## $$.cndi.prompts.projectName\n\nThis is a sample readme"
    }
}
```

## Included Examples

This repository contains example templates for use within [cndi](https://github.com/polyseam/cndi). There is at least one unit test within the CNDI project that verifies remote templates are working correctly by installing a template from this repository.
