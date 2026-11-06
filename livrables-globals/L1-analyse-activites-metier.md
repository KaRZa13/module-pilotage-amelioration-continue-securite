# Livrable 1 — Analyse des activités métier

> Partie 3 de l'énoncé. Identifiants d'actifs : voir [L2](L2-registre-actifs.md) ; catégories de données : voir [L3](L3-inventaire-donnees.md).

Raisonnement suivi : **activité métier → données et services nécessaires → composants techniques → conséquences d'un incident → priorités de sécurité.**

## 1. Tableau de correspondance activités ↔ composants

| Activité métier | Données manipulées | Applications ou services | Composants techniques dépendants |
|---|---|---|---|
| **A — Gestion des clients** (A01) | Coordonnées clients, contrats, prestations vendues, facturation, documents commerciaux (D01, D03) | Application(s) métier de gestion clients / facturation [H : nom et périmètre exact à confirmer] ; stockage documentaire (SharePoint, Samba) ; messagerie | Applications Web et API (A16, A17) sur Kubernetes (A22) ; PostgreSQL interne ou RDS (A18 / A19) [V] ; Keycloak (A14) ; SharePoint / Microsoft 365 (A37) ; Samba (A21) ; Ingress / Load Balancer (A22, A34) ; OPNsense (A05) ; sauvegardes (A36, A33) |
| **B — Production des services** (A02) | Projets, affectations des collaborateurs, suivi des prestations, temps passés, événements métier (D02, D11) | Applications Web de gestion de projets, API, microservices, traitements automatisés | Kubernetes interne et EKS (A22-A25) ; PostgreSQL (A18) ; Kafka (A20) ; Keycloak (A14) ; registre d'images (A26) ; Vault (A35) ; DNS interne (A11) ; sauvegardes (A36) |
| **C — Communication et collaboration** (A03) | Messages, pièces jointes, documents partagés, calendriers (D03, D12) | Microsoft 365 (messagerie, SharePoint), Discord [V], serveur de fichiers | Microsoft 365 (A37) ; Discord (A38) ; fédération Keycloak (A14) [V] ; Samba (A21) ; postes et portables (A39, A40) ; VPN collaborateurs (A06) ; proxy Squid (A10) |
| **D — Administration du SI** (A04) | Comptes, rôles, droits (D04), secrets techniques (D05), code et IaC (D06), configurations, journaux de sécurité (D10) | Keycloak, AWS IAM, GitLab et CI/CD, Terraform, Ansible, Vault, VPN, outils de supervision | Keycloak (A14) ; AWS IAM (A15) ; GitLab / CI/CD (A27, A28) ; Terraform (A29) ; Ansible (A30) ; Vault (A35) ; VPN collaborateurs et fournisseurs (A06, A07) ; API Kubernetes (A23) ; Wazuh / OpenSearch (A42, A43) |

## 2. Questions par activité

### A — Gestion des clients

| Question | Réponse |
|---|---|
| Activité perturbée si l'application principale devient indisponible | Gestion des contrats, suivi commercial, **facturation** (impact direct sur la trésorerie), réponse aux demandes des clients. Les commerciaux sont tous concernés (cf. INC-002). |
| Données consultables ou modifiables sans autorisation | Coordonnées et contrats clients (données personnelles des interlocuteurs → obligations RGPD), conditions tarifaires, factures (fraude au changement de RIB possible si les données sont modifiées). |
| Autres composants affectés par une compromission | PostgreSQL (données partagées avec d'autres applications), Keycloak si l'application dispose d'un client confidentiel avec des droits étendus, Kubernetes (rebond via le pod), documents SharePoint liés. |
| Qui doit contribuer à la criticité | Direction commerciale / responsable de l'ADV (propriétaire métier), direction financière (facturation), DPO (données personnelles), DSI (contraintes techniques), RSSI. |

### B — Production des services

| Question | Réponse |
|---|---|
| Activité perturbée si l'application principale devient indisponible | Planification et affectation des consultants, suivi des prestations chez les clients, pilotage des projets → retards de livraison, pénalités contractuelles possibles. |
| Données consultables ou modifiables sans autorisation | Projets et livrables clients (potentiellement confidentiels pour les clients eux-mêmes), affectations et données des collaborateurs, événements transitant par Kafka. Une modification (intégrité) fausse le suivi et la facturation à l'acte. |
| Autres composants affectés | Kafka (propagation d'événements falsifiés vers d'autres applications), PostgreSQL, API, applications consommatrices, système d'analyse. |
| Qui doit contribuer à la criticité | Direction des opérations / de la production, chefs de projet, DSI (équipe plateforme Kubernetes), RSSI. |

### C — Communication et collaboration

| Question | Réponse |
|---|---|
| Activité perturbée si l'application principale devient indisponible | Échanges internes et avec les clients, partage de documents, coordination des équipes en télétravail. Un mode dégradé existe (téléphone), d'où une criticité un cran en dessous des activités A, B et D. |
| Données consultables ou modifiables sans autorisation | Courriels (souvent porteurs de données clients et de pièces jointes contractuelles), documents SharePoint. Une boîte compromise sert de tremplin à l'hameçonnage interne ou à la fraude au président. |
| Autres composants affectés | Keycloak si la fédération est bidirectionnelle [V], postes utilisateurs (pièces jointes malveillantes), SharePoint. |
| Qui doit contribuer à la criticité | Direction générale, responsable de la communication, DSI (administrateur M365), DPO, RSSI. Le statut de Discord doit être tranché par la direction et le RSSI. |

### D — Administration du SI

| Question | Réponse |
|---|---|
| Activité perturbée si l'application principale devient indisponible | Impossibilité de gérer les comptes, de déployer, de corriger ou de restaurer : **toutes les autres activités** perdent leur capacité de reprise. Une indisponibilité de Keycloak bloque en plus l'accès des utilisateurs aux applications. |
| Données consultables ou modifiables sans autorisation | Comptes et rôles, politiques IAM, secrets (Vault), code et IaC, configurations réseau. Leur compromission donne la main sur l'ensemble du SI. |
| Autres composants affectés | Potentiellement **tous** : c'est l'activité qui détient les « clés ». Compromission d'AWS IAM → S3 (y compris les sauvegardes), EKS, RDS ; compromission de GitLab CI → production Kubernetes et AWS via Terraform. |
| Qui doit contribuer à la criticité | DSI, responsables infrastructure / Cloud / plateforme, RSSI ; la direction générale doit valider puisque les conséquences touchent toute l'entreprise. |

## 3. Conséquences possibles d'un incident par activité

| Activité | Confidentialité | Intégrité | Disponibilité | Conséquences principales |
|---|---|---|---|---|
| A — Gestion des clients | Fuite de données clients et de contrats | Factures ou coordonnées bancaires falsifiées | Facturation bloquée | Perte de confiance, sanction RGPD possible, perte de chiffre d'affaires |
| B — Production des services | Fuite de livrables / projets clients | Affectations ou suivi falsifiés | Prestations non pilotées | Retards, pénalités, insatisfaction des clients |
| C — Communication | Fuite de courriels et documents | Usurpation (fraude, hameçonnage interne) | Coordination dégradée | Fraude financière, propagation d'une compromission |
| D — Administration du SI | Vol de secrets et d'identités | Modification de droits, d'infrastructure, de code | Impossibilité d'administrer / de restaurer | **Compromission totale** du SI et perte de la capacité de reprise |

## 4. Enseignements pour la suite

- Les activités **A, B et D** sont jugées critiques (4) : A et B portent directement le chiffre d'affaires, D conditionne la sécurité de toutes les autres.
- Des composants **transverses** apparaissent dans plusieurs activités : **Keycloak, PostgreSQL, Kubernetes, sauvegardes**. Leur criticité est héritée du cumul des activités qu'ils supportent, et non de la technologie elle-même.
- Point à lever : la cartographie précise *application ↔ base de données ↔ hébergement (interne / AWS)* n'est pas fournie et doit être obtenue auprès de la DSI [V].
