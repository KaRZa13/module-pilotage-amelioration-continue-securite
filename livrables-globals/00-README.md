# TP1 — NOVA SERVICES : identifier le SI, les actifs critiques et les besoins de sécurité

Dossier de livrables du TP1 du module *Pilotage et amélioration continue de la sécurité*.
Ces documents constituent le socle réutilisé dans les TP suivants (analyse de risques, indicateurs, tableau de bord).

## Sommaire

| Fichier | Contenu | Partie de l'énoncé |
|---|---|---|
| [01-analyse-initiale-SI.md](01-analyse-initiale-SI.md) | Exercices 1 à 5 + mise en situation (5 événements) | Partie 2 |
| [L1-analyse-activites-metier.md](L1-analyse-activites-metier.md) | **Livrable 1** — Activités métier ↔ composants techniques | Partie 3 |
| [L2-registre-actifs.md](L2-registre-actifs.md) | **Livrable 2** — Registre de 48 actifs ou groupes d'actifs | Partie 5 |
| [L3-inventaire-donnees.md](L3-inventaire-donnees.md) | **Livrable 3** — Inventaire de 12 catégories de données + réflexion 4.3 | Partie 4 |
| [L4-evaluation-criticite.md](L4-evaluation-criticite.md) | **Livrable 4** — Notation C / I / D / impact métier | Partie 6 |
| [L5-cartographie-SI.md](L5-cartographie-SI.md) | **Livrable 5** — Cartographie du SI (Mermaid, légende) | Partie 8 |
| [L6-exposition-dependances.md](L6-exposition-dependances.md) | **Livrable 6** — Exposition de tous les actifs + dépendances des 5 actifs prioritaires | Partie 7 |
| [L7-besoins-et-matrice-pilotage.md](L7-besoins-et-matrice-pilotage.md) | **Livrable 7** — Besoins de sécurité + première matrice de pilotage | Parties 9 et 11 |
| [L8-analyse-incidents.md](L8-analyse-incidents.md) | **Livrable 8** — INC-001 et INC-002 + critères US-SEC-001 | Partie 13 |
| [L9-synthese-decisionnelle.md](L9-synthese-decisionnelle.md) | **Livrable 9** — Synthèse d'une page + choix des priorités | Partie 12 |
| [10-referentiel-ANSSI.md](10-referentiel-ANSSI.md) | Trois points de vérification au regard du guide d'hygiène ANSSI | Partie 10 |
| [11-questions-comprehension.md](11-questions-comprehension.md) | Réponses aux 12 questions de compréhension | Partie 16 |

## Conventions utilisées dans tout le dossier

### Échelle de criticité (convention du TP)

| Note | Niveau | Interprétation |
|---|---|---|
| 1 | Faible | Conséquences limitées, solution de remplacement simple |
| 2 | Modéré | Perturbation contenue, impact maîtrisable |
| 3 | Important | Perturbation notable ou conséquences significatives |
| 4 | Critique | Impact majeur sur l'activité ou sur plusieurs services essentiels |

La **criticité retenue** correspond au critère le plus défavorable (conséquence la plus grave), et non à une moyenne. Tout écart à cette règle est signalé et justifié.

### Statut des informations

Pour éviter de raisonner sur une architecture supposée, chaque affirmation est qualifiée :

| Marqueur | Signification |
|---|---|
| **[C]** | Confirmé par la description du SI fournie par la DSI (énoncé) |
| **[H]** | Hypothèse de travail raisonnable, à valider |
| **[V]** | Information manquante — à confirmer auprès de la DSI |

### Propriétaires

L'énoncé ne nomme aucun responsable. Tous les actifs sont donc marqués **« propriétaire à désigner »**, avec le *rôle pressenti* qui devrait l'endosser. Aucun nom fictif n'est utilisé.

### Identifiants

- `A01`…`A48` : actifs du registre (L2), réutilisés dans tous les livrables.
- `D01`…`D12` : catégories de données (L3).

## Hypothèses globales et informations manquantes

| # | Sujet | Statut | Détail |
|---|---|---|---|
| H1 | Hébergement des applications métier | [H] | Les applications de gestion des clients / facturation et de production sont supposées tourner sur Kubernetes (interne et/ou EKS) et être publiées via l'Ingress ou l'AWS Load Balancer. La répartition interne / AWS n'est pas précisée. |
| H2 | Exposition de Keycloak | [H] | Keycloak est hébergé sur les serveurs Linux internes [C]. Comme il authentifie des applications accessibles depuis Internet et fédère des services externes, sa page d'authentification est supposée joignable depuis Internet. |
| H3 | Exposition de GitLab | [V] | Non précisée : accessible uniquement en interne / via VPN, ou publié sur Internet ? |
| H4 | Registre d'images | [V] | L'énoncé cite « Harbor ou GitLab Registry ou Amazon ECR » ; le registre réellement utilisé en production est à confirmer. |
| H5 | Sauvegardes | [C]/[V] | S3 stocke des sauvegardes [C]. L'existence d'une copie hors ligne / immuable et les tests de restauration ne sont pas documentés [V]. |
| H6 | Secrets | [C]/[V] | Vault protège les secrets [C]. La présence de secrets ailleurs (Secrets Kubernetes, variables CI/CD GitLab, état Terraform, inventaires Ansible) est probable mais à confirmer [V]. |
| H7 | Discord | [V] | Cité comme messagerie : usage officiel, toléré ou non maîtrisé ? Données échangées ? |
| H8 | Collecte des journaux AWS | [V] | L'énoncé décrit la collecte Keycloak / Linux / OPNsense / GitLab / Kubernetes / Suricata vers Wazuh, mais ne mentionne pas de source d'audit AWS (type CloudTrail). |
| H9 | Isolation du Wi-Fi visiteurs | [C]/[V] | L'énoncé affirme qu'il est isolé [C] ; l'efficacité réelle de l'isolation n'est pas vérifiée [V]. |
| H10 | Procédure de gestion des incidents | [V] | Aucune procédure interne n'est fournie (qui alerter, astreinte, RSSI). |
