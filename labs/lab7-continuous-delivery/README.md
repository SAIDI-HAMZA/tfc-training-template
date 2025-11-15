# 📘 Lab 7 — Continuous Delivery avec Terraform Cloud

## 🎯 Objectifs

À la fin de ce lab, vous saurez :

- Utiliser **Terraform Cloud** comme moteur principal de Continuous Delivery  
- Configurer un **workspace VCS-driven** connecté à votre repo GitHub  
- Déclencher automatiquement des **speculative plans** sur les Pull Requests  
- Activer l’**auto-apply** sur les commits de certaines branches (par ex. `develop`)  
- Protéger la branche `main` avec des **applies manuels** (approvals)  
- Utiliser les **Run Triggers** pour chaîner plusieurs workspaces (dev → stage → prod)

Aucun GitHub Actions n’est utilisé dans ce lab : tout passe par **TFC + Git**.

---

# 🧩 Étape 0 — Pré-requis

- Vous avez déjà un repo GitHub de formation basé sur le template  
- Vous avez déjà des workspaces Terraform Cloud (par ex. `lz-dev`, `lz-stage`, `lz-prod` du Lab 6)  
- Vous connaissez la différence **CLI-driven** vs **VCS-driven** (vue dans les labs précédents)

---

# 🧩 Étape 1 — Configurer la connexion VCS dans Terraform Cloud

Dans Terraform Cloud :

1. Allez dans **Organization Settings**  
2. Menu **VCS Providers**  
3. Cliquez sur **Add VCS Provider**  
4. Choisissez **GitHub** (ou GitHub App)  
5. Suivez l’assistant pour autoriser TFC à accéder à votre organisation/repo GitHub

🎯 Objectif : permettre à TFC d’écouter les commits et PR de votre repo.

---

# 🧩 Étape 2 — Créer un Workspace VCS-driven pour l’environnement Dev

Nous allons créer un workspace `lz-dev-cd` qui :

- est directement connecté à votre repo GitHub  
- écoute une branche donnée (par exemple `develop`)  
- applique automatiquement les changements

Dans Terraform Cloud :

1. **Workspaces → New Workspace**  
2. Choisissez **Version control workflow**  
3. Sélectionnez votre repo GitHub de formation  
4. Donnez le nom du workspace :

```
lz-dev-cd
```

5. Branche par défaut :  
   ```
   develop
   ```
6. Type d’exécution : **Remote**  
7. Dans les settings du workspace, activez :

- **Automatic speculative plans** (par défaut)
- **Auto apply** pour ce workspace

📌 **Résultat :**

- Chaque commit sur `develop` déclenche automatiquement un **Plan** puis un **Apply** dans `lz-dev-cd`.

---

# 🧩 Étape 3 — Préparer la config Terraform pour ce workspace

Dans votre repo, créez un dossier :

```
env/dev/
```

Dans `env/dev/main.tf`, vous pouvez réutiliser l’infra du Lab 6 :

```hcl
terraform {
  cloud {
    organization = "TON_ORG"

    workspaces {
      name = "lz-dev-cd"
    }
  }

  required_version = ">= 1.5.0"
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

Commitez et poussez :

```bash
git add env/dev/main.tf
git commit -m "Add dev environment (lz-dev-cd)"
git push origin develop
```

Allez dans Terraform Cloud :

- Workspace `lz-dev-cd` → un run **Plan + Apply** doit être en cours ou terminé ✅

---

# 🧩 Étape 4 — Activer les speculative plans sur Pull Requests

Les speculative plans permettent d’avoir un **plan de changement** pour chaque PR, sans appliquer.

Dans TFC, sur le workspace `lz-dev-cd` :

- Assurez-vous que l’option **Automatic speculative plans** est activée.

Test :

1. Créez une nouvelle branche :

```bash
git checkout -b feature/change-tags
```

2. Modifiez par exemple les tags ou le nom du module dans `env/dev/main.tf`.  
3. Commit + push :

```bash
git commit -am "Change tags for dev env"
git push origin feature/change-tags
```

4. Ouvrez une **Pull Request** de `feature/change-tags` vers `develop`.

Résultat :

- TFC détecte la PR  
- TFC exécute un **Speculative Plan**  
- Dans TFC, vous voyez le run marqué comme **Speculative** (non appliqué)  

🎉 Vous avez maintenant des plans auto générés pour chaque PR, sans GitHub Actions.

---

# 🧩 Étape 5 — Continuous Delivery sur Dev (auto-apply)

Lorsque la PR est **mergée** dans `develop` :

- Un nouveau commit apparaît sur `develop`  
- TFC déclenche automatiquement un nouveau **Plan + Apply** sur `lz-dev-cd` grâce à l’option auto-apply

💡 C’est le **Continuous Delivery** pour l’environnement Dev.

---

# 🧩 Étape 6 — Créer un Workspace VCS-driven pour Prod (apply manuel)

Répétez la création d’un workspace, mais cette fois pour prod :

1. Workspaces → New  
2. Type : **Version control workflow**  
3. Repo GitHub : le même  
4. Workspace name :

```
lz-prod-cd
```

5. Branche surveillée :  
   ```
   main
   ```

6. Désactivez **Auto apply** (important)  
7. Gardez les speculative plans activés.

Dans votre repo, créez :

```
env/prod/main.tf
```

Contenu similaire au dev, en changeant l’environnement :

```hcl
terraform {
  cloud {
    organization = "TON_ORG"

    workspaces {
      name = "lz-prod-cd"
    }
  }

  required_version = ">= 1.5.0"
}

provider "aws" {
  region = "eu-west-1"
}

module "lz_s3" {
  source  = "app.terraform.io/TON_ORG/s3_lz/aws"
  version = "1.0.0"

  name        = "landing-zone-bucket"
  environment = "prod"
}
```

---

# 🧩 Étape 7 — Promotion Dev → Prod via Git + TFC

Flux recommandé :

1. Développer et tester les changements sur `develop` (workspace `lz-dev-cd`)  
2. Une fois validés, merger `develop` → `main` :

```bash
git checkout main
git merge develop
git push origin main
```

3. TFC détecte le nouveau commit sur `main` → déclenche un **Plan** sur `lz-prod-cd`  
4. Un opérateur va dans TFC, ouvre le run du workspace `lz-prod-cd`, vérifie le Plan, puis clique sur :

   👉 **Confirm & Apply**

🎉 Vous avez une **chaîne complète** :

- Dev : auto-apply  
- Prod : plan automatique + apply manuel

---

# 🧩 Étape 8 — Run Triggers (chaînage Dev → Stage → Prod)

Les **Run Triggers** permettent de dire :

> “Quand ce workspace a fini un apply, déclenche un run sur ce/ces autres workspaces.”

Dans Terraform Cloud :

1. Allez dans le workspace `lz-dev-cd`  
2. Onglet **Run Triggers**  
3. Cliquez **Add run trigger**  
4. Sélectionnez le workspace `lz-stage` (ou `lz-stage-cd` si existant)  

Résultat :

- Quand Dev applique, Stage est automatiquement planifié/apply (selon config).

Vous pouvez ainsi chaîner :

`lz-dev-cd` → `lz-stage` → `lz-prod-cd`

---

# 🧩 Validation du Lab 7

✔ Workspace VCS-driven Dev (`develop` + auto-apply)  
✔ Workspace VCS-driven Prod (`main` + apply manuel)  
✔ Speculative plans sur PR  
✔ Continuous Delivery Dev via commits sur `develop`  
✔ Promotion Dev → Prod via merge Git + TFC  
✔ Run Triggers pour chaîner les environments  

🎉 **Fin du Lab 7 — Continuous Delivery full Terraform Cloud !**
