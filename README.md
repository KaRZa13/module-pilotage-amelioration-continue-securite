# Module Pilotage & Amélioration Continue de la Sécurité

Ce dépôt regroupe plusieurs projets sous forme de submodules Git :

| Submodule        | Dépôt                                                        |
|------------------|--------------------------------------------------------------|
| `api-gateway`    | [KaRZa13/api-gateway](https://github.com/KaRZa13/api-gateway)       |
| `pre-commit`     | [KaRZa13/pre-commit](https://github.com/KaRZa13/pre-commit)         |
| `dast-owasp`     | [KaRZa13/dast-owasp](https://github.com/KaRZa13/dast-owasp)         |
| `sonarqube-nova` | [KaRZa13/sonarqube-nova](https://github.com/KaRZa13/sonarqube-nova) |
| `traefik`        | [KaRZa13/traefik](https://github.com/KaRZa13/traefik)               |

## Cloner le projet

Cloner le dépôt avec tous ses submodules :

```bash
git clone --recurse-submodules git@github.com:KaRZa13/module-pilotage-amelioration-continue-securite.git
```

Si le dépôt a déjà été cloné sans les submodules :

```bash
git submodule update --init --recursive
```
