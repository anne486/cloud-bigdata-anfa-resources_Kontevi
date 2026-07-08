# Rendu — Séance 9

**Nom et prénom :** KONTEVI Akossiwa anne 
**Identifiant GitHub :** anne486
**Date de soumission :** 08/07/2026

## Résumé de la séance

Au cours de cette séance, j'ai déployé une stack de monitoring avec Prometheus, Grafana, Node Exporter et cAdvisor afin de superviser l'application Anfa. J'ai appris à visualiser les métriques, créer une métrique métier sur la fraîcheur des données et configurer une alerte capable de détecter une panne.

## Étapes principales

1. Déploiement de Prometheus, Node Exporter, cAdvisor, Grafana et d'un exportateur
   métier custom (fraîcheur des données Anfa).
2. Exploration des cibles Prometheus et premières requêtes PromQL.
3. Import du dashboard "Node Exporter Full" et construction d'un panneau custom.
4. Configuration d'une alerte Grafana sur la fraîcheur des données.
5. Simulation d'une panne silencieuse et observation du déclenchement de l'alerte.

## Captures d'écran

### Les 4 cibles Prometheus à l'état UP
![Targets](captures/prometheus-targets.png)

### Dashboard "Node Exporter Full" importé
![Node Exporter Dashboard](captures/grafana-node-exporter.png)

### Alerte à l'état Firing après panne simulée
![Alerte Firing](captures/grafana-alerte-firing.png)

## Réflexion personnelle

Cette séance m'a montré que le monitoring est indispensable pour détecter rapidement les anomalies avant qu'elles n'impactent les utilisateurs. Grâce aux métriques et aux alertes Grafana, il devient possible de surveiller en temps réel l'état de l'application et d'intervenir dès qu'un problème est détecté.

## Difficultés rencontrées

<Aucune | Décrivez brièvement.>
