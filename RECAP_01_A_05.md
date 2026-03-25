# Récapitulatif du Projet DevOps (Étapes 01 à 05)

Ce document sert de pense-bête pour comprendre l'architecture du projet, les technologies mises en place et la façon d'utiliser ce qui a été construit durant la première partie de votre cursus.

---

## 01 & 02 : Intégration Continue et GitHub Actions
*L'usine logicielle (CI)*

**But :** Automatiser les tests, le build Docker et préparer le déploiement dès que quelqu'un pousse du code sur le dépôt.

**Ce qui a été créé :**
- `.github/workflows/ci.yml` : Le pipeline principal. C'est le chef d'orchestre qui se réveille à chaque `push` ou `pull_request`. Il lance les tests et gère le déploiement en séparant les environnements (`staging` vs `production`).
- `.github/workflows/reusable-docker.yml` : Un workflow "réutilisable" (une sorte de fonction). Il ne fait qu'une seule chose mais il le fait bien : compiler et packager l'image Docker de votre application. Il est appelé par `ci.yml`.
- `.github/actions/setup-tools/action.yml` : Une "Action Composite". C'est un mini-script qui installe automatiquement les logiciels nécessaires (Docker, Terraform, Ansible...) sur les machines temporaires de GitHub.

**Comment l'utiliser :**
Tout est automatisé. Il suffit de faire un `git commit` et un `git push` vers GitHub. Vous pourrez voir les engrenages tourner dans l'onglet "Actions" de votre dépôt GitHub.

---

## 03 & 04 : L'Infrastructure as Code (Terraform)
*L'architecte de vos fondations*

**But :** Créer vos serveurs automatiquement via du code (ici simulés par de simples conteneurs Docker en local sur votre PC), plutôt que de cliquer manuellement sur le site d'un hébergeur (AWS, Azure, etc.). Le code de l'étape 04 a par la suite été rendu 100% "modulaire" pour pouvoir créer 100 environnements identiques si besoin.

**Ce qui a été créé :**
- `infra/terraform/modules/webapp` et `modules/database` : Ce ne sont pas des serveurs, ce sont vos **"Plans de construction"** génériques (vos modèles). Ils sont remplis de variables pour pouvoir s'adapter à la fois au mode *Dev* et au mode *Prod*.
- `infra/terraform/environments/dev` : C'est ici que l'infrastructure prend réellement vie. Ce dossier "appelle" les plans de construction (les modules) en leur donnant les vraies valeurs et les vrais accès (grâce au fichier `terraform.tfvars`).

**Comment l'utiliser :**
1. Placez votre terminal dans `cd infra/terraform/environments/dev`
2. Construisez les fondations (téléchargement du plugin) : `terraform init`
3. Séparez vos environnements de travail : `terraform workspace new dev`
4. Déployez votre infrastructure : `terraform apply -var-file="terraform.tfvars"`

---

## 05 : Gestion de la Configuration (Ansible)
*Le décorateur d'intérieur de vos serveurs*

**But :** Une fois que Terraform a allumé vos serveurs nus, il faut s'y connecter (SSH) pour installer vos logiciels (Mise à jour, Nginx, droits d'accès, etc.). Ansible le fait de manière 100% sécurisée, automatisée, et surtout "idempotente" (on peut le relancer 100 fois, il ne cassera rien et ignorera ce qui est déjà installé).

**Ce qui a été créé :**
- `infra/ansible/docker-compose.yml` & `Dockerfile` : *Une astuce technique exceptionnelle.* Ils créent 2 faux serveurs cibles (`node1` & `node2`) et un "cerveau" (`ansible-controller`). Cela vous a évité de salir votre Windows en y installant Ansible de force. 
- `infra/ansible/inventory.yml` : Le fichier d'inventaire. C'est le carnet d'adresses d'Ansible (qui liste les serveurs, leurs adresses IP et leurs groupes).
- `infra/ansible/roles/` (`base` et `nginx`) : Les "rôles". Au lieu d'avoir un fichier de 5000 lignes, vos actions d'installation sont découpées et rangées proprement par thématique. Le rôle *base* installe de vrais utilitaires serveurs. Le rôle *Nginx* s'occupe du web.
- `infra/ansible/site.yml` : Le Master Playbook. Il est trivialement simple car il se contente de dire : *"Pour ces serveurs, applique le rôle Base. Pour ces autres, applique le rôle Nginx"*. 

**Comment l'utiliser :**
1. Allumez vos fermes de simulation (et installez le contrôleur et ses bibliothèques) : 
   `cd infra/ansible` puis `docker compose up -d`
2. Entrez *à l'intérieur* de votre cerveau Ansible (C'est depuis ce bash que vous écrivez les vraies commandes) : 
   `docker exec -it ansible-controller sh`
3. Vérifiez la connexion aux serveurs de l'inventaire : 
   `ansible all -i inventory.yml -m ping`
4. Lancez l'installation de vos rôles : 
   `ansible-playbook -i inventory.yml site.yml`

---

## ⚠️ Problèmes rencontrés (et résolus)

### 1. Le bug du Plugin Docker introuvable (Terraform)
Lors du passage à l'architecture modulaire (Étape 04), Terraform retournait une erreur fatale (`Could not retrieve the list of available versions for provider hashicorp/docker`).
**La raison :** Nous avions bien spécifié que le plugin `docker` n'était pas l'officiel (Hashicorp) mais venait de `kreuzwerker` dans l'environnement de dev, mais **les modules Terraform vivent en totale autarcie**. Un module ignore complètement le contexte dans lequel il est appelé.
**La solution :** Il a fallu ajouter manuellement le bloc déclaration `terraform { required_providers { docker = { source = "kreuzwerker/docker" } } }` au tout début des fichiers `main.tf` de **chaque module** (`webapp` et `database`).

### 2. Les erreurs "Python introuvable" et "sudo: not found" (Ansible)
Ansible n'arrivait pas à "ping" les faux serveurs et crashait totalement lors de l'exécution du playbook avec les droits d'administration (`become: true`).
**La raison :** L'image Docker `ubuntu:22.04` brute est conçue pour être la plus légère possible. Par défaut, elle ne possède **pas Python 3** installé (alors qu'Ansible *doit* avoir Python sur la machine distante pour fonctionner), ni la commande **sudo**.
**La solution :** Sans toucher à Ansible, la méthode la plus élégante a été d'éditer notre `docker-compose.yml` de simulation local, pour dire à Docker "au moment de démarrer les noeuds Ubuntu, met les à jour et installe `python3` ainsi que `sudo` juste avant de les laisser allumés à l'infini".
*(La ligne exacte qui a sauvé la mise est : `command: /bin/sh -c "apt-get update && apt-get install -y python3 sudo && sleep infinity"`).*
