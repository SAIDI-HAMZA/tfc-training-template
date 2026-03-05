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


## Configurer les identifiants AWS

Avant d’exécuter Terraform, configurez vos identifiants AWS avec la commande suivante :

```bash
aws configure
```

Lorsque les informations sont demandées, **remplacez les valeurs par les identifiants AWS qui vous ont été fournis** :

```
AWS Access Key ID [None]: <remplacer par la clé fournie>
AWS Secret Access Key [None]: <remplacer par la clé fournie>
Default region name [None]: eu-west-3
Default output format [None]: json
```

Vérifiez ensuite que la configuration fonctionne :

```bash
aws sts get-caller-identity
```

Si la commande retourne les informations du compte AWS, la configuration est correcte.

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
    Name = "tfc-lab2_add_your_name"
  }
}

resource "random_id" "suffix" {
  byte_length = 4
}
```

---
⚠️ Remplacez le tag de bucket "tfc-lab2_add_your_name" par votre nom

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

Ajoutez le bloc de configuration suivant à la fin de votre fichier `main.tf` pour configurer l’intégration avec Terraform Cloud et ajouter le backend :

```hcl
cloud {
  organization = "TON_ORGANIZATION_TFC"

  workspaces {
    name = "lab2-workspaces"
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
## Ajouter les identifiants AWS dans Terraform Cloud

Lors du premier `terraform plan`, vous verrez une erreur indiquant que **les credentials AWS sont introuvables**.

Cette erreur montre que Terraform n’exécute plus le plan **localement**, mais dans **Terraform Cloud**.

Pour corriger cela, vous devez ajouter vos identifiants AWS **dans le workspace Terraform Cloud**.

### Étapes

1. Ouvrez votre workspace **lab2-workspaces** dans Terraform Cloud.

2. Cliquez sur **Variables**.

3. Dans la section **Environment Variables**, ajoutez les variables suivantes :

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
```

4. **Remplacez les valeurs par les identifiants AWS qui vous ont été fournis.**

5. Relancez ensuite la commande :

```bash
terraform plan
terraform apply -auto-approve
```

Le run sera alors exécuté correctement dans **Terraform Cloud**.

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
