# Partie 16 — Questions de compréhension

### 1. Pourquoi faut-il comprendre les activités métier avant de classer les actifs techniques ?

Parce que la valeur d'un actif technique **n'est pas intrinsèque** : elle vient des activités et des données qu'il supporte. Le même serveur PostgreSQL est critique s'il contient les données clients et la facturation, et secondaire s'il héberge une base de test. Partir des activités permet de savoir *ce que l'entreprise ne peut pas se permettre de perdre*, puis de remonter vers les composants qui le rendent possible. Sans cela, on classe les actifs selon leur complexité ou leur visibilité, et l'on protège en priorité ce qui est technique plutôt que ce qui est important.

### 2. Quelle différence existe entre un actif métier et un actif technique ?

- Un **actif métier** est une activité, un processus ou une information dont l'entreprise a besoin pour fonctionner (ex. la gestion des clients, les données de facturation). Il porte la **valeur**.
- Un **actif technique** est un moyen qui permet de réaliser ces activités (application, serveur, base, réseau, service d'identité). Il porte les **vulnérabilités**.

Le risque naît de la rencontre des deux : une vulnérabilité d'un actif technique devient préoccupante parce qu'elle menace un actif métier.

### 3. Pourquoi la criticité d'un actif dépend-elle des données et des activités qu'il supporte ?

Parce que la criticité mesure les **conséquences d'un incident**, et que ces conséquences sont métier : perte de chiffre d'affaires, atteinte aux données personnelles, arrêt de la production, perte de confiance. Kubernetes ou Keycloak ne sont critiques chez NOVA SERVICES que parce qu'ils exécutent ou authentifient les applications de gestion des clients et de production. Un actif qui supporte plusieurs activités (Keycloak) cumule leurs enjeux ; un actif qui n'en supporte aucune d'importance reste peu critique, quelle que soit sa technologie.

### 4. Quelle différence existe entre criticité, exposition et risque ?

| Notion | Question | Exemple NOVA SERVICES |
|---|---|---|
| **Criticité** | Quelle serait la gravité si l'actif était atteint ? | PostgreSQL : 4 (données clients) |
| **Exposition** | À quel point l'actif est-il accessible à une menace ? | PostgreSQL : indirecte ; VPN fournisseurs : directe |
| **Risque** | Quelle est la possibilité qu'une menace exploite une vulnérabilité **et** provoque un impact ? | Fuite des données clients via une application exposée vulnérable |

Le risque combine **impact** (lié à la criticité) et **vraisemblance** (liée à l'exposition, aux vulnérabilités et aux menaces). Un actif très exposé mais peu critique (Wi-Fi visiteurs isolé) peut présenter un risque faible ; un actif interne très critique (sauvegardes) peut présenter un risque élevé.

### 5. Pourquoi un composant non exposé directement sur Internet peut-il rester concerné par un risque important ?

Parce que les attaques procèdent **par rebonds**. Un attaquant compromet d'abord un point exposé (application vulnérable, compte VPN, poste par hameçonnage), puis utilise les relations de confiance pour atteindre ce qui ne l'était pas : la base PostgreSQL via les identifiants de l'application, l'API Kubernetes via un compte de service, les sauvegardes S3 via une clé IAM. Les rançongiciels ciblent justement les serveurs de sauvegarde, qui ne sont jamais exposés. L'exposition doit donc intégrer les chemins **indirects** : VPN, identités, applications, relations de confiance avec AWS ou les SaaS.

### 6. Pourquoi faut-il identifier les dépendances entre applications, identités, bases de données et infrastructures ?

Pour trois raisons :
- **Évaluer les effets en cascade** : la compromission de Keycloak touche toutes les applications fédérées ; celle de GitLab CI atteint Kubernetes et AWS.
- **Hériter correctement la criticité** : un composant dont dépendent des actifs critiques devient lui-même critique (DNS, Vault).
- **Préparer la réponse et la reprise** : savoir dans quel ordre restaurer, quoi isoler sans tout arrêter, et quels journaux croiser pour comprendre un incident.

### 7. Pourquoi un actif sans propriétaire identifié est-il difficile à piloter ?

Parce que personne n'est en mesure de : définir son niveau de criticité métier, accepter un risque résiduel, arbitrer entre disponibilité et correction (cf. INC-002), valider les droits d'accès, ni décider en cas d'incident. Les actions de sécurité le concernant n'ont pas de destinataire, et les écarts ne sont pas suivis. Sans propriétaire, l'actif est géré « par défaut », c'est-à-dire souvent pas du tout : c'est typiquement ainsi qu'apparaissent les serveurs oubliés, non mis à jour, et les comptes orphelins.

### 8. Pourquoi faut-il distinguer les informations confirmées des hypothèses ?

Pour **ne pas prendre de décisions sur une architecture supposée**. Si l'on suppose à tort que GitLab n'est pas exposé ou que le Wi-Fi visiteurs est isolé, on sous-estime le risque et on ne surveille pas ce qu'il faudrait. Inversement, présenter une hypothèse comme un fait peut conduire à déclarer une non-conformité inexistante. Marquer les hypothèses permet de savoir **quelles conclusions sont fragiles**, de lister les vérifications à mener et de réviser les décisions quand l'information arrive.

### 9. Pourquoi ne faut-il pas choisir immédiatement un SIEM ou un EDR ?

Parce qu'un outil ne répond qu'aux questions qu'on lui pose. Sans connaître les actifs critiques, les risques et les besoins de surveillance, on collecte tout (coût, bruit, alertes impossibles à qualifier — ce que NOVA SERVICES constate déjà) ou on oublie l'essentiel (chez NOVA SERVICES : les journaux PostgreSQL et l'audit AWS ne sont pas collectés alors que ce sont des actifs de criticité 4). La démarche correcte est : besoin métier → risque → information nécessaire → source de données → méthode d'analyse → **outil adapté**. L'outil vient en dernier et se juge à sa capacité à couvrir les besoins identifiés.

### 10. Pourquoi une connexion inhabituelle ne prouve-t-elle pas à elle seule une compromission ?

Parce que « inhabituel » signifie seulement « différent de la référence connue ». De nombreuses causes légitimes existent : intervention d'astreinte, déplacement, changement de fournisseur d'accès, nouveau prestataire. Pour conclure à une compromission, il faut des éléments supplémentaires : confirmation par le titulaire, absence d'intervention planifiée, actions illégitimes pendant la session, corrélation avec d'autres signaux. Traiter chaque anomalie comme un incident sature les équipes ; l'ignorer fait rater les vrais incidents. D'où la nécessité de **qualifier** (cf. INC-001).

### 11. Pourquoi la criticité technique d'une vulnérabilité ne suffit-elle pas à déterminer la priorité de traitement ?

Parce que le score technique (CVSS) décrit la faille **hors contexte**. La priorité réelle dépend aussi de : la criticité métier de l'actif touché, son exposition effective, l'exploitabilité (code d'exploitation public, exploitation active), les mesures compensatoires existantes, et les contraintes d'exploitation. Une vulnérabilité critique sur un serveur de test isolé est moins prioritaire qu'une vulnérabilité « élevée » sur l'application exposée qui contient les données clients. La décision doit aussi intégrer les contraintes métier, comme dans INC-002, et peut passer par une mitigation temporaire plutôt qu'un arrêt.

### 12. Comment le registre et la cartographie des actifs contribuent-ils au pilotage de la sécurité ?

Ils constituent **le référentiel commun** de toutes les étapes suivantes :
- **Analyse de risques** (TP2) : chaque risque est rattaché à un actif, sa criticité et son exposition.
- **Surveillance** : les besoins de surveillance et les sources de données découlent des actifs prioritaires ; les angles morts deviennent visibles (ex. absence d'audit AWS).
- **Indicateurs** : on peut mesurer des couvertures (% d'actifs critiques avec propriétaire, sauvegarde testée, journaux collectés).
- **Priorisation des vulnérabilités et des incidents** : savoir instantanément si une alerte touche un actif de criticité 4 et exposé.
- **Amélioration continue** : le registre est révisé à chaque incident, changement ou audit, et permet de suivre l'évolution de l'exposition dans le temps.

Sans registre ni cartographie, la sécurité réagit aux alertes au fil de l'eau ; avec eux, elle peut **décider** et **démontrer** ses priorités.
