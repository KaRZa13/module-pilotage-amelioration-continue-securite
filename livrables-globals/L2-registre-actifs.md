# Livrable 2 — Registre des actifs

> Partie 5 de l'énoncé. 48 actifs ou groupes d'actifs couvrant toutes les zones du SI.
> Criticité : échelle 1 à 4 (voir [00-README.md](00-README.md)), détaillée critère par critère dans [L4](L4-evaluation-criticite.md).
> **Propriétaire** : aucun responsable n'est nommé dans l'énoncé → « À désigner » + rôle pressenti.

**Règles de regroupement** : des composants sont regroupés lorsqu'ils partagent le même rôle et le même niveau de risque (ex. les ~150 postes fixes). À l'inverse, le cluster Kubernetes est découpé en trois actifs (cluster/charges, API + RBAC, secrets) et GitLab en deux (dépôts / CI-CD), car leurs risques diffèrent.

---

## 1. Actifs métier

| ID | Nom | Catégorie | Propriétaire | Fonction | Données traitées | Localisation | Exposition | Dépendances | Crit. | Justification |
|---|---|---|---|---|---|---|---|---|---|---|
| A01 | Gestion des clients | Métier (activité) | À désigner (pressenti : direction commerciale) | Contrats, prestations, facturation | D01, D03 | Hybride | Héritée des applications (Internet) | A16, A17, A18/A19, A14, A21, A37 | 4 | Porte le chiffre d'affaires et des données personnelles clients |
| A02 | Production des services | Métier (activité) | À désigner (pressenti : direction des opérations) | Projets, affectations, suivi des prestations | D02, D11 | Hybride | Héritée (Internet via API) | A16, A17, A18, A20, A22, A25 | 4 | Cœur de l'activité de l'ESN ; interruption = prestations non pilotées |
| A03 | Communication et collaboration | Métier (activité) | À désigner (pressenti : direction générale / DSI) | Échanges internes et externes, documents | D03, D12 | SaaS + interne | Internet (SaaS) | A37, A38, A14, A21, A39, A40 | 3 | Mode dégradé possible, mais vecteur majeur d'hameçonnage |
| A04 | Administration du SI | Métier (processus) | À désigner (pressenti : DSI) | Gestion des comptes, serveurs, réseaux, Cloud | D04, D05, D06, D10 | Hybride | Distant (VPN), Internet (API AWS) | A14, A15, A27-A30, A35, A06, A07 | 4 | Détient les accès à privilèges sur l'ensemble du SI |

## 2. Infrastructure interne — réseau et accès

| ID | Nom | Catégorie | Propriétaire | Fonction | Données traitées | Localisation | Exposition | Dépendances | Crit. | Justification |
|---|---|---|---|---|---|---|---|---|---|---|
| A05 | Pare-feu OPNsense | Infrastructure réseau / sécurité | À désigner (pressenti : responsable réseau) | Filtrage des flux Internet ↔ interne | Règles de filtrage, journaux de flux | Interne | **Internet (direct)** | Alimentation, Cisco (A09) | 4 | Point de passage unique : indisponibilité = coupure de tous les services exposés ; modification des règles = exposition de tout le SI |
| A06 | VPN collaborateurs | Point d'accès distant | À désigner (pressenti : responsable réseau) | Accès des télétravailleurs aux ressources internes | Identifiants, flux chiffrés | Interne | **Internet (direct)** | A05, authentification (A14 ? [V]) | 3 | Indispensable au télétravail ; compte VPN volé = accès au réseau interne |
| A07 | VPN fournisseurs | Point d'accès tiers | À désigner (pressenti : responsable réseau + gestionnaire des contrats) | Accès des prestataires pour la maintenance | Identifiants des tiers, flux d'administration | Interne | **Internet (direct), partenaire** | A05, A41 | 3 | Utilisé par des tiers dont les postes ne sont pas maîtrisés ; donne accès à des ressources d'administration |
| A08 | Liaison sécurisée interne ↔ AWS | Infrastructure réseau | À désigner (pressenti : responsable réseau / Cloud) | Interconnexion de l'infrastructure interne et du VPC | Flux applicatifs et d'administration | Hybride | Internet (tunnel) | A05, A31 | 3 | Panne = rupture des applications hybrides ; abus = pont entre deux environnements |
| A09 | Routeurs / commutateurs Cisco | Infrastructure réseau | À désigner (pressenti : responsable réseau) | Interconnexion des équipements, segmentation | Configurations, VLAN | Interne (siège + 2 agences) | Non exposé directement [V] | Alimentation | 3 | Indisponibilité = coupure d'un site ; mauvaise configuration = segmentation contournée |
| A10 | Proxy Web Squid | Infrastructure réseau / sécurité | À désigner (pressenti : responsable réseau) | Filtrage de la navigation Internet | Historique de navigation | Interne | Interne (sortant vers Internet) | A05, A11 | 2 | Contournable temporairement, mais historique de navigation = donnée personnelle |
| A11 | DNS interne (Bind / CoreDNS) | Service d'infrastructure | À désigner (pressenti : équipe système) | Résolution des noms internes | Zones DNS | Interne | Non exposé directement | A41 | 3 | Panne = la plupart des services internes injoignables ; falsification = redirection de flux |
| A12 | Wi-Fi entreprise | Infrastructure réseau | À désigner (pressenti : responsable réseau) | Accès des collaborateurs au réseau interne | Identifiants Wi-Fi | Interne | Radio locale | A09, A05 | 2 | Alternative filaire possible ; accès frauduleux = présence sur le LAN |
| A13 | Wi-Fi visiteurs | Infrastructure réseau | À désigner (pressenti : responsable réseau) | Accès Internet isolé pour les invités | Aucune donnée interne (si isolé) | Interne | Radio locale + Internet | A05 | 1 | Faible **sous réserve d'une isolation vérifiée** ; sinon à réévaluer à 3 |

## 3. Gestion des identités

| ID | Nom | Catégorie | Propriétaire | Fonction | Données traitées | Localisation | Exposition | Dépendances | Crit. | Justification |
|---|---|---|---|---|---|---|---|---|---|---|
| A14 | Keycloak | Service d'identité | À désigner (pressenti : responsable IAM / DSI) | SSO, MFA, rôles, fédération | D04 (comptes, rôles, secrets MFA) | Interne (serveurs Linux) | Internet (pages d'authentification) [H] | A41, A18 ? (base Keycloak [V]), A11 | 4 | Authentifie applications, API, GitLab et accès Kubernetes : compromission ou panne = impact sur toutes les activités |
| A15 | AWS IAM | Service d'identité Cloud | À désigner (pressenti : responsable Cloud) | Permissions sur toutes les ressources AWS | D04 (identités, politiques, clés d'accès) | AWS | Internet (API AWS) | — | 4 | Une permission excessive donne accès à S3 (dont les sauvegardes), EKS, RDS, EC2 |

## 4. Applications et données

| ID | Nom | Catégorie | Propriétaire | Fonction | Données traitées | Localisation | Exposition | Dépendances | Crit. | Justification |
|---|---|---|---|---|---|---|---|---|---|---|
| A16 | Applications Web métier (gestion clients / facturation / production) | Application | À désigner (pressenti : responsables métier A01/A02 + responsable applicatif) | Interfaces utilisateurs des activités A et B | D01, D02 | Kubernetes interne / EKS [V] | **Internet** (pour certaines) [C] | A22/A25, A17, A14, A18/A19 | 4 | Supportent directement les deux activités qui génèrent le chiffre d'affaires |
| A17 | API métier et microservices | Application | À désigner (pressenti : responsable applicatif) | Exposition des fonctions métier aux applications et partenaires | D01, D02, D11 | Kubernetes interne / EKS | **Internet** (pour certaines) | A22/A25, A14, A18, A20, A35 | 4 | Accès programmatique direct aux données : une faille d'autorisation expose massivement les données |
| A18 | PostgreSQL interne | Base de données | À désigner (pressenti : responsable des données + DBA) | Stockage des données de plusieurs applications métier | D01, D02 | Interne | Indirecte (via applications) | A41, A36, A35 | 4 | Concentre les données clients et de production : C, I et D critiques |
| A19 | Amazon RDS / PostgreSQL | Base de données Cloud | À désigner (pressenti : responsable Cloud + DBA) | Bases de certaines applications | D01 / D02 (à confirmer) | AWS | Indirecte (via EKS / EC2) [V] | A31, A15 | 3 | Provisoire : dépend des bases hébergées [V] ; passe à 4 si des données clients y sont |
| A20 | Apache Kafka | Middleware | À désigner (pressenti : équipe plateforme) | Transport des événements entre applications | D11 (événements applicatifs, utilisateurs, métier) | Interne | Indirecte | A41, A11 | 3 | Panne = désynchronisation des applications ; injection d'événements = intégrité de plusieurs systèmes |
| A21 | Serveur de fichiers (Samba) | Stockage | À désigner (pressenti : DSI + responsables de services) | Documents internes partagés | D03 | Interne | Indirecte (postes, VPN) | A41, A36, A12 | 3 | Documents commerciaux et procédures ; cible classique des rançongiciels |

## 5. Kubernetes et conteneurs

| ID | Nom | Catégorie | Propriétaire | Fonction | Données traitées | Localisation | Exposition | Dépendances | Crit. | Justification |
|---|---|---|---|---|---|---|---|---|---|---|
| A22 | Cluster Kubernetes interne (nœuds, pods, namespaces, services, Ingress) | Infrastructure d'exécution | À désigner (pressenti : équipe plateforme) | Exécution des applications Web, API, microservices | Données en mémoire des applications | Interne | **Internet via Ingress** | A41, A26, A14, A18, A11 | 4 | Héberge A16/A17 : indisponibilité = arrêt des activités A et B |
| A23 | API Kubernetes, comptes et rôles RBAC | Administration / identité | À désigner (pressenti : équipe plateforme) | Pilotage du cluster, attribution des droits | Configuration du cluster, comptes de service | Interne | Indirecte (CI/CD, administrateurs, pods) | A14 (OIDC [V]), A28 | 4 | Qui contrôle l'API contrôle toutes les charges et tous les secrets du cluster |
| A24 | Secrets Kubernetes | Donnée technique | À désigner (pressenti : équipe plateforme + RSSI) | Fourniture des identifiants aux pods | D05 (mots de passe BDD, tokens) | Interne (etcd) | Indirecte (RBAC) | A23, A35 [V] | 4 | Lisibles par un compte de service trop permissif → accès aux bases et API |
| A25 | Amazon EKS | Infrastructure d'exécution Cloud | À désigner (pressenti : responsable Cloud / plateforme) | Exécution d'applications conteneurisées dans AWS | Données des applications | AWS | Internet via Load Balancer | A15, A31, A26, A19 | 3 | Provisoire : dépend des applications hébergées [V] |
| A26 | Registre d'images (Harbor / GitLab Registry / ECR) et images Docker | Chaîne logicielle | À désigner (pressenti : équipe DevOps) | Stockage des images déployées | D07 | Interne et/ou AWS [V] | Indirecte (CI/CD) | A28, A15 (ECR) | 3 | Image remplacée = code malveillant déployé ; reste conditionné à un déploiement (A28) |

## 6. Développement et déploiement

| ID | Nom | Catégorie | Propriétaire | Fonction | Données traitées | Localisation | Exposition | Dépendances | Crit. | Justification |
|---|---|---|---|---|---|---|---|---|---|---|
| A27 | GitLab — dépôts, comptes, tokens | Outil de développement | À désigner (pressenti : responsable des développements) | Gestion du code source et de l'IaC | D06 (code, IaC, configurations) | Interne | À confirmer (interne / VPN / Internet) [V] | A41, A14, A36 | 4 | Code source et IaC de toute l'entreprise ; modification = altération de la production |
| A28 | GitLab CI/CD (pipelines, runners, variables) | Chaîne de déploiement | À désigner (pressenti : équipe DevOps) | Build, tests, analyse, publication, déploiement | D05 (variables CI), D07 | Interne | Indirecte (via GitLab) | A27, A26, A23, A15, A48 | 4 | Possède les droits de déployer en production Kubernetes et AWS |
| A29 | Terraform (code IaC et fichiers d'état) | IaC | À désigner (pressenti : responsable Cloud) | Création et modification des ressources AWS / Kubernetes | D06, état de l'infrastructure (secrets possibles [V]) | GitLab + emplacement de l'état [V] | Indirecte | A27, A28, A15 | 3 | Risque porté surtout par le pipeline (A28) et les identifiants AWS ; l'état peut contenir des secrets [V] |
| A30 | Ansible | Outil d'administration | À désigner (pressenti : équipe système) | Configuration et mises à jour des serveurs Linux | Inventaires, playbooks, identifiants SSH | Interne | Non exposé directement | A27, A41, A35 | 4 | Accès à privilèges sur tous les serveurs Linux : compromission = contrôle du parc serveur |

## 7. Infrastructure AWS

| ID | Nom | Catégorie | Propriétaire | Fonction | Données traitées | Localisation | Exposition | Dépendances | Crit. | Justification |
|---|---|---|---|---|---|---|---|---|---|---|
| A31 | Amazon VPC (sous-réseaux, Security Groups) | Infrastructure réseau Cloud | À désigner (pressenti : responsable Cloud) | Isolement réseau des ressources AWS | Règles réseau | AWS | Configuration via API AWS | A15 | 3 | Une règle trop ouverte expose directement RDS / EC2 / EKS |
| A32 | Amazon EC2 | Infrastructure Cloud | À désigner (pressenti : responsable Cloud) | Services non conteneurisés | À identifier [V] | AWS | À confirmer [V] | A31, A15 | 2 | Provisoire : services hébergés non identifiés — à réévaluer |
| A33 | Amazon S3 | Stockage Cloud | À désigner (pressenti : responsable Cloud + responsable sauvegardes) | Fichiers, **sauvegardes**, artefacts | D08, D09, D07 | AWS | Internet (API S3 ; publique si politique permissive [V]) | A15 | 4 | Contient des sauvegardes [C] : altération/suppression = perte de la capacité de reprise |
| A34 | AWS Load Balancer | Point d'entrée Cloud | À désigner (pressenti : responsable Cloud) | Distribution des connexions vers les services | Flux applicatifs (TLS terminé ? [V]) | AWS | **Internet (direct)** | A31, A25 / A32 | 3 | Indisponibilité = services AWS injoignables ; pas de stockage de données |

## 8. Secrets et sauvegardes

| ID | Nom | Catégorie | Propriétaire | Fonction | Données traitées | Localisation | Exposition | Dépendances | Crit. | Justification |
|---|---|---|---|---|---|---|---|---|---|---|
| A35 | HashiCorp Vault | Sécurité (gestion des secrets) | À désigner (pressenti : RSSI + équipe plateforme) | Protection des mots de passe, tokens, clés | D05 | Interne [V] | Indirecte (applications, CI/CD) | A41, A14 ? [V], A36 | 4 | Donne accès à toutes les ressources dont il garde les secrets ; indisponibilité = applications incapables de démarrer |
| A36 | Serveur de sauvegarde (Restic / Veeam) | Continuité | À désigner (pressenti : responsable infrastructure) | Sauvegarde des données et systèmes critiques | D09 (copie de D01-D06) | Interne (+ S3 [C]) | Non exposé directement | A41, A33, A08 | 4 | Dernier recours face à un rançongiciel ; contient une copie de toutes les données sensibles |

## 9. Services collaboratifs

| ID | Nom | Catégorie | Propriétaire | Fonction | Données traitées | Localisation | Exposition | Dépendances | Crit. | Justification |
|---|---|---|---|---|---|---|---|---|---|---|
| A37 | Microsoft 365 (messagerie, SharePoint) | Service SaaS | À désigner (pressenti : DSI — administrateur M365) | Messagerie, stockage et partage documentaire | D03, D12 | SaaS | **Internet (direct)** | Fédération Keycloak (A14) [V] | 3 | Messagerie = vecteur n°1 d'hameçonnage ; données clients dans les échanges |
| A38 | Discord | Service SaaS (messagerie) | À désigner (pressenti : DSI / direction) | Communication entre collaborateurs | D12 (contenu à identifier) | SaaS | **Internet (direct)** | — | 2 | Provisoire : statut et données échangées à confirmer [V] ; hors du périmètre d'administration probable |

## 10. Postes et serveurs

| ID | Nom | Catégorie | Propriétaire | Fonction | Données traitées | Localisation | Exposition | Dépendances | Crit. | Justification |
|---|---|---|---|---|---|---|---|---|---|---|
| A39 | Postes utilisateurs fixes (~150, Windows / Linux) | Poste de travail | À désigner (pressenti : responsable du support / poste de travail) | Outil de travail quotidien | Copies locales de D01-D03, sessions | Interne (3 sites) | Indirecte (messagerie, Web) | A12, A10, A14, A42 | 3 | Logiciels obsolètes constatés [C] ; principal point d'entrée d'une compromission |
| A40 | Ordinateurs portables nomades | Poste de travail | À désigner (pressenti : responsable du support / poste de travail) | Travail en mobilité et chez les clients | Copies locales, identifiants VPN | Nomade | **Internet (réseaux tiers)** | A06, A14, A42 | 3 | Risque de perte / vol et de connexion depuis des réseaux non maîtrisés |
| A41 | Serveurs Linux (Ubuntu Server / Debian) | Infrastructure serveur | À désigner (pressenti : équipe système) | Socle d'hébergement des services internes | Données des services hébergés | Interne | Indirecte (SSH via VPN / LAN) | A09, A11, A30 | 3 | Hérite de la criticité des services hébergés (A14, A18, A20, A27 → protégés au niveau de leur propre actif) ; vulnérabilités détectées [C] |

## 11. Sécurité et supervision

| ID | Nom | Catégorie | Propriétaire | Fonction | Données traitées | Localisation | Exposition | Dépendances | Crit. | Justification |
|---|---|---|---|---|---|---|---|---|---|---|
| A42 | Wazuh (manager + agents) | Sécurité (SIEM / détection) | À désigner (pressenti : RSSI / équipe SOC) | Collecte et analyse des événements de sécurité | D10 | Interne | Non exposé directement | A43, A41 | 3 | Sans lui, les incidents ne sont pas détectés ; un attaquant cherchera à le désactiver |
| A43 | OpenSearch + OpenSearch Dashboards | Sécurité (stockage / visualisation) | À désigner (pressenti : RSSI / équipe SOC) | Stockage, recherche, visualisation des événements | D10 | Interne | Non exposé directement [V] | A41 | 3 | Preuves pour l'investigation ; contient des données personnelles (IP, comptes) |
| A44 | Prometheus / Loki / Grafana | Observabilité | À désigner (pressenti : équipe plateforme) | Métriques, logs applicatifs, tableaux de bord | Métriques, logs applicatifs | Interne | Non exposé directement [V] | A41, A22 | 2 | Utile au diagnostic ; les logs applicatifs peuvent contenir des données sensibles |
| A45 | Suricata / Snort, Zeek | Sécurité (détection réseau) | À désigner (pressenti : RSSI / responsable réseau) | Détection et analyse des communications réseau | Métadonnées de flux, alertes | Interne | Non exposé directement | A05 / A09 (copie de trafic [V]) | 2 | Perte de visibilité réseau, sans arrêt de l'activité |
| A46 | Greenbone / OpenVAS | Sécurité (vulnérabilités) | À désigner (pressenti : RSSI) | Recherche des vulnérabilités | Inventaire des vulnérabilités | Interne | Non exposé directement | Accès réseau étendu | 2 | Indisponibilité supportable ; rapports à protéger (carte des faiblesses) |
| A47 | Velociraptor | Sécurité (investigation) | À désigner (pressenti : RSSI / équipe SOC) | Investigation et collecte sur les systèmes | Artefacts forensiques | Interne | Non exposé directement | A39-A41 (agents) | 4 | Le serveur peut exécuter des collectes / commandes sur tous les postes équipés : compromission = contrôle du parc |
| A48 | SonarQube, Trivy, MISP | Sécurité (code, images, threat intelligence) | À désigner (pressenti : équipe DevOps / RSSI) | Analyse du code, des images, gestion des IoC | Résultats d'analyse, IoC | Interne | Non exposé directement | A28 | 2 | Contournement = vulnérabilités non détectées, mais pas d'impact direct sur la production |

---

## Synthèse du registre

| Criticité | Nombre | Actifs |
|---|---|---|
| 4 — Critique | 19 | A01, A02, A04, A05, A14, A15, A16, A17, A18, A22, A23, A24, A27, A28, A30, A33, A35, A36, A47 |
| 3 — Important | 20 | A03, A06, A07, A08, A09, A11, A19, A20, A21, A25, A26, A29, A31, A34, A37, A39, A40, A41, A42, A43 |
| 2 — Modéré | 8 | A10, A12, A32, A38, A44, A45, A46, A48 |
| 1 — Faible | 1 | A13 |

**Points à clarifier en priorité auprès de la DSI** : désignation des propriétaires (100 % des actifs), répartition des applications entre l'interne et AWS (A16, A17, A19, A25, A32), registre d'images réellement utilisé (A26), emplacement de tous les secrets (A24, A28, A29, A35), exposition de GitLab (A27).
