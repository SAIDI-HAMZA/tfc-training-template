# 📘 Lab 2 — Workspaces, Backend local → Terraform Cloud, State & Locking

## 🎯 Objectifs du Lab

À la fin de ce Lab, vous saurez :

- Créer un **workspace Terraform Cloud (TFC)**
- Déployer une ressource AWS en **local execution**
- Migrer le state vers un **workspace TFC**
- Utiliser le mode **remote execution**
- Comprendre et observer le **state locking**
- Comprendre la différence entre backend **local** et **TFC**

---

# 🧩 Étape 0 — Préparation dans votre Codespace

Dans votre Codespace :

```bash
mkdir -p labs/lab2-workspaces
cd labs/lab2-workspaces
```

Créez les fichiers Terraform suivants :

---

## 📄 `main.tf`

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

provider "aws" {
  region = "eu-west-1"
}

resource "aws_s3_bucket" "demo" {
  bucket = "tfc-lab2-${random_id.suffix.hex}"
  tags = {
    Name = "tfc-lab2"
  }
}

resource "random_id" "suffix" {
  byte_length = 4
}
```

---

# 🧩 Étape 1 — Déploiement Local (Backend local)

Initialisez Terraform :

```bash
terraform init
```

Appliquez l’infrastructure :

```bash
terraform apply -auto-approve
```

Vérifiez :

- Le bucket est créé dans AWS
- Un fichier `terraform.tfstate` existe localement

---

# 🧩 Étape 2 — Créer un Workspace Terraform Cloud

1. Connectez-vous à Terraform Cloud  
2. Allez dans votre Organization  
3. Menu **Workspaces**  
4. Cliquez **Create Workspace**  
5. Nom du workspace :

```
lab2-workspaces
```

6. Type : **CLI-driven workflow**

---

# 🧩 Étape 3 — Migrer le Backend Local → Terraform Cloud

Modifiez `main.tf` pour y ajouter le backend :

```hcl
terraform {
  cloud {
    organization = "TON_ORGANIZATION_TFC"

    workspaces {
      name = "lab2-workspaces"
    }
  }

  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

⚠️ Remplacez **TON_ORGANIZATION_TFC** par votre nom d’organization.

---

### Re-initialisez :

```bash
terraform init
```

Terraform va demander :

```
Do you want to migrate your state to Terraform Cloud?
```

Répondez :

```
yes
```

### Vérifiez :

- Le fichier local `terraform.tfstate` a disparu
- Le state apparaît dans :  
  **TFC → Workspace → lab2-workspaces → States**

🎉 Migration réussie !

---

# 🧩 Étape 4 — Tester le Remote Execution

Dans votre Codespace :

```bash
terraform plan
terraform apply -auto-approve
```

Vous verrez :

- Le run sera exécuté **dans Terraform Cloud**
- Une URL de suivi du run s'affiche dans le terminal
- Le run apparaît dans **TFC → Workspace → Runs**

---

# 🧩 Étape 5 — Tester le State Locking

Dans votre Codespace :

1️⃣ Lancez un apply :

```bash
terraform apply
```

2️⃣ Pendant que le run est **en cours**, dans un NOUVEAU terminal Codespace :

```bash
terraform apply
```

Vous devez voir :

```
State is locked by another operation
```

👏 Preuve du **state locking** de Terraform Cloud.

---

# 🧩 Étape 6 — Nettoyage

À la fin du lab :

```bash
terraform destroy -auto-approve
```

Ou via l’interface TFC → Workspace → **Destroy Workspace Resources**

---

# 🧩 Validation du Lab

✔ Déploiement local → OK  
✔ Migration du backend → OK  
✔ State TFC visible → OK  
✔ Exécution remote → OK  
✔ Locking démontré → OK  
✔ Nettoyage → OK  

🎉 **Fin du Lab 2 — Excellent travail !**
