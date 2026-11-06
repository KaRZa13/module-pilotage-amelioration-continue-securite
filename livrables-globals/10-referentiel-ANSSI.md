# Partie 10 — Lien avec le Guide d'hygiène informatique de l'ANSSI

> Démarche : **référence de sécurité → état constaté du SI → écart à traiter → actif et enjeu concernés → priorité → action et indicateur de suivi.**
>
> Numérotation des mesures : version 2017 du *Guide d'hygiène informatique* de l'ANSSI (42 mesures) — à vérifier par rapport à l'édition en vigueur.
>
> **Le sujet ne permet pas d'établir l'état réel des mesures** : les trois points ci-dessous sont des **vérifications nécessaires**, pas des constats de non-conformité.

## Point 1 — Connaissance du SI et gouvernance des comptes à privilèges

| Rubrique | Contenu |
|---|---|
| Référence ANSSI | Mesure 4 — *Identifier les informations et serveurs les plus sensibles et maintenir un schéma du réseau* ; mesure 5 — *Disposer d'un inventaire exhaustif des comptes privilégiés et le maintenir à jour* ; mesure 8 — *Identifier nommément chaque personne accédant au système et distinguer les rôles utilisateur / administrateur* |
| Actifs concernés | Tous, en particulier Keycloak (A14), AWS IAM (A15), GitLab (A27-A28), API Kubernetes (A23), Vault (A35), PostgreSQL (A18) |
| Situation constatée / information manquante | Aucun propriétaire n'est identifié pour les 48 actifs. Aucun inventaire des comptes à privilèges n'est fourni, alors qu'ils sont répartis dans au moins cinq référentiels. Un compte **générique** `admin` est utilisé sur Keycloak (événement 1). La cartographie de ce TP est une première version non validée par la DSI. |
| Risque potentiel | Comptes orphelins ou excessifs non détectés ; actions d'administration non attribuables à une personne ; décisions de sécurité sans décideur identifié ; incidents mal qualifiés (cf. INC-001). |
| Action à envisager | 1) Faire valider le registre et la cartographie par la DSI et désigner un propriétaire pour chaque actif de criticité 4. 2) Établir l'inventaire des comptes à privilèges (Keycloak, AWS IAM, GitLab, Kubernetes, Vault, PostgreSQL, serveurs Linux), nominatifs, avec leur justification. 3) Remplacer l'usage du compte `admin` générique par des comptes nominatifs. 4) Planifier une revue périodique. |
| Preuve de réalisation | Registre des actifs daté et validé (propriétaires renseignés) ; export daté des comptes à privilèges par référentiel avec propriétaire et justification ; compte-rendu de la première revue des droits. |
| Indicateur de suivi (pour les TP suivants) | % d'actifs critiques avec propriétaire désigné ; nombre de comptes à privilèges non nominatifs ; date de la dernière revue des droits. |

## Point 2 — Accès distants des prestataires (VPN fournisseurs)

| Rubrique | Contenu |
|---|---|
| Référence ANSSI | Mesure 3 — *Maîtriser les risques de l'infogérance* ; mesure 13 — *Privilégier lorsque c'est possible une authentification forte* ; mesure 25 — *Sécuriser les interconnexions réseau dédiées avec les partenaires* ; mesure 36 — *Activer et configurer les journaux des composants les plus importants* |
| Actif concerné | VPN fournisseurs (A07), et les serveurs Linux (A41) / bases (A18) accessibles par ce biais |
| Situation constatée / information manquante | Le VPN fournisseurs est exposé sur Internet et donne accès à des ressources de maintenance [C]. Ne sont **pas documentés** : la liste des prestataires et de leurs comptes, le caractère nominatif des comptes, l'usage du MFA, les ressources autorisées, les plages horaires, la journalisation et sa collecte (le VPN n'apparaît pas dans les sources de Wazuh). |
| Risque potentiel | Compromission d'un prestataire (poste, identifiants) → accès d'administration au SI ; impossibilité de savoir qui a fait quoi ; rebond vers les serveurs critiques. |
| Action à envisager | Vérifier et, si nécessaire, mettre en place : comptes nominatifs par intervenant, MFA, restriction aux seules ressources nécessaires (filtrage OPNsense), activation à la demande ou par plages horaires, journalisation collectée dans Wazuh, clauses de sécurité dans les contrats. |
| Preuve de réalisation | Export de la configuration VPN et des règles OPNsense associées ; liste des comptes prestataires avec titulaire et date d'expiration ; preuve d'activation du MFA ; présence des journaux VPN dans OpenSearch ; extrait des clauses contractuelles. |
| Indicateur de suivi | Nombre de comptes prestataires actifs / non nominatifs / sans MFA ; % de connexions prestataires rattachées à une intervention planifiée. |

## Point 3 — Sauvegardes et capacité de restauration

| Rubrique | Contenu |
|---|---|
| Référence ANSSI | Mesure 37 — *Définir et appliquer une politique de sauvegarde des composants critiques* ; mesure 9 — *Attribuer les bons droits sur les ressources sensibles* |
| Actifs concernés | Serveur de sauvegarde (A36), Amazon S3 (A33), AWS IAM (A15), et toutes les données critiques sauvegardées (PostgreSQL A18, Samba A21, GitLab A27, Keycloak A14, Vault A35) |
| Situation constatée / information manquante | Une solution de sauvegarde existe (Restic / Veeam) et S3 contient des sauvegardes [C]. Ne sont **pas documentés** : le périmètre exact sauvegardé, la fréquence, l'existence d'une copie **hors ligne ou immuable**, la séparation des droits (les identités d'administration courantes peuvent-elles supprimer les sauvegardes ?), les **tests de restauration** et les durées de restauration atteintes. |
| Risque potentiel | Un rançongiciel ou un attaquant disposant de droits IAM / d'administration supprime les sauvegardes avant de chiffrer la production : **perte définitive des données** et arrêt prolongé de l'activité. |
| Action à envisager | Vérifier le périmètre et la politique de sauvegarde ; vérifier l'immuabilité des sauvegardes S3 (versioning / verrouillage des objets) et l'existence d'une copie isolée ; séparer les comptes de sauvegarde des comptes d'administration courants ; réaliser un **test de restauration** complet d'une application critique (base PostgreSQL + application). |
| Preuve de réalisation | Politique de sauvegarde validée ; configuration des buckets S3 (versioning, verrouillage) ; procès-verbal daté de test de restauration avec durée mesurée et résultat ; liste des comptes autorisés à supprimer des sauvegardes. |
| Indicateur de suivi | Taux de réussite des sauvegardes ; date du dernier test de restauration réussi ; durée de restauration mesurée vs objectif ; % des actifs critiques couverts par une sauvegarde testée. |

## Autres vérifications identifiées (non retenues dans les trois prioritaires)

| Sujet | Référence ANSSI | Vérification nécessaire |
|---|---|---|
| Isolation du Wi-Fi visiteurs | Mesure 20 — sécurité des réseaux Wi-Fi et séparation des usages | Test d'accès depuis le Wi-Fi visiteurs vers le LAN, le VPN et les interfaces d'administration |
| Logiciels obsolètes sur les postes | Mesures 34-35 — politique de mise à jour, anticipation des fins de support | Inventaire des versions, taux de postes à jour |
| Collecte des journaux | Mesure 36 — journaux des composants les plus importants | Activation et collecte des journaux PostgreSQL, AWS, Vault, VPN, sauvegardes |
| Procédure de gestion des incidents | Mesure 40 — procédure de gestion des incidents | Existence, diffusion, contacts (cf. INC-001) |
| Usage de Discord | Mesure 24 — protéger sa messagerie professionnelle (et maîtrise des services utilisés) | Statut officiel, données échangées, règles d'usage |
| Administration depuis des postes dédiés | Mesures 27-28 — postes et réseau d'administration | Les administrateurs (dont `admin-jdupont`) administrent-ils depuis un poste dédié ? |
