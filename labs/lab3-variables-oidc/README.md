# 📘 Lab 3 — Variables, Variable Sets & Dynamic Credentials (AWS OIDC)

## 🎯 Objectifs

À la fin de ce lab, vous saurez :

- Créer des **variables Terraform** dans Terraform Cloud  
- Utiliser les **Variable Sets**  
- Configurer l’authentification **OIDC Terraform Cloud → AWS IAM**  
- Utiliser des **AWS dynamic credentials**  
- Éliminer les clés AWS statiques  
- Tester une exécution sécurisée dans TFC

---

# 🧩 Étape 0 — Préparation du dossier du Lab

Dans votre Codespace :

```bash
mkdir -p labs/lab3-variables-oidc
cd labs/lab3-variables-oidc
```

Créer les fichiers suivants :

---

## 📄 main.tf

```hcl
terraform {
  cloud {
    organization = "TON_ORGANIZATION_TFC"

    workspaces {
      name = "lab3-variables-oidc"
    }
  }

  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
}

resource "aws_s3_bucket" "demo" {
  bucket = "tfc-lab3-${random_id.suffix.hex}"

  tags = {
    Name = "tfc-lab3-oidc-demo"
  }
}

resource "random_id" "suffix" {
  byte_length = 4
}

variable "region" {
  type    = string
  default = "eu-west-1"
}
```

👉 Remplacez **TON_ORGANIZATION_TFC** par le nom de votre organisation TFC.

---

# 🧩 Étape 1 — Créer le Workspace Terraform Cloud

Dans Terraform Cloud :

1. Allez dans votre Organization  
2. Menu **Workspaces**  
3. **Create Workspace**  
4. Nom du workspace :

```
lab3-variables-oidc
```

5. Type : **CLI-driven workflow**

---

# 🧩 Étape 2 — Comprendre les variables dans TFC

Terraform Cloud distingue :

### 🔹 Variables Terraform
- équivalent des fichiers `.tfvars`
- peuvent être *HCL* ou *sensitive*

### 🔹 Variables d’environnement
- ex : `AWS_REGION`, `TF_LOG`, etc.

### 🔹 Variable Sets
- variables partagées entre plusieurs workspaces
- idéal pour AWS credentials, policies communes, environment flags

Nous allons utiliser un **Variable Set** pour configurer l’OIDC AWS.

---

# 🧩 Étape 3 — Configurer OIDC Terraform Cloud → AWS IAM

## 🎯 Objectif :
Terraform Cloud va assumer un rôle IAM dans AWS **sans clé statique**, via OIDC.

---

# 🔧 Étape 3.1 — Ajouter le fournisseur OIDC dans AWS IAM

1. IAM → **Identity Providers**  
2. **Add provider**  
3. Type : **OIDC**  
4. Provider URL :  
   ```
   https://app.terraform.io
   ```
5. Audience :
   ```
   aws.workload.identity
   ```

Validez.

---

# 🔧 Étape 3.2 — Créer un rôle IAM utilisé par Terraform Cloud

Dans AWS IAM :

1. IAM → Roles → **Create Role**  
2. Type : **Web Identity**  
3. Provider : celui que vous venez de créer  
4. Audience : `aws.workload.identity`  

Modifiez ensuite la **Trust Policy** :

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<AWS_ACCOUNT_ID>:oidc-provider/app.terraform.io"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "app.terraform.io:aud": "aws.workload.identity"
        },
        "StringLike": {
          "app.terraform.io:sub": "organization:TON_ORG_NAME:workspace:lab3-variables-oidc:run_phase:*"
        }
      }
    }
  ]
}
```

➡️ Remplacez :
- `<AWS_ACCOUNT_ID>`  
- `TON_ORG_NAME`

### 📌 Permissions du rôle :

Attacher une policy (pour ce lab) :

```
AmazonS3FullAccess
```

---

# 🧩 Étape 4 — Créer un Variable Set dans Terraform Cloud

Dans Terraform Cloud → Workspace lab3-variables-oidc → Variables → Environment variables, ajoutez :
```
TFC_AWS_PROVIDER_AUTH = true
TFC_AWS_RUN_ROLE_ARN = <remplacer par le Role ARN AWS fourni>
AWS_REGION = eu-west-1 (ou la région du lab)

```
---

# 🧩 Étape 5 — Tester les AWS Dynamic Credentials

Dans votre Codespace :

```bash
terraform init
terraform apply -auto-approve
```

Dans Terraform Cloud, ouvrez le Run :

➡️ **View raw logs**

Vous devez voir :

```
Refreshing AWS credentials via AssumeRoleWithWebIdentity
```

🎉 Cela prouve que l’OIDC fonctionne sans clé statique.

---

# 🧩 Étape 6 — Validation du Lab

✔ Workspace créé  
✔ Provider OIDC configuré  
✔ IAM Role avec trust policy TFC  
✔ Variable Set opérationnel  
✔ Aucun secret AWS statique  
✔ S3 bucket créé grâce à OIDC  
✔ Logs confirment l’utilisation des credentials dynamiques  

🎉 **Fin du Lab 3 — Excellent travail !**

