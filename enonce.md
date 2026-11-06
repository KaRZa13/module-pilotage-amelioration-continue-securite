Identifier le SI, les actifs critiques et les besoins de sécurité
Introduction
Dans ce module, nous allons apprendre à piloter la sécurité d'un système d'information.
Le pilotage de la sécurité ne consiste pas simplement à installer :
un pare-feu ;
un antivirus ;
un EDR ;
un SIEM ;
un scanner de vulnérabilités ;
un outil de supervision.
Ces technologies ne sont que des moyens.
Le véritable objectif est de répondre à des questions de pilotage :
Que devons-nous protéger ?
Pourquoi devons-nous le protéger ?
Quels sont les risques prioritaires ?
Que devons-nous surveiller ?
Quelles informations devons-nous collecter ?
Comment allons-nous mesurer l'efficacité des mesures de sécurité ?


Le module va donc suivre une logique progressive :
Besoins métier
      │
     ▼
Actifs critiques
      │
     ▼
Risques
      │
     ▼
Besoins de surveillance
      │
     ▼
Données / événements
      │
     ▼
Indicateurs
      │
     ▼
Actions correctives
      │
     ▼
Amélioration continue

Dans ce premier TP, nous allons construire le socle du dispositif de pilotage : la connaissance du système d'information, de ses actifs et de ses enjeux métier.
Les travaux réalisés seront réutilisés dans tous les prochains TP.

Le fil rouge du module
Pendant l'ensemble des TP, nous allons travailler sur la même entreprise fictive.
Entreprise — NOVA SERVICES
NOVA SERVICES est une entreprise de services numériques de taille moyenne.
Elle compte :
180 collaborateurs ;
1 siège social ;
2 agences ;
des collaborateurs en télétravail ;
plusieurs applications métiers ;
une infrastructure hybride ;
des services exposés sur Internet ;
plusieurs prestataires externes.
L'entreprise souhaite améliorer le pilotage de sa sécurité après plusieurs événements :
plusieurs tentatives de connexion suspectes ;
des postes utilisateurs présentant des logiciels obsolètes ;
des vulnérabilités détectées sur des serveurs ;
des alertes de sécurité difficiles à qualifier ;
une absence de tableau de bord de sécurité ;
une difficulté à déterminer quelles actions doivent être traitées en priorité.
La direction demande donc au service informatique de construire un dispositif permettant de :
mesurer l'exposition du SI, surveiller les risques importants et piloter les actions de sécurité dans le temps.
Vous êtes intégrés à l'équipe chargée de cette mission.

Objectif général du TP
À la fin de ce TP, vous devez être capables de :
identifier les principaux composants d'un SI ;
identifier les actifs importants pour l'entreprise ;
distinguer actif technique et actif métier ;
identifier les données sensibles ;
identifier les dépendances entre actifs ;
repérer les éléments exposés ;
identifier les premiers besoins de sécurité ;
construire une première cartographie du SI ;
commencer à structurer un registre des actifs ;
préparer l'analyse des risques des prochains TP.
Prérequis
connaissances générales en sécurité des systèmes d'information ;
notions de menace, vulnérabilité, exposition et risque ;
lecture d'un schéma d'architecture ;
connaissances générales sur les réseaux ;
notions de serveurs, postes de travail, bases de données et applications ;
connaissance générale des principes de sécurité informatique.

Menace : ce qui peut causer un dommage.
 Exemple : un pirate informatique.
Vulnérabilité : une faiblesse qui peut être exploitée.
 Exemple : un logiciel non mis à jour.
Exposition : le niveau auquel un système est accessible ou visible par une menace.
 Exemple : un serveur directement accessible depuis Internet.
Risque : la possibilité qu'une menace exploite une vulnérabilité et provoque un impact.
 Exemple : un pirate exploite un serveur vulnérable et vole des données.


Partie 1 — Comprendre le problème
1.1 Une entreprise ne protège pas uniquement des serveurs
Considérons l'infrastructure suivante :
Internet
   │
   ▼
Pare-feu
   │
   ├───────────────► Application Web
   │                       │
   │                      ▼
   │                 Base de données
   │
   └───────────────► Messagerie
                           │
                          ▼
                    Postes utilisateurs

On pourrait être tenté de dire :
« Il faut sécuriser le pare-feu. »
Mais cette réponse est insuffisante.
Pourquoi ?
Parce que le pare-feu n'est pas nécessairement l'actif le plus important pour l'entreprise.
Si la base de données contient les informations clients, sa compromission peut avoir des conséquences beaucoup plus importantes.
Le pilotage commence donc par :
SI
│
├── Quelles applications ?
├── Quelles données ?
├── Quels utilisateurs ?
├── Quels serveurs ?
├── Quels équipements ?
├── Quels services ?
└── Quelles dépendances ?

Partie 2 — Découvrir le système d'information de NOVA SERVICES
La DSI de NOVA SERVICES vous fournit les informations suivantes concernant son système d'information.
NOVA SERVICES possède une infrastructure hybride, composée d'une infrastructure interne et d'une infrastructure Cloud hébergée sur AWS.
L'entreprise utilise principalement des technologies Linux, open source, conteneurisées et Cloud-native.
L'objectif de cette partie est de permettre aux étudiants de :
comprendre l'architecture générale du SI ;
identifier les principaux actifs ;
identifier les systèmes exposés ;
repérer les données sensibles ;
identifier les principales sources de journaux et d'événements ;
comprendre quels outils pourront être utilisés pour surveiller le SI.
Attention : à ce stade, il ne s'agit pas encore de définir les règles de détection ou les indicateurs. Il faut d'abord comprendre ce qui compose le SI et ce qu'il faut protéger.
2.1 Une infrastructure hybride
Le système d'information de NOVA SERVICES est réparti entre :
une infrastructure interne située dans les locaux de l'entreprise ;
une infrastructure AWS ;
un cluster Kubernetes ;
plusieurs applications accessibles depuis Internet ;
des postes utilisateurs et ordinateurs portables ;
des services SaaS ;
des accès distants pour les collaborateurs et certains fournisseurs.
La DSI cherche progressivement à standardiser son infrastructure autour de technologies Linux, Docker, Kubernetes et Cloud.

2.2 Infrastructure interne
La partie interne du SI héberge les services nécessaires au fonctionnement quotidien de l'entreprise.
Élément
Description
Technologie utilisée
Pare-feu
Contrôle les connexions entre Internet et le réseau interne. Il autorise ou bloque les flux selon des règles de sécurité.
OPNsense
VPN collaborateurs
Permet aux collaborateurs en télétravail d'accéder aux ressources internes autorisées.
WireGuard / OpenVPN
VPN fournisseurs
Permet à certains prestataires d'accéder à des ressources internes pour effectuer des opérations de maintenance.
WireGuard / OpenVPN
Serveurs Linux
Hébergent les applications et services internes de NOVA SERVICES.
Ubuntu Server / Debian
Serveur de fichiers
Stocke les documents internes et certaines données partagées entre les collaborateurs.
Linux / Samba
Serveur de sauvegarde
Réalise les sauvegardes des données et systèmes critiques.
Restic / Veeam
Serveur PostgreSQL
Stocke les données de plusieurs applications métier.
PostgreSQL
Serveur Kafka
Transporte les événements et messages entre différentes applications.
Apache Kafka
DNS interne
Permet la résolution des noms des ressources internes.
Bind / CoreDNS
Serveur GitLab
Héberge les projets, dépôts Git et pipelines CI/CD.
GitLab
Wi-Fi entreprise
Permet aux collaborateurs d'accéder au réseau interne.
Infrastructure Wi-Fi professionnelle
Wi-Fi visiteurs
Fournit un accès Internet isolé du réseau interne.
Réseau invité
Postes utilisateurs
Environ 150 postes utilisés quotidiennement par les collaborateurs.
Windows / Linux
Ordinateurs portables
Utilisés par les collaborateurs nomades et lors des déplacements chez les clients.
Windows / Linux


2.3 Gestion des identités avec Keycloak
NOVA SERVICES utilise Keycloak comme solution centrale de gestion des identités et de l'authentification des applications.
Keycloak permet notamment :
de gérer les comptes utilisateurs ;
de gérer les rôles et groupes ;
d'assurer le Single Sign-On (SSO) ;
de gérer l'authentification multifacteur ;
de centraliser l'authentification de plusieurs applications ;
de fédérer les identités avec certains services externes.
Exemple :
                      Utilisateur
                            │
                           ▼
                      ┌──────────┐
                             Keycloak          
                      └────┬─────┘
                                   │
              ┌────────────┼────────────┐
               │                              │                               │
             ▼                             ▼                              ▼
          Application                  API                          GitLab
              │
             ▼
         Kubernetes

Les événements produits par Keycloak pourront être utilisés dans les prochains TPs.
Exemples d'événements intéressants :
tentative de connexion ;
échec d'authentification ;
connexion réussie ;
création d'un compte ;
suppression d'un compte ;
modification d'un rôle ;
connexion d'un compte administrateur ;
changement de configuration ;
tentative répétée depuis une même adresse IP.

2.4 Serveurs Linux
Les principaux serveurs internes de NOVA SERVICES utilisent Linux.
Ils hébergent notamment :
PostgreSQL ;
Logic tier, python, fastapi, flask ;
Kafka ;
Keycloak ;
GitLab ;
les applications internes ;
les outils de supervision ;
différents services techniques.
Les serveurs Linux produisent plusieurs catégories de journaux :
Serveur Linux
     │
     ├── Logs système
     ├── Logs SSH
     ├── Logs d'authentification
     ├── Logs applicatifs
     ├── Logs réseau
     ├── Logs sudo
     └── Logs des services

Ces données pourront ensuite être collectées et analysées pour identifier des comportements inhabituels.

2.5 Infrastructure Kubernetes
Une partie des applications de NOVA SERVICES est conteneurisée.
L'entreprise utilise Kubernetes pour déployer et administrer ces applications.
Le cluster Kubernetes contient notamment :
des applications Web ;
des API ;
des microservices ;
des services internes ;
des traitements automatisés.
L'architecture simplifiée est la suivante :
                               INTERNET
                                    │
                                   ▼
                       ┌─────────┐
                       │ OPNsense      │
                       └────┬────┘
                                    │
                                   ▼
                                 Ingress
                                    │
                  ┌──────▼─────────┐
                  │    Kubernetes    
                  │      Cluster      
                  └─────────┬──────┘
                          
             ┌───────────┼──────────────┐
             │                             │                                   │
             ▼                           ▼                                  ▼
         Application                API                      Services internes
             │                             │                                   │
             └───────────┼──────────────┘
                                            │
                                           ▼
                                   PostgreSQL

Tu dois considérer Kubernetes comme un actif critique, mais également prendre en compte ses différents composants :
nœuds ;
pods ;
services ;
namespaces ;
comptes ;
rôles RBAC ;
secrets ;
images Docker ;
API Kubernetes.
Une vulnérabilité dans une application ou un conteneur ne signifie pas nécessairement qu'un attaquant peut compromettre tout le cluster. L'impact dépend notamment des droits accordés au composant compromis.

2.6 Docker et registre de conteneurs
Les applications sont construites sous forme d'images Docker.
Les images sont stockées dans un registre de conteneurs.
NOVA SERVICES utilise :
Docker pour la conteneurisation ;
Harbor ou Gitlab registry ou Amazon ECR pour le stockage des images ;
Trivy pour rechercher les vulnérabilités dans les images.
Exemple :
Code source
    │
   ▼
 GitLab
    │
   ▼
GitLab CI/CD
    │
   ▼
Construction image Docker
    │
   ▼
  Trivy
    │
    ├── Image conforme
    │
    ▼
Registry
    │
    ▼
Kubernetes


2.7 Apache Kafka
NOVA SERVICES utilise Apache Kafka pour faire circuler des événements entre plusieurs applications.
Kafka peut transporter :
des événements applicatifs ;
des événements utilisateurs ;
des événements métier ;
des événements techniques ;
des données provenant de différents services.
Exemple :
Application A ──┐
Application B ──┼──► Kafka ──► Application C
Application C ──┘       │
                        │
                        ▼
                 Système d'analyse

Kafka constitue donc une source potentielle importante de données.
Les étudiants devront progressivement déterminer :
Quels événements doivent être conservés et lesquels sont réellement utiles au pilotage de la sécurité ?

2.8 Infrastructure Cloud AWS
Une partie du système d'information est hébergée dans Amazon Web Services (AWS).
Les ressources Cloud sont séparées de l'infrastructure interne mais communiquent avec elle lorsque cela est nécessaire.
Les principaux services utilisés sont :
Élément
Description
Technologie AWS
Réseau Cloud
Fournit un réseau isolé pour les ressources AWS.
Amazon VPC
Machines virtuelles
Hébergent certains services qui ne sont pas conteneurisés.
Amazon EC2
Kubernetes Cloud
Héberge certaines applications conteneurisées.
Amazon EKS
Stockage objet
Stocke des fichiers, sauvegardes et artefacts.
Amazon S3
Base de données Cloud
Héberge certaines bases de données applicatives.
Amazon RDS / PostgreSQL
Registre de conteneurs
Stocke les images Docker utilisées par les applications.
Amazon ECR
Équilibrage de charge
Distribue les connexions vers plusieurs instances ou services.
AWS Load Balancer
Gestion des identités AWS
Gère les permissions d'accès aux ressources AWS.
AWS IAM

Architecture simplifiée :
                                   AWS CLOUD
                                            │
                    ┌────────▼────────┐
                                         VPC       
                    └────────┬────────┘
                             
              ┌──────────┼──────────────┐
            
              ▼                         ▼                                  ▼
             EC2                     EKS                                S3
                                           │
                    ┌────────┼────────┐
                    │                     │                     │
                    ▼                   ▼                    ▼
                   API                Apps             Services
                                           │
                                          ▼
                                         RDS


2.9 Communication entre l'infrastructure interne et AWS
L'environnement hybride nécessite des communications sécurisées entre l'infrastructure interne et le Cloud.
      INFRASTRUCTURE INTERNE
                │
                │ VPN / liaison sécurisée
                │
               ▼
          ┌───────────┐
            AWS Cloud
          └───────────┘
             
Ces communications constituent également une surface à surveiller.
Exemples :
connexions inhabituelles ;
flux provenant d'une mauvaise zone réseau ;
accès non autorisé ;
modification des règles réseau ;
création d'une nouvelle ressource Cloud ;
modification des droits IAM.

2.10 GitLab et CI/CD
Les équipes de développement utilisent GitLab pour gérer le code source et automatiser les déploiements.
Le pipeline CI/CD peut effectuer :
récupération du code ;
compilation ;
tests ;
analyse du code ;
analyse des dépendances ;
construction de l'image Docker ;
analyse de l'image ;
publication dans le registre ;
déploiement sur Kubernetes.
Exemple :
Développeur
     │
    ▼
   GitLab
     │
    ▼
GitLab CI/CD
     │
     ├── Tests
     │
     ├── SonarQube
     │
     ├── Trivy
     │
     ├── Build Docker
     │
     └── Terraform
            │
            ▼
       AWS / Kubernetes

Les événements GitLab pourront également être intéressants pour la sécurité :
connexion ;
création d'un projet ;
modification des permissions ;
création d'un token ;
modification d'un pipeline ;
exécution d'un pipeline ;
déploiement en production.

2.11 Infrastructure as Code
NOVA SERVICES utilise Terraform pour décrire et déployer une partie de son infrastructure.
Cela permet notamment d'automatiser :
la création de ressources AWS ;
les réseaux ;
certaines machines virtuelles ;
les ressources Kubernetes ;
certaines configurations techniques.


Exemple :
         GitLab
             │
            ▼
       GitLab CI/CD
             │
            ▼
         Terraform
             │
       ┌─┴─────┐
       ▼                 ▼
      AWS      Kubernetes

Les modifications de l'infrastructure deviennent ainsi traçables dans Git.

2.12 Suite collaborative Cloud
NOVA SERVICES utilise également des services Cloud pour la collaboration.
Élément
Description
Technologie
Suite collaborative Cloud
Stockage, partage et collaboration sur les documents et messagerie professionnelle.
Microsoft 365
Messagerie
Communication professionnelle entre les collaborateurs.
Microsoft 365 /
Discord 
Stockage documentaire
Stockage et partage de documents.
Microsoft 365 / SharePoint
Identité fédérée
Permet aux utilisateurs de s'authentifier auprès de différents services.
Keycloak / fédération

2.13 Réseau
Élément
Description
Technologie
Pare-feu
Contrôle les flux entre Internet et le réseau interne.
OPNsense
Routeurs / commutateurs
Assurent la communication entre les équipements du réseau.
Équipements réseau - Cisco
Proxy Web
Contrôle et filtre certains accès Internet.
Squid
VPN
Sécurise les accès distants.
WireGuard / OpenVPN
Wi-Fi entreprise
Permet l'accès au réseau interne.
Infrastructure Wi-Fi
Wi-Fi visiteurs
Fournit un accès Internet isolé.
Réseau invité

2.14 Sécurité des postes et des serveurs
Élément
Description
Technologie
Agent de sécurité
Collecte des informations sur les postes et serveurs et détecte certains comportements suspects.
Wazuh Agent
Gestion des configurations
Automatise la configuration des serveurs Linux.
Ansible
Gestion des secrets
Protège les mots de passe, tokens et clés utilisés par les applications.
HashiCorp Vault
Gestion des mises à jour
Automatise ou facilite le déploiement des mises à jour sur les serveurs Linux.
Ansible / gestionnaire de paquets
Scanner de vulnérabilités
Recherche les vulnérabilités sur les systèmes et applications.
Greenbone / OpenVAS

2.15 Outils de sécurité et d'observabilité
NOVA SERVICES utilise principalement des solutions open source afin de construire progressivement son dispositif de supervision.
Besoin
Outil
Rôle
SIEM / détection
Wazuh
Collecte et analyse des événements de sécurité
Recherche et analyse
OpenSearch
Recherche et stockage des événements
Dashboard sécurité
OpenSearch Dashboards
Visualisation des événements de sécurité
Métriques
Prometheus
Collecte des métriques techniques
Visualisation
Grafana
Création de tableaux de bord
Logs applicatifs
Loki
Centralisation des logs
IDS réseau
Suricata / Snort
Détection d'activités réseau suspectes
Analyse réseau
Zeek
Analyse des communications réseau
Vulnérabilités
Greenbone / OpenVAS
Recherche des vulnérabilités
Threat Intelligence
MISP
Gestion des indicateurs de compromission
Investigation
Velociraptor
Investigation sur les systèmes
Sécurité des conteneurs
Trivy
Analyse des vulnérabilités des images
Sécurité du code
SonarQube
Analyse du code source
Automatisation
Ansible
Automatisation des opérations techniques

2.16 Collecte des événements
Les différentes briques du SI produisent des événements.
                 SOURCES
                     │
       ┌─────────────┼──────────────┐
       │                                  │                                    │
      ▼                                 ▼                                   ▼
    Keycloak                      Linux                         OPNsense
       │                                  │                                    │
       ▼                                ▼                                   ▼
    GitLab                      Kubernetes                   Suricata
       │                                  │                                    │
       └─────────────┼──────────────┘
                     │
                     ▼
                  Wazuh
                     │
                     ▼
                OpenSearch
                     │
                     ▼
                  Grafana

D'autres flux peuvent être utilisés pour l'observabilité :
Applications
     │
     ▼
    Loki
     │
     ▼
   Grafana


Serveurs / Kubernetes
     │
     ▼
 Prometheus
     │
     ▼
   Grafana



2.17 Première analyse du SI
À partir des informations fournies par la DSI, vous devez maintenant commencer à analyser le système d'information.
Exercice 1 — Identifier les actifs
Identifiez au minimum 10 actifs du SI de NOVA SERVICES.
Pour chaque actif, indiquez :
Actif
Fonction
Criticité estimée
Exemple : Keycloak
Gestion des identités
Élevée




















Exercice 2 — Identifier les actifs exposés
Quels éléments peuvent être directement ou indirectement accessibles depuis Internet ?
Classez-les en trois catégories :
Exposition directe
Exposition indirecte
Non exposé directement

Exercice 3 — Identifier les données sensibles
Identifiez les systèmes susceptibles de contenir ou de permettre l'accès à des données sensibles.
Exemples :
données clients ;
comptes utilisateurs ;
identifiants ;
documents internes ;
données applicatives ;
secrets techniques ;
données stockées dans AWS.

Exercice 4 — Identifier les sources de données
Pour chacun des éléments suivants, indiquez au moins deux types d'événements intéressants pour la sécurité :
OPNsense ;
Keycloak ;
Linux ;
Kubernetes ;
Kafka ;
GitLab ;
AWS ;
PostgreSQL ;
Suricata.

Exercice 5 — Identifier les outils
Associez chaque besoin à l'outil le plus pertinent.
Besoin
Outil
Rechercher des vulnérabilités


Détecter des activités réseau suspectes


Centraliser les événements de sécurité


Collecter les métriques


Visualiser les indicateurs


Analyser le code


Analyser les images Docker


Faire de l'investigation sur un poste


Gérer les indicateurs de compromission


Centraliser les logs applicatifs








2.19 Mise en situation
La DSI vous informe de plusieurs événements récents :
Événement 1
Le compte admin de Keycloak a subi 35 échecs d'authentification en 5 minutes.
Événement 2
Un serveur Linux reçoit plusieurs connexions SSH depuis une adresse IP inhabituelle.
Événement 3
Greenbone détecte une vulnérabilité critique sur une application exposée sur Internet.
Événement 4
Un pipeline GitLab déploie une nouvelle image Docker en production.
Événement 5
Suricata détecte un comportement réseau suspect provenant d'un serveur Kubernetes.
Questions
Pour chaque événement :
Quel actif est concerné ?
Quelle pourrait être la menace ?
Quelle pourrait être la vulnérabilité ?
Quel est le niveau d'exposition ?
Quel pourrait être le risque ?
Quelle source de données permettrait de confirmer l'événement ?
Quel outil pourrait être utilisé pour l'analyser ?

2.20 Point important pour la suite du projet
À ce stade, vous devez retenir une idée essentielle :
On ne commence pas par choisir un outil de sécurité. On commence par comprendre le SI et les risques auxquels il est exposé.
La démarche suivie dans le projet sera :
             ACTIVITÉS MÉTIER
                     │
                     ▼
                  ACTIFS
                     │
                     ▼
                  MENACES
                     │
                     ▼
              VULNÉRABILITÉS
                     │
                     ▼
                 EXPOSITION
                     │
                     ▼
                   RISQUE
                     │
                     ▼
        BESOINS DE SURVEILLANCE
                     │
                     ▼
             SOURCES DE DONNÉES
                     │
                     ▼
          INDICATEURS / ALERTES
                     │
                     ▼
              ANALYSE / TRIAGE
                     │
                     ▼
             ACTIONS CORRECTIVES
                     │
                     ▼
        AMÉLIORATION CONTINUE

Cette chaîne constituera le fil rouge de l'ensemble des TPs.


Partie 3 — Identifier les activités métier de NOVA SERVICES
3.1 Pourquoi partir des activités métier ?
La partie 2 nous a permis de découvrir l'architecture technique de NOVA SERVICES : infrastructure interne, AWS, Kubernetes, Keycloak, GitLab, PostgreSQL, Kafka et outils de sécurité.
Cependant, connaître les technologies utilisées ne suffit pas pour déterminer les priorités de sécurité.
Il faut maintenant comprendre :
quelles activités permettent à l'entreprise de fonctionner ;
quelles données sont indispensables à ces activités ;
quels services informatiques les supportent ;
quelles seraient les conséquences d'une compromission ou d'une indisponibilité.
Le raisonnement à suivre est le suivant :
Activité métier
      │
     ▼
Données et services nécessaires
      │
     ▼
Applications et composants techniques
      │
     ▼
Conséquences d'un incident
      │
     ▼
Priorités de sécurité

Principe de pilotage : la priorité de sécurité dépend de la valeur de l'actif pour l'entreprise, de son exposition et des conséquences possibles d'un incident.



3.2 Les principales activités de l'entreprise

NOVA SERVICES réalise quatre activités principales.
Activité
Description
Exemples de ressources informatiques
A — Gestion des clients
Gestion des clients, contrats, prestations et facturation
Applications métier, PostgreSQL, stockage documentaire
B — Production des services
Gestion des projets, affectation des collaborateurs et suivi des prestations
Applications Web, API, Kubernetes, PostgreSQL, Kafka
C — Communication et collaboration
Échanges professionnels, messagerie et partage de documents
Microsoft 365, SharePoint, services d'identité
D — Administration du SI
Gestion des comptes, serveurs, réseaux, applications et infrastructures Cloud
Keycloak, accès VPN, AWS IAM, GitLab, outils d'administration

Ces activités s'appuient sur plusieurs composants techniques. Une même activité peut dépendre de plusieurs systèmes, et un même composant peut servir plusieurs activités.
3.3 Travail demandé — Relier les activités aux composants techniques
À partir de la partie 2, complétez le tableau suivant.
Activité métier
Données manipulées
Applications ou services
Composants techniques dépendants
Gestion des clients
À identifier
À identifier
À identifier
Production des services
À identifier
À identifier
À identifier
Communication
À identifier
À identifier
À identifier
Administration du SI
À identifier
À identifier
À identifier

Pour chaque ligne, répondez aux questions suivantes :
Quelle activité serait perturbée si l'application principale devenait indisponible ?
Quelles données pourraient être consultées ou modifiées sans autorisation ?
Quels autres composants seraient affectés par la compromission de cette ressource ?
Qui, dans l'entreprise, devrait contribuer à déterminer son niveau de criticité ?
Attention : ne déduisez pas automatiquement la criticité d'un actif de la technologie utilisée. Kubernetes, PostgreSQL ou Keycloak peuvent être importants, mais leur criticité doit être justifiée par les services et les activités qu'ils supportent.

Partie 4 — Identifier les données sensibles et importantes
4.1 Pourquoi les données sont-elles un élément central du pilotage ?
Les données constituent souvent une part essentielle de la valeur d'un système d'information.
Une donnée peut être importante parce qu'elle :
contient des informations confidentielles ;
permet de réaliser une activité métier ;
doit rester exacte et complète ;
doit être disponible dans un délai donné ;
permet d'administrer ou de sécuriser d'autres systèmes.
Exemples dans NOVA SERVICES :
Catégorie de données
Exemple
Conséquence possible d'un incident
Données clients
Coordonnées, contrats, historique des prestations
Atteinte à la confidentialité, perte de confiance
Données de production
Projets, affectations, suivi des prestations
Perturbation de l'activité
Documents internes
Documents partagés, procédures, documents commerciaux
Divulgation ou altération
Identités et habilitations
Comptes, rôles, groupes et droits d'accès
Accès non autorisé aux applications
Secrets techniques
Tokens, clés et secrets utilisés par les applications
Compromission de services
Données Cloud
Objets stockés dans S3, configurations et données applicatives
Exposition ou altération de ressources
Données de sécurité
Journaux d'authentification, événements système et alertes
Difficulté à détecter ou analyser un incident

4.2 Travail demandé — Recenser les données
Identifiez au minimum six catégories de données utilisées ou produites par NOVA SERVICES.
Pour chaque catégorie, indiquez :
son utilisation ;
son propriétaire métier ou technique ;
le système qui la stocke ou la traite ;
les besoins de confidentialité, d'intégrité et de disponibilité ;
les conséquences possibles d'une perte, d'une divulgation ou d'une modification ;
les besoins de surveillance associés.
Utilisez le modèle suivant :
Donnée
Utilisation
Système concerné
Responsable
Confidentialité
Intégrité
Disponibilité
Données clients
Gestion des clients
Application métier / PostgreSQL
À identifier
À évaluer
À évaluer
À évaluer
Comptes et rôles
Authentification et autorisation
Keycloak
À identifier
À évaluer
À évaluer
À évaluer
Secrets techniques
Accès des applications aux ressources
HashiCorp Vault ou autres emplacements à identifier
À identifier
À évaluer
À évaluer
À évaluer
Journaux de sécurité
Analyse des événements
Wazuh / OpenSearch selon l'architecture fournie
À identifier
À évaluer
À évaluer
À évaluer

Les technologies et emplacements doivent être vérifiés à partir de la description du SI. Si le contexte ne permet pas de déterminer précisément où une donnée est stockée, indiquez « à confirmer auprès de la DSI ».
4.3 Question de réflexion
Une base PostgreSQL et un serveur de sauvegarde sont-ils importants pour les mêmes raisons ?
Expliquez pourquoi la sécurité doit tenir compte :
de la confidentialité des données ;
de leur intégrité ;
de leur disponibilité ;
de la capacité à restaurer les services après un incident.


Partie 5 — Identifier et catégoriser les actifs ( optionnel ) 
5.1 Définition d'un actif
Un actif est un élément ayant de la valeur pour l'organisation et devant être pris en compte dans la démarche de sécurité.
Il peut s'agir :
d'une activité métier ;
d'une donnée ;
d'une application ;
d'un serveur ;
d'un service d'identité ;
d'une infrastructure Cloud ;
d'un composant réseau ;
d'un outil d'administration ;
d'un dispositif de sauvegarde ;
d'un processus organisationnel.
Il faut distinguer deux grandes catégories.
Actifs métier : ils correspondent aux activités, services et informations dont l'entreprise a besoin.
Actifs techniques : ils fournissent les moyens techniques permettant de réaliser les activités métier.
Exemple :
ACTIVITÉ MÉTIER
Gestion des clients
        │
        ├── Application métier
        │       │
        │       ├── API
        │       └── Pods Kubernetes
        │
        ├── Base PostgreSQL
        │
        ├── Service d'identité Keycloak
        │
        ├── Réseau et infrastructure Cloud
        │
        └── Sauvegardes

Le schéma représente des dépendances possibles à vérifier et à documenter. Il ne signifie pas que chaque application de NOVA SERVICES utilise nécessairement tous ces composants.
5.2 Travail demandé — Construire le registre des actifs ( optionnel )
À partir des parties 2, 3 et 4, construisez un registre comprenant au minimum 20 actifs ou groupes d'actifs clairement identifiés.
Votre registre doit représenter les différentes zones du SI :
infrastructure interne ;
gestion des identités ;
applications et données ;
Kubernetes et conteneurs ;
développement et déploiement ;
AWS ;
services collaboratifs ;
sécurité et supervision.
Pour chaque actif, indiquez :
Champ
Description
Identifiant
Référence unique, par exemple A01
Nom
Nom explicite de l'actif
Catégorie
Métier, donnée, application, infrastructure, sécurité, etc.
Propriétaire
Responsable métier ou technique, à confirmer si nécessaire
Fonction
Rôle de l'actif dans le SI
Données traitées
Informations stockées, transmises ou administrées
Localisation
Interne, AWS, SaaS ou environnement hybride
Exposition
Internet, interne, distant, partenaire ou non directement exposé
Dépendances
Services nécessaires à son fonctionnement
Criticité estimée
Niveau provisoire de 1 à 4
Justification
Argument métier ou technique soutenant l'évaluation

Exemples d'actifs à examiner : OPNsense, VPN fournisseurs, Keycloak, PostgreSQL, Kafka, GitLab, cluster Kubernetes, AWS IAM, Amazon S3, Amazon EKS, HashiCorp Vault, sauvegardes, Wazuh et OpenSearch.
Vous pouvez regrouper certains composants si cela reste pertinent pour le pilotage. En revanche, les composants ayant des rôles ou des niveaux de risque différents doivent être distingués. Par exemple, le cluster Kubernetes ne doit pas masquer les enjeux propres à ses identités, à ses rôles RBAC, à ses secrets et à son API.
Important : un actif sans propriétaire identifié doit être marqué « propriétaire à désigner ». Ne renseignez pas un nom de responsable fictif comme s'il s'agissait d'une information confirmée.



Partie 6 — Évaluer la criticité des actifs ( optionnel )
6.1 Pourquoi tous les actifs ne sont-ils pas prioritaires au même niveau ?
NOVA SERVICES ne peut pas consacrer les mêmes moyens de sécurité à tous ses composants.
La criticité représente l'importance d'un actif pour l'entreprise. Elle doit être argumentée à partir des conséquences possibles d'un incident.
Pour cette première évaluation, utilisez quatre critères :
Confidentialité : conséquences d'un accès non autorisé.
Intégrité : conséquences d'une modification ou d'une destruction non autorisée.
Disponibilité : conséquences d'une interruption de service.
Impact métier : conséquences sur les activités, les clients, les obligations et la continuité de l'entreprise.
6.2 Échelle de notation
Note
Niveau
Interprétation
1
Faible
Conséquences limitées, solution de remplacement simple
2
Modéré
Perturbation contenue, impact maîtrisable
3
Important
Perturbation notable ou conséquences significatives
4
Critique
Impact majeur sur l'activité ou sur plusieurs services essentiels

Cette échelle est une convention de travail pour le TP. Les notes doivent être justifiées et pourront être révisées lorsque de nouvelles informations seront disponibles.
6.3 Travail demandé — Noter les actifs
Pour chaque actif du registre, attribuez une note aux quatre critères.
Actif
Confidentialité
Intégrité
Disponibilité
Impact métier
Criticité retenue
Justification
Keycloak
À évaluer
À évaluer
À évaluer
À évaluer
À déterminer
Impact potentiel sur l'authentification de plusieurs applications
PostgreSQL
À évaluer
À évaluer
À évaluer
À évaluer
À déterminer
Dépend des données et applications hébergées
GitLab
À évaluer
À évaluer
À évaluer
À évaluer
À déterminer
Impact possible sur le code, les pipelines et les déploiements
Amazon S3
À évaluer
À évaluer
À évaluer
À évaluer
À déterminer
Dépend de la nature des objets stockés et de leur usage
Wi-Fi visiteurs
À évaluer
À évaluer
À évaluer
À évaluer
À déterminer
À évaluer notamment au regard de l'isolation et des accès autorisés

La criticité finale doit refléter les conséquences les plus importantes pour l'entreprise. Il n'est pas demandé de calculer une moyenne automatique sans justification.

Partie 7 — Étudier l'exposition et les dépendances
7.1 Les différentes formes d'exposition
L'exposition ne se limite pas à la présence d'un service directement accessible depuis Internet.
Dans l'environnement de NOVA SERVICES, un actif peut être accessible :
directement depuis Internet ;
depuis le réseau interne ;
via un VPN ;
par un prestataire externe ;
via une application ou une API ;
par l'intermédiaire d'un compte ou d'un service d'identité ;
via une relation de confiance avec AWS ou un service SaaS.
Un composant non exposé directement peut donc rester accessible indirectement après la compromission d'un autre composant.
7.2 Travail demandé — Cartographier l'exposition
Pour chaque actif du registre, indiquez :
son exposition principale ;
les chemins d'accès possibles ;
les comptes ou services susceptibles d'y accéder ;
les autres actifs dont il dépend ;
les informations manquantes pour confirmer son niveau d'exposition.
Sélectionnez ensuite les cinq actifs présentant l'exposition la plus préoccupante et justifiez vos choix.





Ne confondez pas automatiquement exposition et risque : un actif exposé n'est pas nécessairement le plus risqué, et un actif interne peut être très critique.
7.3 Travail demandé — Construire les dépendances
Pour les cinq actifs que vous jugez les plus critiques, identifiez :
les applications qui les utilisent ;
les identités ou rôles nécessaires ;
les flux réseau importants ;
les données échangées ;
les conséquences d'une indisponibilité ;
les actifs susceptibles d'être affectés en cascade.
Exemple :
Application métier
        │
        ├── Keycloak
        │      └── Authentification
        │
        ├── Kubernetes
        │      └── Exécution de l'application
        │
        ├── PostgreSQL
        │      └── Données métier
        │
        └── Infrastructure réseau
               └── Communication entre composants

Ce schéma est un exemple à adapter à l'architecture réelle fournie dans le sujet.

Partie 8 — Construire une cartographie exploitable pour la sécurité
8.1 Objectif de la cartographie
La cartographie doit permettre de comprendre :
où se trouvent les actifs importants ;
comment les zones communiquent ;
quels services sont exposés ;
quelles dépendances sont essentielles ;
où se trouvent les données sensibles ;
quels composants doivent être pris en compte pour la surveillance.
Il ne s'agit pas de produire un schéma réseau exhaustif, mais une représentation utile à la prise de décision.
8.2 Travail demandé
Produisez une cartographie de NOVA SERVICES faisant apparaître au minimum :
Internet et le pare-feu OPNsense ;
les accès VPN des collaborateurs et des fournisseurs ;
les applications et leurs API ;
Keycloak ;
le cluster Kubernetes ;
PostgreSQL et Kafka ;
GitLab et la chaîne CI/CD ;
l'infrastructure AWS et ses principaux services ;
les données ou stockages importants ;
les dispositifs de sauvegarde ;
les composants de collecte et d'analyse des événements.
Utilisez des couleurs ou des formes distinctes pour différencier :
les zones d'hébergement ;
les actifs métier ;
les actifs techniques ;
les points d'entrée ;
les composants de sécurité et de collecte.
Ajoutez une légende. Représentez les flux importants avec des flèches et indiquez leur fonction lorsque cela est connu.
Consigne : ne représentez pas comme certain un flux ou une dépendance qui n'est pas confirmé par le sujet. Utilisez une annotation « à confirmer » si nécessaire.

Partie 9 — Déduire les premiers besoins de sécurité
9.1 De l'actif au besoin
Les parties précédentes ont permis de connaître les activités métier, les données, les actifs, leur criticité, leur exposition et leurs dépendances.
Nous pouvons maintenant commencer à exprimer les besoins de sécurité.
Actif
  │
  ▼
Enjeu métier
  │
  ▼
Conséquence d'un incident
  │
  ▼
Besoin de sécurité
  │
  ▼
Besoin de surveillance

Exemples :
Pour Keycloak, il peut être nécessaire de suivre les échecs d'authentification et les modifications de rôles.
Pour GitLab, il peut être utile de suivre les changements de permissions, les créations de tokens et les déploiements en production.
Pour AWS IAM, il peut être important d'identifier les modifications de droits et la création de nouvelles permissions.
Pour PostgreSQL, les besoins dépendent notamment des données traitées, des accès autorisés et des conséquences d'une modification ou d'une indisponibilité.
Ces exemples introduisent des pistes de surveillance ; ils ne constituent pas encore des règles de détection complètes.
9.2 Travail demandé — Formuler les besoins
Pour les cinq actifs critiques, complétez la matrice suivante :
Actif
Besoin de confidentialité
Besoin d'intégrité
Besoin de disponibilité
Événements à surveiller
Justification
Keycloak
À déterminer
À déterminer
À déterminer
À préciser
À justifier
PostgreSQL
À déterminer
À déterminer
À déterminer
À préciser
À justifier
Kubernetes
À déterminer
À déterminer
À déterminer
À préciser
À justifier
GitLab
À déterminer
À déterminer
À déterminer
À préciser
À justifier
AWS IAM
À déterminer
À déterminer
À déterminer
À préciser
À justifier

Les cinq actifs ci-dessus sont des exemples de travail : vous pouvez les remplacer par les actifs que vous avez effectivement retenus comme prioritaires.
À ce stade, vous devez décrire ce qu'il serait utile de savoir et pourquoi, sans encore concevoir de règle SIEM, choisir un seuil d'alerte ou construire un tableau de bord.

Partie 10 — Relier le travail au Guide d'hygiène informatique de l'ANSSI
Le Guide d'hygiène informatique de l'ANSSI peut servir de référence pour identifier des mesures de sécurité à examiner et pour structurer un plan d'action.
Dans le cadre de ce TP, le lien principal concerne la connaissance du système d'information : identifier les informations sensibles, repérer les composants critiques et maintenir une cartographie simplifiée du réseau.
La démarche de pilotage devient :
Référence de sécurité
        │
       ▼
État constaté du SI
        │
       ▼
Écart à traiter
        │
        ▼
Actif et enjeu concernés
        │
       ▼
Priorité à déterminer
        │
       ▼
Action et indicateur de suivi

Travail demandé
À partir de votre registre et de votre cartographie, identifiez trois éléments pour lesquels une vérification de conformité ou une action de sécurité pourrait être nécessaire.
Pour chacun, précisez :
l'actif concerné ;
la situation constatée ou l'information manquante ;
le risque potentiel ;
l'action à envisager ;
la preuve permettant de vérifier que l'action a été réalisée.
Si le sujet ne permet pas d'établir l'état réel d'une mesure, ne concluez pas que l'entreprise est non conforme. Indiquez plutôt qu'une vérification est nécessaire.

Partie 11 — Construire la première matrice de pilotage
Vous allez maintenant réunir les résultats des parties précédentes dans une matrice qui sera réutilisée dans les prochains TP.
Actif
Activité métier
Criticité
Exposition
Risque pressenti
Besoin de surveillance
Données à rechercher
Keycloak
Administration et accès aux applications
À évaluer
À confirmer
Compromission d'un compte ou modification de droits
Connexions et changements de rôles
Événements d'authentification et d'administration
PostgreSQL
Gestion ou production métier
À évaluer
À confirmer
Accès non autorisé ou indisponibilité
Accès et événements pertinents sur la base
Journaux de la base et contexte des accès
Kubernetes
Exécution des applications
À évaluer
À confirmer
Compromission d'un composant ou modification non autorisée
Changements sensibles et événements du cluster
Journaux de l'API et événements disponibles
GitLab
Développement et déploiement
À évaluer
À confirmer
Modification du code ou déploiement non autorisé
Permissions, tokens et déploiements
Journaux d'audit et événements CI/CD
AWS IAM
Administration des ressources Cloud
À évaluer
À confirmer
Attribution de permissions excessives
Changements de politiques et d'identités
Événements d'audit AWS disponibles

Les lignes sont des exemples : vous devez adapter la matrice aux actifs et aux dépendances retenus.
Cette matrice doit permettre de répondre à quatre questions :
Pourquoi cet actif est-il important ?
Quel incident pourrait avoir un impact significatif ?
Quelles informations permettraient de mieux comprendre cet incident ?
Pourquoi ces informations sont-elles utiles à la décision ?

Partie 12 — Exercice de synthèse : choisir les priorités
La DSI souhaite savoir sur quels éléments concentrer en premier ses efforts de sécurité.
Travail demandé
Sélectionnez les cinq actifs que vous recommandez de surveiller en priorité.
Pour chaque actif, justifiez votre choix à partir de :
sa criticité ;
son exposition ;
les données qu'il traite ;
les activités métier dépendantes ;
les conséquences possibles d'un incident ;
les besoins de surveillance identifiés ;
les incertitudes restant à lever.
Vous devez expliquer pourquoi ces actifs sont prioritaires par rapport à d'autres composants du SI.
Il n'existe pas nécessairement une seule réponse correcte : la qualité de la justification, la cohérence des critères et la traçabilité des décisions sont essentielles.

Partie 13 — Mise en situation professionnelle
13.1 User Story — US-SEC-001
En tant que responsable sécurité de NOVA SERVICES, je veux disposer d'une vision structurée des actifs critiques, de leurs dépendances et de leur exposition, afin de déterminer les priorités de sécurité et les besoins de surveillance du système d'information.
Critères d'acceptation
Le travail est considéré comme satisfaisant si :
les activités métier principales sont identifiées ;
les données importantes et sensibles sont recensées ;
les actifs métier et techniques sont distingués ;
les propriétaires sont identifiés ou signalés comme inconnus ;
la criticité des actifs est justifiée ;
les expositions et les dépendances sont documentées ;
une cartographie du SI est produite ;
les premiers besoins de sécurité et de surveillance sont exprimés ;
les décisions sont reliées aux enjeux métier ;
les hypothèses et les informations manquantes sont clairement signalées.
13.2 Incident INC-001 — Connexion inhabituelle à un compte administrateur
Le responsable informatique vous informe :
« Nous avons reçu une alerte indiquant qu'un compte administrateur s'est connecté depuis une adresse IP inhabituelle. Nous voulons savoir si cet événement est réellement préoccupant. »
Informations disponibles :
Compte : admin-jdupont
Service : administration du SI
Connexion : VPN
Source : Internet
Actif administré : serveur de base de données
Heure : 02 h 37
Travail demandé
Sans encore construire de règle de détection, répondez :
Quels actifs et services sont concernés ?
Pourquoi la base de données peut-elle être critique ?
Quels risques peuvent être associés à cette connexion ?
Quelles informations supplémentaires permettraient de qualifier l'événement ?
Quels journaux ou sources de données pourraient être utiles ?
Qui devrait être informé selon la procédure interne ?
Quelles informations manquent pour conclure qu'il s'agit d'un incident de sécurité ?
Point de méthode : une connexion inhabituelle est un signal à examiner, pas une preuve suffisante de compromission.
13.3 Incident INC-002 — Vulnérabilité critique sur une application métier
Le scanner de vulnérabilités signale une vulnérabilité critique sur un serveur hébergeant une application métier.
Le responsable métier indique :
« Nous ne pouvons pas arrêter l'application aujourd'hui. Elle est utilisée par tous les commerciaux. »
Travail demandé
Répondez aux questions suivantes :
La criticité technique de la vulnérabilité suffit-elle pour décider de la priorité ?
Quelle est la criticité métier de l'application ?
Le serveur ou le service est-il accessible depuis Internet ou par d'autres chemins ?
Existe-t-il un correctif, une mesure de contournement ou une mitigation possible ?
Quelles dépendances et contraintes d'exploitation faut-il connaître ?
Quel risque résiduel subsisterait si la correction était différée ?
Quelles informations devront être collectées dans le TP2 pour prioriser ce risque ?
Cette situation prépare l'analyse des risques et la priorisation des vulnérabilités. Elle ne doit pas être résolue uniquement à partir du score technique fourni par le scanner.

Partie 14 — Bonnes pratiques de pilotage
14.1 Connaître avant de surveiller
Mauvaise démarche :
Déployer un outil
      │
     ▼
Collecter tous les événements
      │
     ▼
Chercher ensuite leur utilité

Démarche de pilotage :
Activités métier
      │
     ▼
Actifs critiques
      │
     ▼
Risques
      │
     ▼
Besoins de surveillance
      │
     ▼
Sources de données
      │
     ▼
Technologies adaptées
14.2 Ne pas confondre criticité technique et criticité métier
Un composant techniquement complexe n'est pas automatiquement l'actif le plus important.
À l'inverse, une application simple peut être indispensable à une activité critique.
La criticité doit être justifiée par les conséquences d'un incident, les dépendances et les besoins de l'entreprise.
14.3 Documenter les propriétaires
Pour chaque actif, il faut pouvoir déterminer :
qui en est responsable ;
qui l'utilise ;
qui l'administre ;
qui évalue son impact métier ;
qui décide des actions en cas d'incident.
Si l'information n'est pas disponible, elle devient un point à clarifier.
14.4 Documenter les incertitudes
Une cartographie ou un registre n'est jamais nécessairement complet dès sa première version.
Il faut distinguer :
les informations confirmées ;
les hypothèses de travail ;
les informations à vérifier ;
les décisions en attente.
Cette distinction évite de prendre des décisions de sécurité sur la base d'une architecture supposée.
14.5 Ne pas choisir les outils trop tôt
Un outil ne définit pas à lui seul le besoin de sécurité. Le raisonnement attendu est :
Besoin métier
      │
     ▼
Risque à maîtriser
      │
Information nécessaire
      │
     ▼
Source de données
      │
     ▼
Méthode d'analyse
      │
     ▼
Outil adapté

Partie 15 — Livrables attendus
votre dossier devra contenir les éléments suivants.
Livrable 1 — Analyse des activités métier
Les quatre activités principales.
Les données nécessaires.
Les applications et composants techniques qui les supportent.
Les conséquences possibles d'un incident.
Livrable 2 — Registre des actifs
Au minimum 20 actifs ou groupes d'actifs clairement identifiés, comprenant :
identifiant ;
catégorie ;
propriétaire ;
fonction ;
données traitées ;
localisation ;
exposition ;
dépendances ;
criticité estimée ;
justification.
Livrable 3 — Inventaire des données importantes
Au minimum six catégories de données, avec leur utilisation, leur responsable, leur hébergement et leurs besoins de sécurité.
Livrable 4 — Évaluation de la criticité
Une évaluation argumentée de la confidentialité, de l'intégrité, de la disponibilité et de l'impact métier.
Livrable 5 — Cartographie du SI
Un schéma montrant les principales zones, les actifs critiques, les points d'entrée, les dépendances et les dispositifs de sécurité.
Livrable 6 — Cartographie des dépendances
Les dépendances des cinq actifs retenus comme prioritaires, avec les impacts possibles d'une indisponibilité ou d'une compromission.
Livrable 7 — Première matrice de pilotage
Une matrice reliant les actifs, leur criticité, leur exposition, les risques pressentis, les besoins de surveillance et les données nécessaires.
Livrable 8 — Analyse des incidents
Une analyse argumentée des situations INC-001 et INC-002.
Livrable 9 — Synthèse décisionnelle
Une page maximum répondant à la question :
Quels sont les cinq actifs que NOVA SERVICES devrait examiner en priorité pour piloter sa sécurité, et pourquoi ?
La synthèse doit mentionner les critères de sélection, les principales incertitudes et les premières actions à envisager.

Partie 16 — Questions de compréhension
Pourquoi faut-il comprendre les activités métier avant de classer les actifs techniques ?
Quelle différence existe entre un actif métier et un actif technique ?
Pourquoi la criticité d'un actif dépend-elle des données et des activités qu'il supporte ?
Quelle différence existe entre criticité, exposition et risque ?
Pourquoi un composant non exposé directement sur Internet peut-il rester concerné par un risque important ?
Pourquoi faut-il identifier les dépendances entre applications, identités, bases de données et infrastructures ?
Pourquoi un actif sans propriétaire identifié est-il difficile à piloter ?
Pourquoi faut-il distinguer les informations confirmées des hypothèses ?
Pourquoi ne faut-il pas choisir immédiatement un SIEM ou un EDR ?
Pourquoi une connexion inhabituelle ne prouve-t-elle pas à elle seule une compromission ?
Pourquoi la criticité technique d'une vulnérabilité ne suffit-elle pas à déterminer la priorité de traitement ?
Comment le registre et la cartographie des actifs contribuent-ils au pilotage de la sécurité ?




