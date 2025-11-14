# 📘 Lab 1 — Création du Compte Terraform Cloud, Organisation & Authentification CLI

## 🎯 Objectifs du Lab

À la fin de ce lab, chaque participant aura :

- Un **compte Terraform Cloud (TFC)** personnel  
- Une **organization** dédiée dans son espace TFC  
- Son **User API Token** généré  
- Le **Terraform CLI** connecté à TFC depuis son **Codespace GitHub**  
- Une vérification de connexion via une commande simple  

Ce Lab pose les bases nécessaires pour tous les Labs suivants.

---

# 🧩 Étape 0 – Prérequis

Avant de commencer, assurez-vous :

- D’avoir un **compte GitHub**  
- D’avoir cloné le repo de formation via *Use this template*  
- D’avoir ouvert un **Codespace** sur ce repo  
- D’avoir un navigateur moderne

---

# 🧩 Étape 1 — Créer un compte Terraform Cloud

1. Aller sur :  
   👉 https://app.terraform.io/signup

2. Choisir :  
   **Create an account → Free plan**

3. Compléter les informations

4. Confirmer l’email.

---

# 🧩 Étape 2 — Créer une Organization

1. Cliquez sur votre avatar en haut à gauche  
2. **Create new organization**  
3. Nom unique (ex: `tfc-lab-john`)  
4. Mode : **Start from scratch**

---

# 🧩 Étape 3 — Générer un User API Token

1. Menu → **User Settings**  
2. **Tokens**  
3. **Create an API token**  
4. Nommer : `codespace-token`  
5. Copier le token

---

# 🧩 Étape 4 — Authentifier Terraform CLI depuis le Codespace

```bash
terraform login
```

→ Coller le token dans le navigateur.

Tester :

```bash
terraform -version
```

---

# 🧩 Étape 5 — Vérifier la connexion TFC via API

```bash
curl   --header "Authorization: Bearer $(jq -r '."app.terraform.io".token' ~/.terraform.d/credentials.tfrc.json')"   https://app.terraform.io/api/v2/user
```

Vous devez obtenir un JSON utilisateur.

---

# 🧩 Étape 6 — Validation

✔ Compte TFC  
✔ Organization TFC  
✔ Token généré  
✔ CLI connecté  
✔ API TFC répond

🎉 Fin du Lab 1


