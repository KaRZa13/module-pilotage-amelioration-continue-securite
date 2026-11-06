# Livrable 7 — Besoins de sécurité et première matrice de pilotage

> Parties 9 et 11 de l'énoncé.
> À ce stade, on décrit **ce qu'il serait utile de savoir et pourquoi**, sans concevoir de règle SIEM, choisir de seuil ou construire de tableau de bord (ce sera l'objet des TP suivants).

Raisonnement : **actif → enjeu métier → conséquence d'un incident → besoin de sécurité → besoin de surveillance.**

## 1. Matrice des besoins de sécurité des cinq actifs critiques (9.2)

| Actif | Besoin de confidentialité | Besoin d'intégrité | Besoin de disponibilité | Événements à surveiller | Justification |
|---|---|---|---|---|---|
| **Keycloak** (A14) | **Critique** — empreintes de mots de passe, secrets MFA, configuration des clients confidentiels | **Critique** — rôles et groupes déterminent l'accès à toutes les applications | **Critique** — point unique d'authentification | Échecs d'authentification (volume, compte, IP source) ; connexions des comptes administrateurs ; création / suppression de comptes ; modification de rôles, groupes, clients ; désactivation ou réinitialisation du MFA ; changements de configuration du realm et des fédérations | Supporte les 4 activités ; déjà ciblé (35 échecs sur `admin`) ; une compromission ouvre toutes les applications fédérées |
| **PostgreSQL** (A18) | **Critique** — données clients (personnelles) et de production | **Critique** — facturation, contrats, affectations | **Critique** — arrêt des activités A et B | Échecs d'authentification ; connexions de comptes à privilèges ou depuis une source inhabituelle ; modifications de rôles et de droits ; opérations destructrices (suppression de tables, de bases) ; volumes de lecture anormaux (export) ; état des sauvegardes de la base | Données à plus forte valeur métier ; partagée par plusieurs applications ; **aucune collecte décrite** actuellement |
| **Kubernetes** (A22-A24) | **Élevé** — secrets montés dans les pods, données en transit | **Critique** — ce qui tourne en production doit être ce qui a été validé | **Critique** — exécute les applications métier | Modifications RBAC (rôles, liaisons) ; lecture de Secrets ; `exec` / attachement dans un pod ; création de pods privilégiés ou montant l'hôte ; déploiement d'une image hors registre autorisé ; création de namespaces / comptes de service ; trafic sortant inhabituel des pods (Suricata) | Exécute les applications A16/A17 ; une compromission de pod peut s'étendre selon les droits RBAC |
| **GitLab + CI/CD** (A27-A28) | **Élevé** — code source, secrets dans les variables CI | **Critique** — le code et les pipelines sont déployés automatiquement en production | **Important** — nécessaire pour corriger, pas pour produire | Connexions (notamment admin) ; création de tokens ; modifications de permissions et de protections de branches ; modification des fichiers de pipeline ; exécution des pipelines de déploiement en production (qui, quel commit, approuvé ?) ; résultats Trivy / SonarQube ignorés | Pont à privilèges vers Kubernetes et AWS ; risque d'attaque par la chaîne d'approvisionnement |
| **AWS IAM** (A15) | **Critique** — clés d'accès donnant accès à S3, RDS, EKS | **Critique** — une politique modifiée ouvre l'ensemble du Cloud | **Important** | Création / modification d'utilisateurs, rôles, politiques ; création de clés d'accès ; utilisation du compte racine ; connexions console sans MFA ; modification des politiques de buckets S3 et des Security Groups ; suppression d'objets dans les buckets de sauvegarde | Contrôle l'accès aux sauvegardes S3 et à toute l'infrastructure Cloud ; **aucune source d'audit AWS** n'est décrite dans la collecte |

## 2. Première matrice de pilotage (partie 11)

Matrice adaptée aux actifs retenus : les cinq actifs prioritaires + trois actifs « à suivre » (VPN fournisseurs, sauvegardes, Vault) identifiés lors de l'analyse d'exposition et de dépendances.

| Actif | Activité métier | Criticité | Exposition | Risque pressenti | Besoin de surveillance | Données à rechercher |
|---|---|---|---|---|---|---|
| **Keycloak** (A14) | Toutes (A01-A04) — accès aux applications et administration | **4** | Internet (authentification) [H] ; console d'administration [V] | Compromission d'un compte (force brute, *credential stuffing*, hameçonnage) ou modification illégitime de droits | Connexions et échecs, connexions administrateurs, changements de rôles et de configuration | Événements d'authentification et d'administration Keycloak, IP source (OPNsense / Ingress), réputation IP (MISP) |
| **PostgreSQL** (A18) | A — Gestion des clients ; B — Production | **4** | Indirecte (via applications / API) ; accès direct depuis le LAN / VPN [V] | Accès non autorisé aux données clients, falsification, destruction ou indisponibilité | Accès des comptes à privilèges, accès inhabituels, opérations destructrices, état des sauvegardes | Journaux de connexion et d'audit de la base, contexte des accès (source, compte, application), journaux de sauvegarde |
| **Kubernetes** (A22-A24) | A et B — exécution des applications | **4** | Internet via Ingress ; API Kubernetes [V] | Compromission d'un pod et élévation via RBAC ; déploiement non autorisé | Changements sensibles (RBAC, Secrets, pods privilégiés), images déployées, comportement réseau des pods | Journaux d'audit de l'API Kubernetes, événements du cluster, alertes Suricata / Zeek, résultats Trivy |
| **GitLab + CI/CD** (A27-A28) | D — Administration ; B — Production (déploiement) | **4** | À confirmer (VPN / Internet) | Modification du code ou du pipeline, vol de token, déploiement non autorisé | Permissions, tokens, modifications de pipeline, déploiements en production | Journaux d'audit GitLab, historique des pipelines, approbations, résultats SonarQube / Trivy |
| **AWS IAM** (A15) | D — Administration des ressources Cloud | **4** | Internet (API AWS) | Attribution de permissions excessives, vol de clés, suppression des sauvegardes S3 | Changements de politiques et d'identités, usage du compte racine, création de clés | Événements d'audit AWS (source à mettre en place [V]), configuration IAM, politiques S3 |
| VPN fournisseurs (A07) | D — Maintenance par les prestataires | 3 | **Internet, prestataires** | Usurpation d'un compte prestataire, rebond vers les serveurs | Connexions (qui, quand, d'où, combien de temps), ressources atteintes | Journaux VPN, flux OPNsense depuis le pool VPN, journaux SSH des serveurs ciblés |
| Sauvegardes (A36, A33) | Toutes — continuité | **4** | Non exposé directement (A36) ; API S3 (A33) | Suppression / chiffrement des sauvegardes avant un rançongiciel ; restauration impossible | Succès / échec des sauvegardes, suppressions, accès au serveur, tests de restauration | Journaux Restic / Veeam, événements S3 (suppressions), rapports de test de restauration |
| Vault (A35) | Toutes — accès des applications | **4** | Indirecte | Lecture illégitime de secrets, modification des politiques | Accès aux secrets sensibles, création de tokens, changements de politiques | Journal d'audit Vault (activation à vérifier [V]) |

## 3. Les quatre questions de la matrice

### Keycloak
- **Pourquoi important ?** Point unique d'authentification de toutes les applications ; sa compromission vaut celle de tous les comptes.
- **Incident à fort impact ?** Prise de contrôle du compte `admin` du realm → création d'un compte administrateur caché sur les applications métier.
- **Informations utiles ?** Origine et volume des échecs, succès qui suivent une série d'échecs, modifications de rôles juste après une connexion admin.
- **Utilité pour la décision ?** Permet de décider s'il faut bloquer une IP, verrouiller un compte, imposer le MFA ou restreindre l'accès à la console d'administration.

### PostgreSQL
- **Pourquoi important ?** Contient les données clients et de production de plusieurs applications.
- **Incident à fort impact ?** Export massif des données clients par un compte applicatif volé, ou suppression des tables.
- **Informations utiles ?** Quel compte, depuis quelle application ou quelle machine, quel volume, à quelle heure.
- **Utilité pour la décision ?** Distinguer un usage applicatif normal d'un accès illégitime, décider d'une révocation de compte, mesurer l'ampleur d'une fuite (notification RGPD).

### Kubernetes
- **Pourquoi important ?** Exécute les applications des activités A et B.
- **Incident à fort impact ?** Pod exposé compromis qui lit les Secrets et accède à PostgreSQL.
- **Informations utiles ?** Actions sur l'API (`exec`, lecture de secrets, nouveaux rôles), flux sortants du pod.
- **Utilité pour la décision ?** Évaluer le rayon d'impact (droits du compte de service), décider d'isoler un pod / namespace, prioriser la réduction des droits RBAC.

### GitLab + CI/CD
- **Pourquoi important ?** Tout ce qui y est fusionné finit en production (Kubernetes, AWS).
- **Incident à fort impact ?** Token volé utilisé pour modifier un pipeline et déployer une image piégée.
- **Informations utiles ?** Qui a modifié quoi, avec quel token, quel pipeline a déployé quelle image, avec ou sans approbation.
- **Utilité pour la décision ?** Valider qu'un déploiement est légitime (événement 4), révoquer des tokens, imposer des approbations.

### AWS IAM
- **Pourquoi important ?** Contrôle l'accès à toutes les ressources Cloud, y compris les sauvegardes.
- **Incident à fort impact ?** Clé d'accès d'administration divulguée → suppression des sauvegardes S3 puis attaque de la production.
- **Informations utiles ?** Qui a créé / modifié quelle permission, quelles clés sont actives et utilisées depuis où.
- **Utilité pour la décision ?** Révoquer des clés, réduire des permissions, justifier la mise en place de la collecte d'audit AWS (aujourd'hui absente de la description).

## 4. Angles morts identifiés pour le TP suivant

La collecte décrite par la DSI (Keycloak, Linux, OPNsense, GitLab, Kubernetes, Suricata → Wazuh) **ne couvre pas** : PostgreSQL, AWS (IAM, S3), Vault, Kafka, VPN fournisseurs, serveur de sauvegarde, Microsoft 365. Or trois de ces sources concernent des actifs de criticité 4. C'est un premier écart entre les besoins de surveillance et le dispositif existant, à confirmer avec la DSI.
