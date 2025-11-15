# 📘 Lab 8 — Advanced Topics & Recap (Drift Detection, Cost Estimation, Notifications, RBAC, State Management)

## 🎯 Objectifs

Dans ce dernier lab, vous apprendrez :

- Drift Detection dans Terraform Cloud  
- Cost Estimation  
- Notifications (Slack, email, webhooks)  
- Run Queue & Logs avancés  
- RBAC et Team Management avancé  
- State versions, download & rollback  
- Verrouillage manuel du state  
- Run Tasks (compliance, Sécurité, OPA, etc.)  
- Notions avancées : Terraform Agents, intégrations pro  
- Récap complet des meilleures pratiques TFC  

---

# 🧩 Étape 0 — Mise en place

Créer le dossier :

```bash
mkdir -p labs/lab8-advanced
cd labs/lab8-advanced
```

Workspace cible dans TFC :

```
lz-advanced
```

Créer `main.tf` :

```hcl
terraform {
  cloud {
    organization = "TON_ORG"
    workspaces {
      name = "lz-advanced"
    }
  }
}

provider "aws" {
  region = "eu-west-1"
}

resource "random_id" "suffix" {
  byte_length = 4
}

resource "aws_s3_bucket" "advanced" {
  bucket = "lz-advanced-${random_id.suffix.hex}"
}
```

Commit + push.

---

# 🧩 Étape 1 — Drift Detection (dérive)

Dans AWS Console :

1. Modifier le bucket (activer versioning, changer tags)  
2. Retourner dans TFC → Workspace `lz-advanced`  
3. Click → **Start a new Plan**  

Résultat attendu :

- TFC détecte la dérive  
- Le Plan affiche :  
  ```
  Drifted resources detected
  ```

---

# 🧩 Étape 2 — Cost Estimation

Dans TFC :

1. Workspace → **Settings → General**  
2. Activer **Cost Estimation**  

Relancer un run :

```bash
terraform apply -auto-approve
```

Résultat :

- Nouvelle section dans le Plan :  
  **Estimated Monthly Cost**

---

# 🧩 Étape 3 — Notifications (Slack, Email, Webhooks)

### Slack
1. Workspace → **Notifications**  
2. Ajouter → Type : *Slack*  
3. Coller le webhook  
4. Événements :  
   - Run Completed  
   - Run Errored  
   - Drift Detected  

### Email
1. Workspace → Notifications  
2. Add Email  
3. Ajouter votre email utilisateur

### Webhooks
Pour Jenkins, SIEM, ServiceNow, GitOps…

---

# 🧩 Étape 4 — RBAC & Team Management avancé

Créer des équipes :

- **platform-admins**  
- **app-dev**  
- **auditors**

Assigner roles par workspace :

| Workspace   | platform-admins | app-dev         | auditors     |
|-------------|------------------|------------------|--------------|
| lz-dev      | Admin           | Write + Apply    | Read-only    |
| lz-stage    | Admin           | Plan             | Read-only    |
| lz-prod     | Admin + Approve | No Access        | Read-only    |

---

# 🧩 Étape 5 — State Management (Versions, Download, Rollback)

Dans workspace → **States**

Exercice :

- Comparer deux versions  
- Télécharger une version  
- Tester **Make Current Version** (rollback)

⚠️ Important en cas de corruption ou mauvais apply.

---

# 🧩 Étape 6 — State Locking (manual lock)

Dans TFC :

1. Workspace → **States**  
2. **Lock state**

Tester Apply (doit échouer) :

```
State is locked
```

Déverrouiller ensuite.

---

# 🧩 Étape 7 — Run Queue & Run Details avancés

Dans un run :

- Explorer **Graph**  
- Voir les logs Terraform détaillés  
- Ouvrir l’onglet **Changes**  
- Voir les métriques d’exécution  
- Comprendre les métadonnées du run (JSON complet disponible)

---

# 🧩 Étape 8 — Run Tasks (sécurité & conformité)

Exemples populaires :

- Checkov  
- OPA Policies  
- Snyk IaC scans  
- Bridgecrew  
- Custom enterprise API

Exercice :

- Ajouter un Run Task mock  
- Observer que TFC attend sa réponse avant l’apply

---

# 🧩 Étape 9 — Terraform Agents (optionnel)

Agents = exécuter Terraform depuis votre propre réseau (privé / sécurisé).

Concepts :

- Register Agent Token  
- Installer sur VM privée  
- Assigner workspace à un **Agent Pool**

Pas d’installation réelle dans ce lab, mais compréhension.

---

# 🧩 Étape 10 — Recap & Best Practices Terraform Cloud

✔ Toujours utiliser Variable Sets pour credentials  
✔ Séparer dev/stage/prod avec des workspaces différents  
✔ Activer speculative plans sur PR  
✔ Versionner tous les modules Producer (tags)  
✔ Garder les workspaces Prod en manual apply  
✔ Toujours activer :  
  - Cost Estimation  
  - Drift Detection  
  - Notifications  
✔ Configurer RBAC strict  
✔ Utiliser Terraform Cloud comme source unique de vérité du state

---

# 🎉 Fin du Lab 8 — Completion de la formation Terraform Cloud / Enterprise

Vous maîtrisez :

- TFC/TFE  
- Workspaces, modules, landing zones  
- OIDC, variables, policies, CD  
- Enterprise workflows & best practices  

🎓 Bravo pour avoir complété toute la série de Labs !
