# Rendu — Séance 8

**Nom et prénom :** KONTEVI Akossiwa Anne
**Identifiant GitHub :** anne486
**Date de soumission :** 07/07/2026

## Résumé de la séance

Au cours de cette séance, nous avons séparé la logique métier du DAG Airflow afin de la rendre indépendante et facilement testable. Nous avons ensuite écrit des tests unitaires avec pytest ainsi qu'un pipeline GitHub Actions exécutant automatiquement le lint et les tests à chaque push. Enfin, nous avons vérifié que le pipeline bloque le déploiement lorsqu'un test échoue et qu'il autorise le déploiement (simulation) lorsque tous les tests sont validés.

## Étapes principales

1. Séparation de la logique métier (`anfa_logic.py`) du DAG Airflow.
2. Écriture de 5 tests unitaires avec pytest.
3. Écriture du workflow GitHub Actions (lint + tests + déploiement simulé).
4. Démonstration : un bug volontaire bloque le déploiement ; correction et succès.

## Captures d'écran

### Workflow réussi (2 jobs)
![CI succès](captures/ci-succes.png)

### Job en échec, déploiement non exécuté
![CI échec](captures/ci-echec.png)

## Réflexion personnelle

Ce pipeline aurait permis d'éviter l'incident de Mawuli en détectant automatiquement les erreurs grâce aux tests unitaires avant toute mise en production. Ainsi, un code défectueux aurait été bloqué dès la phase d'intégration continue. Le mot-clé needs: dans GitHub Actions permet d'imposer un ordre d'exécution entre les jobs : le déploiement ne démarre que si les étapes précédentes, comme le lint et les tests, se terminent avec succès. Cela garantit qu'un code non validé ne peut pas être déployé.

## Difficultés rencontrées

<Aucune | Décrivez brièvement.>
