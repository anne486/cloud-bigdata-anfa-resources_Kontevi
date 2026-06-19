# Rendu Séance 1
**Nom et prénom :** KONTEVI Akossiwa Anne
## Résumé de la séance
## Étapes principales
## Capture d'écran
## Difficultés rencontrées
## Exercices d'application

Exercices d'application - Séance 1
Exercice 1 : QCM conceptuel


1.1
Réponse : D. Open source obligatoire

Justification : Le NIST définit 5 caractéristiques essentielles du cloud computing (libre-service à la demande, accès réseau large, mutualisation des ressources, élasticité rapide et service mesuré) mais n'impose pas l'open source comme obligation.

1.2
Réponse : C. SaaS

Justification : Gmail est une application complète accessible via navigateur, fournie et maintenue par Google, ce qui correspond au modèle Software as a Service où l'utilisateur n'a pas à gérer l'infrastructure sous-jacente.

1.3
Réponse : D. FaaS

Justification : Le besoin d'exécuter une fonction à chaque événement (arrivée de position GPS) sans serveur dédié permanent correspond parfaitement au modèle Function as a Service (serverless).

1.4
Réponse : C. Cloud hybride

Justification : Un cloud hybride permet de conserver les données sensibles dans un environnement privé/contrôlé tout en utilisant le cloud public pour les analyses non sensibles bénéficiant de l'élasticité.

1.5
Réponse : B. La situation où une entreprise ne peut plus changer de fournisseur sans coûts ou risques majeurs

Justification : Le vendor lock-in désigne la dépendance technologique et contractuelle qui rend difficile et coûteux le changement de fournisseur cloud.

1.6
Réponse : C. Un service open source est forcément moins performant qu'un service managé propriétaire

Justification : Cette affirmation est fausse car la performance dépend de nombreux facteurs (architecture, optimisation, cas d'usage) et de nombreux services open source sont très performants voire utilisés comme base par des services managés.

Exercice 3.1 : Commande docker run
-d : Détache le conteneur (le lance en arrière-plan) afin que le terminal reste disponible.

--name analyse-anfa : Donne le nom "analyse-anfa" au conteneur pour pouvoir le référencer facilement dans d'autres commandes.

-p 8888:8888 : Redirige le port 8888 de l'hôte vers le port 8888 du conteneur, permettant d'accéder à Jupyter depuis le navigateur.

-v /home/koffi/notebooks:/notebooks : Monte le répertoire local /home/koffi/notebooks dans le conteneur à l'emplacement /notebooks pour persister les données.

-e JUPYTER_TOKEN=anfa-token : Définit une variable d'environnement pour définir le token d'accès à Jupyter.

jupyter/pyspark-notebook : Spécifie l'image Docker à utiliser (Jupyter avec support PySpark).

Explication globale : Cette commande lance en arrière-plan un conteneur Jupyter/PySpark nommé "analyse-anfa", accessible sur le port 8888 avec un token sécurisé, et persiste les notebooks dans le répertoire local.

Exercice 3.2 : Lecture docker-compose.yml
Ce fichier docker-compose définit un service MinIO avec les caractéristiques suivantes :

Utilise l'image officielle MinIO en version latest

Nomme le conteneur "anfa-minio" et le redémarre automatiquement en cas d'arrêt

Expose les ports 9000 (API S3) et 9001 (console web)

Configure les identifiants administrateur (anfa-admin / secret)

Monte un volume persistant "minio-data" pour conserver les données

Lance MinIO en mode serveur avec la console web sur le port 9001

Exercice 4 : Diagnostic
a. Cause précise de l'erreur : Le script utilise des identifiants incorrects. Il tente de se connecter avec "anfa-admin" et "anfa-password-2026" alors que l'étudiant a créé une clé applicative nommée "anfa-app-key" avec le secret "anfa-app-secret-2026".

b. Correction du code :

python
import boto3
s3 = boto3.client(
    "s3",
    endpoint_url="http://localhost:9000",
    aws_access_key_id="anfa-app-key",  # Correction ici
    aws_secret_access_key="anfa-app-secret-2026",  # Correction ici
    region_name="us-east-1",
)
s3.upload_file("trajets.csv", "anfa-raw", "trajets.csv")
c. Pourquoi les identifiants administrateur ne fonctionnent pas : MinIO distingue les comptes administrateur (pour la console web et la gestion) des clés d'accès applicatives (pour les appels API S3). Les identifiants administrateur ne peuvent pas être utilisés directement pour les opérations S3 via l'API, il faut créer des clés d'accès dédiées via mc ou la console.

Exercice 5 : Mini-cas d'architecture
a. Limites de l'architecture actuelle :

Scalabilité limitée : Le PC portable de Toyi ne peut pas supporter des calculs intensifs ni des pics de demande.

Manque de réactivité : L'export CSV mensuel ne permet pas des prédictions horaires en quasi temps réel.

b. Correspondance besoins-cloud (NIST) :

Besoin	Caractéristique NIST	Justification
Prédictions horaires	Élasticité rapide	Permet d'ajuster les ressources de calcul automatiquement selon les besoins
Tableau de bord partagé	Accès réseau large	Accessible depuis n'importe quel poste sans installation
Augmentation capacité pics	Élasticité rapide	Les ressources peuvent être augmentées à la demande
Maîtrise des coûts	Service mesuré	Paiement à l'usage, pas de surprovisionnement
Conformité données clients	Mutualisation des ressources	Isolation sécurisée des données sensibles
c. Modèles de service proposés :

(i) Tableau de bord partagé : SaaS (ex: Metabase Cloud, Power BI) - Solution clé en main sans gestion d'infrastructure.

(ii) Calcul des prédictions : FaaS (ex: AWS Lambda, Google Cloud Functions) - Exécution déclenchée toutes les heures, sans serveur permanent.

(iii) Stockage des données clients : IaaS ou PaaS (ex: base de données managée) - Besoin de contrôle sur la sécurité et la conformité.

d. Modèle de déploiement recommandé : Cloud hybride

Justification : Ce modèle permet de conserver les données clients sensibles dans un environnement privé (on-premise ou cloud privé) pour garantir la conformité, tout en utilisant le cloud public pour les calculs élastiques et le tableau de bord. Les données non sensibles et les traitements peuvent bénéficier de l'élasticité du cloud public.

e. Trois stratégies anti-vendor lock-in :

Conteneurisation : Utiliser Docker/Kubernetes pour rendre les applications portables entre fournisseurs.

Outils open source : Privilégier des solutions comme MinIO (stockage), PostgreSQL (base de données), Metabase (BI) qui peuvent être déployées chez n'importe quel fournisseur.

API standards : Utiliser des API ouvertes et normalisées (comme l'API S3 pour le stockage) plutôt que des services propriétaires spécifiques à un fournisseur.

