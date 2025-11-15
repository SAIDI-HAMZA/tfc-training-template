# 📘 Lab 5 — Continuous Validation & Policy as Code (Sentinel / OPA)

## 🎯 Objectifs

À la fin de ce lab, vous saurez :

- Activer et utiliser **Continuous Validation** dans Terraform Cloud  
- Écrire des **policies Sentinel ou OPA/Rego**  
- Tester les policies localement  
- Déclencher des **Policy Checks** automatiques dans TFC  
- Bloquer les déploiements non conformes  
- Intégrer les policies dans un workflow Git (PR checks)

---

# 🧩 Étape 0 — Préparation

Dans votre Codespace :

```bash
mkdir -p labs/lab5-continuous-validation
cd labs/lab5-continuous-validation
```

Créer un fichier `main.tf` :

```hcl
terraform {
  cloud {
    organization = "TON_ORG"
    workspaces {
      name = "lab5-continuous-validation"
    }
  }
}

provider "aws" {
  region = "eu-west-1"
}

resource "random_id" "suffix" {
  byte_length = 4
}

resource "aws_s3_bucket" "bad_bucket" {
  bucket = "lab5-policy-non-compliant-${random_id.suffix.hex}"
}
```

Créer dans Terraform Cloud un workspace :

```
lab5-continuous-validation
```

---

# 🧩 OPTION A — Sentinel (HashiCorp)

## Étape 1A – Créer une policy Sentinel

Créer :

```
labs/lab5-continuous-validation/sentinel/restrict_s3_public_access.sentinel
```

Contenu :

```hcl
import "tfplan"

deny_public_buckets = rule {
    some resource in tfplan.resources.aws_s3_bucket as bucket {
        not bucket.applied.block_public_acls and
        not bucket.applied.block_public_policy
    }
}

main = rule {
    deny_public_buckets is false
}
```

---

## Étape 2A — Publier la policy dans Terraform Cloud

Dans TFC :

1. Organization Settings  
2. Policy Sets  
3. **Create Policy Set**  
4. Source : GitHub repo  
5. Associer le workspace :  
   ```
   lab5-continuous-validation
   ```

Test :

```bash
terraform plan
terraform apply
```

📌 Le run échoue → policy appliquée.

---

# 🧩 OPTION B (recommandée) — OPA / Rego + Run Tasks

## Étape 1B — Installer OPA (local test)

```bash
curl -L -o opa https://openpolicyagent.org/downloads/latest/opa_linux_amd64
chmod +x opa
sudo mv opa /usr/local/bin
```

---

## Étape 2B — Créer une policy OPA

Créer :

```
labs/lab5-continuous-validation/policies/deny-unencrypted-s3.rego
```

Contenu :

```rego
package tfplan

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_s3_bucket"

  not resource.change.after.server_side_encryption_configuration

  msg := sprintf("S3 bucket '%s' must enable encryption", [resource.address])
}
```

---

## Étape 3B — Tester la policy localement

```bash
terraform plan -out tfplan.out
terraform show -json tfplan.out > plan.json
opa eval --input plan.json --data policies 'data.tfplan.deny'
```

🎉 Une violation doit apparaître.

---

# 🧩 Étape 4 — Continuous Validation dans Terraform Cloud

Dans TFC :

1. Organization → Run Tasks  
2. **Create Run Task**  
3. Nom :  
   ```
   opa-validation
   ```  
4. Attacher le workspace :  
   ```
   lab5-continuous-validation
   ```

---

# 🧩 Étape 5 — Tester dans TFC

```bash
terraform apply
```

Dans TFC :

- Le run affiche un **Policy Check**
- Le run échoue si la policy est violée

🎉 Continuous Validation fonctionnelle.

---

# 🧩 Étape 6 — Intégration GitHub (PR checks)

Dans GitHub :

1. Repo → Settings  
2. Branch protection rules  
3. Activer :  
   - **Require status checks to pass**
   - Bloquer les merges si policy échoue

🎉 Les PR non conformes sont automatiquement bloquées.

---

# 🧩 Validation du Lab 5

✔ Policies Sentinel ou OPA écrites  
✔ Tests locaux  
✔ Continuous Validation active dans TFC  
✔ Run Tasks fonctionnels  
✔ Blocage automatiques des mauvais plans  
✔ Protection GitHub PR activée  

🎉 **Fin du Lab 5 — Excellent travail !**
