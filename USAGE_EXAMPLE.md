# Using key_material_base64 with Jenkins

## How to Encode Your Key

First, encode your PEM private key to base64:

```bash
# On your local machine
base64 -w 0 ~/.chef/your-key.pem
```

Or in one line:
```bash
cat ~/.chef/your-key.pem | base64 -w 0
```

This will output a single line of base64-encoded text without newlines.

## Store in Jenkins

1. Go to Jenkins → Credentials → Add Credentials
2. Select **Kind: Secret text**
3. Paste the base64-encoded string (the output from above command)
4. Set ID to something like `chef-api-pem-base64`

## Use in Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {
        CHEF_SERVER_URL = "https://api.chef.io/organizations/yourorg/"
        CHEF_CLIENT_NAME = "your-client-name"
        CHEF_KEY_MATERIAL_BASE64 = credentials('chef-api-pem-base64')
    }

    stages {
        stage('Terraform Apply') {
            steps {
                sh 'terraform apply -auto-approve'
            }
        }
    }
}
```

## Or in Terraform Configuration

```hcl
provider "chef" {
  server_url           = "https://api.chef.io/organizations/yourorg/"
  client_name          = "your-client-name"
  key_material_base64  = var.chef_key_base64
}
```

No more dealing with newlines! The provider will automatically decode the base64 string and handle it properly.
