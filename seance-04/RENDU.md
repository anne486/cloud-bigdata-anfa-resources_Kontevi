# Rendu — Séance 4
**Nom et prénom :** Kontevi akossiwa anne 
**Identifiant GitHub :** anne486
**Date de soumission :** 27/06/2026

## Résumé de la séance
Terraform installé avec succès. Infrastructure Docker complète (réseau, volume, conteneur MinIO) décrite en HCL. Workflow init/plan/apply/destroy maîtrisé. Code paramétré via variables avec fichiers .tfvars. Seul bémol : l'image MinIO n'a pas pu être supprimée car utilisée par un conteneur d'une séance précédente.

## Étapes principales
1. Installation de Terraform et premier `main.tf` minimal.
2. Maîtrise du workflow `init` → `plan` → `apply` → `destroy`.
3. Compréhension du state Terraform et bonnes pratiques de versioning.
4. Stack complète : réseau, volume, conteneur MinIO.
5. Refactoring en variables et fichier `.tfvars`.
## Captures d'écran
### terraform plan (création initiale)
![terraform plan](captures/terraform-plan.png)
### terraform apply réussi
![terraform apply](captures/terraform-apply.png)
### Console MinIO créée par Terraform
![Console MinIO](captures/console-minio-tf.png)
### terraform destroy
![terraform destroy](captures/terraform-destroy.png)
## Réponses aux exercices d'application
<Exercice 1 - QCM conceptuel
1.1 Parmi ces affirmations sur l'Infrastructure as Code, laquelle est fausse ?
Réponse : B. L'IaC remplace totalement la nécessité de comprendre l'infrastructure sous-jacente.

Justification : L'IaC est un outil puissant mais ne dispense pas de comprendre l'infrastructure ; il faut toujours savoir ce qu'on déploie et comment les ressources fonctionnent.

1.2 Quelle est la différence fondamentale entre une approche déclarative et une approche impérative ?
Réponse : B. Le déclaratif décrit l'état souhaité ; l'impératif décrit la séquence d'actions à effectuer.

Justification : Terraform est déclaratif : on décrit ce qu'on veut, il détermine comment y arriver, contrairement à un script impératif qui spécifie chaque étape.

1.3 Que signifie qu'une opération est idempotente ?
Réponse : B. Elle produit le même résultat quel que soit le nombre de fois où elle est appliquée.

Justification : L'idempotence garantit qu'exécuter plusieurs fois la même opération ne change pas le résultat final après la première exécution.

1.4 À quoi sert un provider dans Terraform ?
Réponse : B. À fournir un plugin qui sait communiquer avec une API spécifique (AWS, Docker, Kubernetes...).

Justification : Le provider est le pont entre Terraform et l'API du service cible, il implémente les opérations CRUD sur les ressources.

1.5 Que se passe-t-il si vous lancez terraform apply deux fois de suite sans modifier votre code ?
Réponse : B. Terraform compare le state au code, ne voit aucun écart, et n'effectue aucune action.

Justification : C'est le principe même de l'idempotence de Terraform : il vérifie que l'état réel correspond à la configuration déclarée.

1.6 Quelle est la fonction du fichier terraform.tfstate ?
Réponse : C. Mémoriser ce que Terraform a créé pour pouvoir suivre les changements incrémentaux.

Justification : Le state est la source de vérité qui permet à Terraform de savoir ce qui existe et de calculer les différences.

1.7 Pourquoi ne faut-il jamais committer le fichier terraform.tfstate dans Git ?
Réponse : B. Parce qu'il peut contenir des secrets en clair (mots de passe, clés API) et peut être corrompu par des commits concurrents.

Justification : Le state contient des données sensibles en clair et n'est pas conçu pour être partagé via Git, d'où l'utilisation des backends distants.

1.8 Quelle commande exécutez-vous avant terraform apply pour vérifier ce qui va changer ?
Réponse : C. terraform plan

Justification : terraform plan est la commande de prévisualisation qui montre exactement ce qui sera créé, modifié ou détruit.

1.9 Que représente OpenTofu ?
Réponse : B. Un fork open source de Terraform créé après le changement de licence de HashiCorp en 2023.

Justification : OpenTofu est le fork communautaire maintenu par la Linux Foundation suite au passage de Terraform en licence BSL.

1.10 Terraform et Ansible sont-ils des outils concurrents ?
Réponse : B. Non, Terraform provisionne l'infrastructure, Ansible configure des machines existantes — ils sont complémentaires.

Justification : Terraform s'occupe du provisioning (création des ressources), Ansible de la configuration (installation de logiciels, déploiement d'applications).

Exercice 2 - Lecture et interprétation d'un fichier Terraform
2.1 Listez les 4 resources définies dans ce fichier
docker_network.back : Crée un réseau Docker nommé "anfa-backend" pour permettre la communication entre les conteneurs.

docker_volume.data : Crée un volume persistant nommé "postgres-data" pour stocker les données de PostgreSQL.

docker_image.postgres : Télécharge l'image Docker PostgreSQL version 15 qui servira à créer le conteneur.

docker_container.db : Crée et démarre un conteneur PostgreSQL nommé "anfa-postgres" avec ses configurations (ports, volumes, variables d'environnement).

2.2 Dans la ligne image = docker_image.postgres.image_id
Cette référence correspond à l'ID de l'image Docker PostgreSQL 15 qui a été téléchargée par la ressource docker_image.postgres.

Avantages par rapport à image = "postgres:15" :

Dépendance explicite : Terraform sait qu'il doit d'abord télécharger/télécharger l'image avant de créer le conteneur.

Traçabilité : L'image est gérée comme une ressource Terraform, donc elle apparaît dans le state.

Mise à jour contrôlée : On peut modifier la version de l'image dans un seul endroit (la ressource image).

ID exact : L'utilisation de image_id garantit qu'on utilise l'image exacte qui a été téléchargée (même si le tag "postgres:15" est mis à jour entre-temps).

2.3 Ordre de création lors du premier terraform apply
Terraform créera les ressources dans cet ordre :

docker_volume.data (volume)

docker_network.back (réseau)

docker_image.postgres (image)

docker_container.db (conteneur)

Raison : Le conteneur dépend des trois autres ressources (il référence le volume, le réseau et l'image). Terraform construit un graphe de dépendances et crée d'abord les dépendances avant ce qui en dépend.

2.4 Problème principal de sécurité
Problème : Le mot de passe PostgreSQL est en clair dans le code (POSTGRES_PASSWORD=secret123).

Correction :

hcl
# variables.tf
variable "postgres_password" {
  description = "Mot de passe pour l'utilisateur PostgreSQL"
  type        = string
  sensitive   = true
}

# terraform.tfvars (non versionné)
postgres_password = "secret123"

# main.tf
resource "docker_container" "db" {
  # ...
  env = [
    "POSTGRES_DB=anfa",
    "POSTGRES_USER=anfa_user",
    "POSTGRES_PASSWORD=${var.postgres_password}",
  ]
  # ...
}
2.5 Comportement après terraform destroy et modification du port
Ce que Terraform fera :

Après terraform destroy, le state est vide.

Quand l'étudiant modifie external = 5432 en external = 5433 et relance apply :

Terraform va créer un nouveau conteneur avec le port 5433.

Rien n'est détruit car le précédent a déjà été supprimé par destroy.

Remarque : Si le conteneur existait encore, Terraform aurait dû le détruire et recréer car le port est un paramètre qui ne peut pas être modifié à chaud sur un conteneur Docker en cours d'exécution.

Exercice 3 - Diagnostic
3.1 - L'apply qui échoue avec une dépendance circulaire
a. Signification de l'erreur
L'erreur "Cycle" signifie que Terraform a détecté une dépendance circulaire entre les deux conteneurs : chacun dépend de l'autre pour sa création.

b. Pourquoi Terraform refuse d'appliquer
Terraform ne peut pas déterminer quel conteneur créer en premier car :

Le conteneur A nécessite le nom du conteneur B (via LINKED_TO)

Le conteneur B nécessite le nom du conteneur A (via LINKED_TO)

C'est un paradoxe : aucun des deux ne peut exister sans l'autre.

c. Solution pour résoudre le problème

hcl
# Solution 1 : Supprimer la dépendance circulaire
resource "docker_container" "a" {
  name  = "container-a"
  image = "alpine"
  # On retire la référence à b dans les variables
}

resource "docker_container" "b" {
  name  = "container-b"
  image = "alpine"
  env   = ["LINKED_TO=${docker_container.a.name}"]  # Dépendance uniquement de b vers a
}

# Solution 2 : Utiliser les noms statiques
resource "docker_container" "a" {
  name  = "container-a"
  image = "alpine"
  env   = ["LINKED_TO=container-b"]  # Nom statique
}

resource "docker_container" "b" {
  name  = "container-b"
  image = "alpine"
  env   = ["LINKED_TO=container-a"]  # Nom statique
}
3.2 - Le plan qui veut tout recréer
a. Pourquoi -/+ plutôt que ~
Terraform marque le conteneur avec -/+ car les variables d'environnement ne peuvent pas être modifiées sur un conteneur Docker en cours d'exécution. Docker nécessite un redémarrage complet du conteneur, donc Terraform doit le détruire et le recréer.

b. Les données seront-elles perdues ?
Non, les données ne seront pas perdues car elles sont persistées dans le volume Docker (docker_volume.minio_data). Terraform détruit le conteneur mais conserve le volume associé, puis le rattache au nouveau conteneur.

c. Impact opérationnel en production
Cette opération n'est pas gratuite en production :

Temps d'arrêt : Le service sera indisponible pendant la recréation (quelques secondes/minutes).

Impact utilisateur : Toutes les sessions actives seront perdues.

Risque de cache : Les caches en mémoire seront vidés.

Stratégie recommandée : Pour éviter cela en production, utiliser des stratégies de déploiement sans interruption (blue/green, rolling update) ou une orchestration comme Kubernetes.

3.3 - Le state corrompu
a. Problème de sécurité immédiat
Le fichier terraform.tfstate contient le mot de passe MinIO en clair (anfa-password-2026). En le poussant sur GitHub, ces secrets deviennent publics (ou accessibles à tous les collaborateurs ayant accès au dépôt).

b. Risque technique quand Awa applique avec ce state récupéré
Awa risque de :

Corrompre le state : Si elle applique des modifications sur sa machine avec un state obsolète.

Créer des conflits : Deux personnes modifiant le même state créent une incohérence.

Perdre la traçabilité : Les modifications faites par l'un ne sont pas visibles par l'autre.

Créer des duplications : Terraform pourrait essayer de recréer des ressources qui existent déjà.

c. Solution pérenne pour l'équipe

Utiliser un backend distant (S3, Terraform Cloud, Azure Storage) pour stocker le state en commun.

Configurer le verrouillage d'état (state locking) pour éviter les modifications concurrentes.

Ajouter .tfstate dans .gitignore pour ne jamais le versionner.

Utiliser des variables sensibles (sensitive = true) pour protéger les secrets dans les outputs.

Mettre en place des revues de code avant tout apply sur l'infrastructure critique.

Exercice 4 - Adaptation Compose → Terraform
hcl
# versions.tf
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0"
    }
  }
}

provider "docker" {}

# ===== Variables =====
variable "minio_root_user" {
  description = "Utilisateur root pour MinIO"
  type        = string
  default     = "anfa-admin"
}

variable "minio_root_password" {
  description = "Mot de passe root pour MinIO"
  type        = string
  sensitive   = true
}

variable "jupyter_token" {
  description = "Token d'accès à Jupyter"
  type        = string
  default     = "anfa-token"
}

# ===== Réseau =====
resource "docker_network" "anfa_network" {
  name = "anfa-network"
}

# ===== Volume =====
resource "docker_volume" "minio_data" {
  name = "minio-data"
}

# ===== Image MinIO =====
resource "docker_image" "minio" {
  name = "minio/minio:latest"
}

# ===== Conteneur MinIO =====
resource "docker_container" "minio" {
  name  = "anfa-minio"
  image = docker_image.minio.image_id

  command = ["server", "/data", "--console-address", ":9001"]
  restart = "unless-stopped"

  ports {
    internal = 9000
    external = 9000
  }

  ports {
    internal = 9001
    external = 9001
  }

  env = [
    "MINIO_ROOT_USER=${var.minio_root_user}",
    "MINIO_ROOT_PASSWORD=${var.minio_root_password}",
  ]

  volumes {
    volume_name    = docker_volume.minio_data.name
    container_path = "/data"
  }

  networks_advanced {
    name = docker_network.anfa_network.name
  }

  lifecycle {
    ignore_changes = [log_opts]
  }
}

# ===== Image Jupyter =====
resource "docker_image" "jupyter" {
  name = "jupyter/scipy-notebook:latest"
}

# ===== Conteneur Jupyter =====
resource "docker_container" "jupyter" {
  name  = "anfa-jupyter"
  image = docker_image.jupyter.image_id

  restart = "unless-stopped"

  ports {
    internal = 8888
    external = 8888
  }

  env = [
    "JUPYTER_TOKEN=${var.jupyter_token}",
  ]

  networks_advanced {
    name = docker_network.anfa_network.name
  }

  # Dépendance implicite : Terraform sait que Jupyter doit être créé après MinIO
  # car Jupyter utilise le même réseau que MinIO (référence dans networks_advanced)

  lifecycle {
    ignore_changes = [log_opts]
  }
}
Exercice 5 - Mini-cas d'architecture
5.1 Types de ressources Terraform pour le cloud OVHcloud
Object Storage (bucket) : Stockage des fichiers CSV, logs GPS, données brutes.

Compute Instance (VM/Instance) : Serveur virtuel pour Spark (master et workers).

Kubernetes Managed Cluster : Orchestration des conteneurs pour les microservices.

Load Balancer : Distribution du trafic vers Grafana (accès public).

Database (Managed PostgreSQL/MySQL) : Base de données pour les métadonnées.

VPC / Network : Réseau privé isolant les ressources entre elles.

Security Group / Firewall : Règles de sécurité pour protéger les ressources.

5.2 Organisation des fichiers Terraform
Recommandation : B. Plusieurs fichiers séparés (network.tf, storage.tf, compute.tf, monitoring.tf)

Justification :

Modularité : Chaque fichier gère un domaine spécifique, facilitant la maintenance.

Lisibilité : Un fichier de 800 lignes est difficile à comprendre et réviser.

Travail en équipe : Plusieurs personnes peuvent travailler sur différents fichiers sans conflits.

Réutilisation : On peut facilement réutiliser un module ou un fichier dans d'autres projets.

5.3 Mécanismes pour gérer dev et prod
Workspaces : terraform workspace new dev et terraform workspace new prod pour isoler les états.

Fichiers de variables : terraform.tfvars.dev et terraform.tfvars.prod utilisés avec -var-file.

Variables d'environnement : Utiliser des variables système avec TF_VAR_ pour chaque environnement.

Backends différents : Chaque environnement peut avoir son propre backend de state.

5.4 Migration vers un nouveau fournisseur cloud
Réponse au directeur technique :

La migration demandera un effort significatif mais maîtrisé :

Ce qui se transpose facilement :

La logique métier (traitements Spark, code des applications) reste inchangée.

Le code Terraform (structure, dépendances, variables) est réutilisable.

Ce qui demandera du travail :

Les providers : OVHcloud → AWS nécessite de réécrire les resources avec les types AWS équivalents.

Les noms et types de resources : Un bucket S3 n'a pas la même API qu'un container OVHcloud.

Les réseaux : VPC, sous-réseaux, routes doivent être redéfinis.

Les IAM/identités : Les politiques de sécurité sont spécifiques à chaque cloud.

Les tests : Il faut valider que tout fonctionne sur le nouveau cloud.

Estimation : 2-4 semaines de travail pour la migration complète, avec une phase de test avant la mise en production.

5.5 Bonnes pratiques pour l'équipe de 4 personnes
Repository Git : Un dépôt unique avec une branche principale protégée et des branches de feature.

Code review obligatoire : Validation en équipe avant tout apply sur l'infrastructure.

Backend distant : Utiliser un backend partagé (S3/Terraform Cloud) avec verrouillage d'état.

Prévisualisation systématique : Toujours faire terraform plan avant apply.

Variables sensibles : Utiliser sensitive = true et ne jamais versionner .tfvars.

Formatage du code : Utiliser terraform fmt pour un code uniforme.

Modules réutilisables : Créer des modules pour les composants récurrents.

CI/CD : Mettre en place une pipeline d'intégration continue pour valider automatiquement le code.

Documentation : Maintenir un README avec les prérequis et les commandes importantes.

Tests d'infrastructure : Utiliser terraform validate et terraform plan -detailed-exitcode pour détecter les erreurs tôt.



## Difficultés rencontrées
Lors du `terraform destroy`, l'image MinIO n'a pas pu être supprimée car elle était encore utilisée par un conteneur d'une séance précédente (`1ae79258accd`). Solution : supprimer manuellement le conteneur avec `docker rm -f` avant de relancer `terraform destroy`.

<Aucune | Décrivez brièvement.>