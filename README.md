# example-cndi-templates

## Wbat is a CNDI Template?

A [cndi](https://cndi.run/gh?utm_content=gh_cndi_template_readme&utm_campaign=cndi_template_readme_v1&utm_source=github.com/polyseam/cndi-template-basic&utm_medium=repo&utm_id=1007) Template is a JSON file that CNDI can parse in order to provide a simplified deployment experience for some cloud-native application. The template declares 3 `"output"` blocks, each relating to a particular file, and one `"prompts"` array which allows each output to be customized by end users. Let's start by looking at the `"prompts"` array.

## prompts

The `prompts` array is a list of questions that CNDI will ask the user when they run `cndi init --interactive`, if the user is not in interactive mode the value will fallback to the default provided.

```jsonc
{
  "prompts": [
    {
      // each prompts object is created leveraging the `prompts` module from
      // https://cliffy.io/docs/prompt#prompt-list
      // the following input types are all supported:
      // "Input" | "Secret" | "Confirm" | "Toggle" | "Select" | "List" | "Checkbox" | "Number"
      "type": "Input",
      "name": "argocdDomainName",
      "message": "What domain name should argocd be deployed on?",
      "default": "argocd.example.com"
    },
    {
      "type": "Secret",
      "name": "myAirflowPassword",
      "message": "Please enter a default password for Airflow:",
      "default": "admin"
    }
  ]
}
```

When using the `--interactive` flag in CNDI each of these prompts will provide an interactive prompt to populate the value. 

If the user does not provide a value _or_ if the user is not using interactive mode, the default will be used. 

These values are then accessible inside of a template `"output"` using the following syntax: `{{ $.cndi.prompts.responses.<prompt_name> }}`. 

For example, if we wanted to use the value of `argocdDomainName` in a template we would use the following syntax: `{{ $.cndi.prompts.responses.argocdDomainName }}`.

## outputs

## cndi-config

### Summary

The `outputs["cndi-config"]` block is used to provide a templating interface around the `cndi-config.jsonc` file that acts as the core of each CNDI project. The shape of the object is as follows:

```jsonc
{
  "outputs": {
    "cndi-config": {
      "infrastructure": {
        /*...*/
      },
      "cluster_manifests": {
        /*...*/
      },
      "applications": {
        /*...*/
      }
    }
  }
}
```

Let's look at an example of leveraging a `"prompt"` entry to populate a value into the `cndi-config.jsonc` file.

### Example

Input:

```jsonc
{
  "cndi-config": {
    "template": {
      "applications": {
        /*...*/
      },
      "infrastructure": {
        /*...*/
      },
      "cluster_manifests": {
        "argo-ingress": {
          "apiVersion": "networking.k8s.io/v1",
          "kind": "Ingress",
          "metadata": {
            /*...*/
          },
          "spec": {
            "tls": [
              {
                "hosts": ["{{ $.cndi.prompts.responses.argocdDomainName }}"],
                "secretName": "lets-encrypt-private-key"
              }
            ],
            "rules": [
              {
                "host": "{{ $.cndi.prompts.responses.argocdDomainName }}",
                "http": {
                  "paths": [
                    /*...*/
                  ]
                }
              }
            ]
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
  "applications": {
    /*...*/
  },
  "infrastructure": {
    /*...*/
  },
  "cluster_manifests": {
    "argo-ingress": {
      "apiVersion": "networking.k8s.io/v1",
      "kind": "Ingress",
      "metadata": {
        /*...*/
      },
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
              "paths": [
                /*...*/
              ]
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
  "outputs":{
    {...},
    "env": {
      "extend_basic_env": "aws",
      "entries": [
        {
          "name": "POSTGRES_PORT",
          "value": "5432"
        }
      ]
    }
  }
}
```


The `extend_basic_env` key is used to extend the built-in `.env` file cndi provides for a given deployment target. CNDI provides the necessary prompts for each deployment target so that the `.env` file can be extended after those basic values are collected. The value of `extend_basic_env` should be the name of the deployment target you wish to extend, for example `gcp`, `azure` or `aws` as shown above.

It's also possible to leverage `"prompt"` values here in `outputs["env"]`, let's look at an example:

### Example

Input:

```jsonc
{
  "prompts": [
    {
      "name": "secretAppPassword",
      "type": "Input",
      "message": "What password should be used for SecretApp?"
    },
  ],
  "outputs": {
    "env": {
      "extend_basic_env": "aws",
      "entries": [
        {
          "name": "SECRETAPP_PASSWORD",
          "value": "{{ $.cndi.prompts.responses.secretAppPassword }}"
        }
      ]
    }
  }
}
```

```bash

```

Output:

```bash

SECRETAPP_PASSWORD='myairflowpassword'

```

## readme

### Summary

The final block of a [cndi](https://github.com/polyseam/cndi) template is the `readme` block, which behave similarly to the other two sections. We specify a readme block in order to provide a templating interface around the `README.md` file that is generated when a user runs `cndi init`. The shape of the object is as follows:

```jsonc
{
  "readme": {
    "extends_basic_readme": "aws",
    "template": "## This is a sample readme section"
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
    "template": {
      /*...*/
    },
    "prompts": [
      /*...*/
    ]
  },
  "env": {
    "prompts": [
      /*...*/
    ],
    "extend_basic_env": "aws"
  },
  "readme": {
    "extends_basic_readme": "aws",
    "template": "##This is a sample readme\n\nneato!"
  }
}
```

## Included Examples

This repository contains example templates for use within [cndi](https://github.com/polyseam/cndi). There is at least one unit test within the CNDI project that verifies remote templates are working correctly by installing a template from this repository.
