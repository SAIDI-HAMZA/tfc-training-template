# 📘 Lab 6 — Terraform Landing Zone Workflow (Producer / Consumer, Git Branching)

## 🎯 Objectifs

À la fin de ce lab, vous saurez :

- Concevoir un workflow **Landing Zone** avec séparation Producer / Consumer  
- Structurer des **modules Producer** versionnés et publiés  
- Mettre en place une **Git branching strategy** complète  
- Utiliser des **workspaces par environnement** (dev/stage/prod)  
- Promouvoir les versions d’infra via Git + TFC  
- Comprendre le cycle complet Producer → Consumer → Promotion  

---

# 🧩 Étape 0 — Préparation du Lab

Dans votre Codespace :

```bash
mkdir -p labs/lab6-landing-zone
cd labs/lab6-landing-zone
mkdir producer consumer
```

---

# 🧩 Étape 1 — Producer (Module officiel de la Landing Zone)

Créer :

```
labs/lab6-landing-zone/producer/s3_lz/
```

Structure :

```
s3_lz/
 ├── main.tf
 ├── variables.tf
 ├── outputs.tf
 ├── versions.tf
 ├── README.md
```

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

## 📄 variables.tf
```hcl
variable "name" {
  type        = string
  description = "Nom du bucket"
}

variable "environment" {
  type        = string
  description = "Environnement (dev, stage, prod)"
}
```

## 📄 main.tf
```hcl
resource "aws_s3_bucket" "lz" {
  bucket = "${var.name}-${var.environment}"

  tags = {
    environment = var.environment
    owner       = "producer"
  }
}
```

## 📄 outputs.tf
```hcl
output "bucket_name" {
  value = aws_s3_bucket.lz.id
}
```

## 📄 README.md
Résumé du module Producer.

---

# 🧩 Étape 2 — Git Branch Strategy

Créer les branches :

```
main      → production
develop   → préproduction
feature/* → développement module
```

Dans Codespace :

```bash
git checkout -b develop
git checkout -b feature/s3-enhancement
```

Workflow complet :

```
feature/*  →  develop  →  main
```

---

# 🧩 Étape 3 — Versionner le Producer

Depuis `main` :

```bash
git add .
git commit -m "Add s3 landing zone module"
git tag v1.0.0
git push origin main --tags
```

Terraform Cloud Private Registry importe automatiquement la version.

---

# 🧩 Étape 4 — Consumer (Environnements Dev / Stage / Prod)

Créer les dossiers :

```
labs/lab6-landing-zone/consumer/env-dev/
labs/lab6-landing-zone/consumer/env-stage/
labs/lab6-landing-zone/consumer/env-prod/
```

Dans chaque `main.tf` :

```hcl
terraform {
  cloud {
    organization = "TON_ORG"
    workspaces {
      name = "lz-dev"   # ajuster selon env
    }
  }
}

provider "aws" {
  region = "eu-west-1"
}

module "lz_s3" {
  source  = "app.terraform.io/TON_ORG/s3_lz/aws"
  version = "1.0.0"

  name        = "landing-zone-bucket"
  environment = "dev"
}
```

Créer les workspaces TFC :

```
lz-dev
lz-stage
lz-prod
```

---

# 🧩 Étape 5 — Promotion Producer → Consumer

Cycle complet :

1. Développeur Producer crée une feature  
2. PR vers `develop`  
3. Tests & validation  
4. Promote vers `main`  
5. Tag de release `v1.1.0`  
6. Consumer adopte la nouvelle version :

```hcl
version = "1.1.0"
```

Puis :

```bash
terraform init
terraform apply
```

---

# 🧩 Étape 6 — Environnements TFC & Credentials

Configurer les workspaces TFC :

- `lz-dev` → environnement dev  
- `lz-stage` → préproduction  
- `lz-prod` → production sécurisée  

Activer les **Dynamic AWS Credentials (OIDC)** (voir Lab 3).

---

# 🧩 Étape 7 — Exemple de promotion complète

## Producer :

```bash
git commit -am "New feature"
git tag v1.1.0
git push origin main --tags
```

## Consumer dev :

```bash
version = "1.1.0"
```

```bash
terraform apply
```

## Consumer stage :

Même chose mais appliqué depuis env-stage.

## Consumer prod :

Promotion finale avec approvals si nécessaires.

---

# 🧩 Étape 8 — RBAC & Approvals

Dans TFC :

- Mettre l’environnement **prod** en workspace protégé  
- Ajouter des équipes :  
  - Producer Team  
  - Consumer Dev  
  - Consumer Ops  
- Activer :  
  - **Run Approvals pour prod**  
  - **Permissions limitées pour Dev**

---

# 🧩 Validation du Lab 6

✔ Module Producer propre  
✔ Branching Git (main / develop / feature)  
✔ Versionning via tags  
✔ Registry TFC  
✔ Consumer par environnement  
✔ Workflows dev → stage → prod  
✔ RBAC et approvals configurés  

🎉 **Fin du Lab 6 — Vous maîtrisez un vrai workflow Landing Zone !**
