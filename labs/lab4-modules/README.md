# 📘 Lab 4 — Modules Terraform : création, versionning & tests (tflint / validate / structure pro)

## 🎯 Objectifs

À la fin du lab, vous saurez :

- Créer un **module Terraform professionnel**
- Respecter une **structure standardisée**
- Publier un module dans le **Terraform Cloud Private Registry**
- Gérer les **releases & versions** (tags Git)
- Tester le module (terraform validate, tflint)
- Consommer le module depuis un consumer
- Mettre à jour le module et publier une nouvelle version

---

# 🧩 Étape 0 — Préparation

Dans votre Codespace :

```bash
mkdir -p labs/lab4-modules
cd labs/lab4-modules
mkdir modules consumer
```

---

# 🧩 Étape 1 — Créer un module Terraform professionnel

Créer :

```
labs/lab4-modules/modules/s3_bucket/
```

Structure :

```
s3_bucket/
 ├── main.tf
 ├── variables.tf
 ├── outputs.tf
 ├── versions.tf
 ├── README.md
```

---

## 📄 versions.tf

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

---

## 📄 variables.tf

```hcl
variable "bucket_prefix" {
  type        = string
  description = "Préfixe du nom du bucket"
}

variable "tags" {
  type        = map(string)
  description = "Tags appliqués au bucket"
  default     = {}
}
```

---

## 📄 main.tf

```hcl
resource "random_id" "suffix" {
  byte_length = 4
}

resource "aws_s3_bucket" "this" {
  bucket = "${var.bucket_prefix}-${random_id.suffix.hex}"

  tags = var.tags
}
```

---

## 📄 outputs.tf

```hcl
output "bucket_name" {
  value = aws_s3_bucket.this.bucket
}
```

---

## 📄 README.md

```md
# Module S3 Bucket

Ce module crée :
- un bucket S3 avec suffix random
- des tags personnalisables

## Inputs
- bucket_prefix (string, required)
- tags (map(string), optional)

## Outputs
- bucket_name
```

---

# 🧩 Étape 2 — Tester le module localement

```bash
cd labs/lab4-modules/modules/s3_bucket
terraform init
terraform validate
```

---

# 🧩 Étape 3 — Installer & exécuter tflint

Dans le Codespace :

```bash
curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash
tflint
```

Corriger les warnings si nécessaire.

---

# 🧩 Étape 4 — Publier le module dans le Terraform Cloud Private Registry

⚠️ Le module doit respecter cette structure dans GitHub :

```
modules/<votre_github_username>/s3-bucket/aws
```

Exemple :

```
modules/hamza/s3-bucket/aws
```

Ensuite dans Terraform Cloud :

1. Private Registry  
2. **Publish Module**  
3. Sélectionner votre repo GitHub  
4. TFC détecte automatiquement le module

---

# 🧩 Étape 5 — Versionner le module

Dans votre repo GitHub :

```bash
git add .
git commit -m "Add s3 bucket module"
git tag v0.1.0
git push origin main --tags
```

TFC importe automatiquement la version.

---

# 🧩 Étape 6 — Consumer du module

Dans :

```
labs/lab4-modules/consumer/main.tf
```

Créer le fichier :

```hcl
terraform {
  cloud {
    organization = "TON_ORG"
    workspaces {
      name = "lab4-modules"
    }
  }
}

provider "aws" {
  region = "eu-west-1"
}

module "my_bucket" {
  source  = "app.terraform.io/TON_ORG/s3-bucket/aws"
  version = "0.1.0"

  bucket_prefix = "lab4-module"

  tags = {
    Environment = "lab4"
    Owner       = "student"
  }
}

output "bucket_name" {
  value = module.my_bucket.bucket_name
}
```

Créer dans TFC un workspace :

```
lab4-modules
```

---

# 🧩 Étape 7 — Tester

Dans le consumer :

```bash
terraform init
terraform apply -auto-approve
```

✔ Le module fonctionne  
✔ Le run est exécuté dans TFC  
✔ Le bucket est créé

---

# 🧩 Étape 8 — Ajouter une nouvelle version du module

Dans le module modifier :

### variables.tf
```hcl
variable "enable_versioning" {
  type    = bool
  default = false
}
```

### main.tf

```hcl
resource "aws_s3_bucket_versioning" "versioning" {
  bucket = aws_s3_bucket.this.id

  versioning_configuration {
    status = var.enable_versioning ? "Enabled" : "Suspended"
  }
}
```

Commit + tag :

```bash
git commit -am "Add versioning parameter"
git tag v0.2.0
git push origin main --tags
```

---

# 🧩 Étape 9 — Mise à jour du consumer

```hcl
version = "0.2.0"
enable_versioning = true
```

Puis :

```bash
terraform apply -auto-approve
```

✔ La nouvelle version s’applique  
✔ Le module met à jour le bucket  
✔ Le consumer consomme bien la version 0.2.0  

---

# 🧩 Validation du Lab

✔ Module structure pro  
✔ Tests validate & tflint  
✔ Publication Private Registry  
✔ Versionning Git  
✔ Consumer opérationnel  
✔ Mise à jour consumer → nouvelle version du module  

🎉 **Fin du Lab 4 — Excellent travail !**
