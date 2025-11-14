# 📘 Lab 1 — Création du Repo de Formation, du Codespace, du Compte Terraform Cloud et Authentification CLI

## 🎯 Objectifs du Lab

À la fin de ce lab, chaque participant aura :

- Un **repo GitHub personnel** basé sur le template de formation  
- Un **Codespace** ouvert et fonctionnel sur ce repo  
- Un **compte Terraform Cloud (TFC)** personnel  
- Une **organization** dédiée dans son espace TFC  
- Son **User API Token** généré  
- Le **Terraform CLI** connecté à TFC depuis son Codespace  
- Une vérification de connexion via une commande simple  

Ce Lab pose les bases nécessaires pour tous les Labs suivants.

---

# 🧩 Étape 0 – Prérequis

Avant de commencer, assurez-vous :

- D’avoir un **compte GitHub** actif  
- D’avoir reçu le lien du repo **template** de formation  
- D’avoir un navigateur moderne (Chrome, Firefox, Edge)

---

# 🧩 Étape 1 — Créer votre repo GitHub à partir du template

1. Ouvrez le lien du **repo template** fourni par le formateur :  

   👉 **https://github.com/SAIDI-HAMZA/tfc-training-template.git**

2. En haut à droite de la page du repo, cliquez sur :  
   **Use this template** → **Create a new repository**

3. Choisissez :
   - **Repository name** :  
     Exemple : `tfc-training-votre-prenom`
   - **Owner** : votre compte GitHub personnel
   - **Visibility** : Private ou Public (au choix)

4. Cliquez sur :  
   **Create repository from template**

Vous avez maintenant **votre propre repo personnel** pour suivre la formation.

---

# 🧩 Étape 2 — Démarrer un GitHub Codespace sur votre repo

1. Sur la page de **votre** repo nouvellement créé, cliquez sur le bouton vert **Code**.  
2. Onglet **Codespaces**  
3. Cliquez sur **Create codespace on main**.

Une fenêtre VS Code dans le navigateur va s’ouvrir.  
Ce Codespace contient déjà :
- Terraform  
- AWS CLI  
- Extensions VSCode nécessaires  

👉 **Toutes les commandes se feront dans le Terminal intégré du Codespace.**

---

# 🧩 Étape 3 — Créer un compte Terraform Cloud

1. Aller sur :  
   👉 https://app.terraform.io/signup

2. Cliquer sur **Create an account → Free plan**  
3. Renseigner les informations  
4. Confirmer votre email  
5. Se connecter

---

# 🧩 Étape 4 — Créer une Organization Terraform Cloud

1. En haut à gauche, cliquer sur votre avatar  
2. Choisir **Create new organization**  
3. Donner un nom unique, par exemple :  
   - `tfc-lab-selma`  
   - `tfc-lab-adam`  
4. Choisir : **Start from scratch**

Votre Organization est prête.

---

# 🧩 Étape 5 — Générer un User API Token Terraform Cloud

1. En haut à droite, cliquer sur votre avatar → **User Settings**  
2. Cliquer sur **Tokens**  
3. Sous *User API Token*, cliquer **Create an API Token**  
4. Nommer : `codespace-token`  
5. Copier le token affiché (vous ne le reverrez plus)

---

# 🧩 Étape 6 — Authentifier Terraform CLI dans le Codespace

Dans le **terminal du Codespace**, exécuter :

```bash
terraform login
```

1. Répondre **yes**  
2. Un lien s’ouvre dans le navigateur → collez votre token  
3. Vérifiez la connexion :

```bash
terraform -version
```

---

# 🧩 Étape 7 — Vérifier l’accès Terraform Cloud via l’API

Installez `jq` si nécessaire :

```bash
sudo apt-get update && sudo apt-get install -y jq
```

Puis exécutez :

```bash
curl   --header "Authorization: Bearer $(jq -r '."app.terraform.io".token' ~/.terraform.d/credentials.tfrc.json')"   https://app.terraform.io/api/v2/user
```

Vous devez obtenir un JSON contenant votre username, email, etc.

---

# 🧩 Étape 8 — Validation du Lab

✔ Repo GitHub créé depuis le template  
✔ Codespace fonctionnel  
✔ Compte Terraform Cloud opérationnel  
✔ Organisation TFC créée  
✔ API Token généré  
✔ `terraform login` fonctionne  
✔ API `/api/v2/user` renvoie vos infos  

🎉 **Fin du Lab 1 – Vous êtes prêts pour le Lab 2 !**

