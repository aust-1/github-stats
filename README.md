# github-stats

<div align="center">
<a href="https://github.com/aust-1/github-stats">
<img src="https://github.com/aust-1/github-stats/blob/generated/overview.svg#gh-dark-mode-only" alt="Statistiques GitHub d'Eliott Roussille" />
<img src="https://github.com/aust-1/github-stats/blob/generated/languages.svg#gh-dark-mode-only" alt="Langages les plus utilisés" />
<img src="https://github.com/aust-1/github-stats/blob/generated/overview.svg#gh-light-mode-only" alt="Statistiques GitHub d'Eliott Roussille" />
<img src="https://github.com/aust-1/github-stats/blob/generated/languages.svg#gh-light-mode-only" alt="Langages les plus utilisés" />
</a>
</div>

Ce dépôt calcule chaque jour les statistiques GitHub de [aust-1](https://github.com/aust-1), dépôts privés compris, et publie :

- deux images, `overview.svg` (vue d'ensemble) et `languages.svg` (langages les plus utilisés), affichées sur le [profil GitHub](https://github.com/aust-1) et ci-dessus. Elles suivent automatiquement le thème clair ou sombre de GitHub ;
- `portfolio.json`, des agrégats publics lus au build du site [eliott-roussille.fr](https://eliott-roussille.fr) (dépôt `aust-1/portfolio`).

Il est issu de [jstrieb/github-stats](https://github.com/jstrieb/github-stats) (Jacob Strieb), distribué sous licence [GPL-3.0](LICENSE), comme ce dépôt.

## Fonctionnement

Le workflow [`Generate Stats Images`](.github/workflows/main.yml) :

1. se déclenche à chaque push sur `master`, chaque jour à 04:28 UTC, ou à la main (**Actions** → **Generate Stats Images** → **Run workflow**) ;
2. compile le programme Zig (`zig build`, Zig 0.16.0) ;
3. interroge l'API GitHub avec le jeton `ACCESS_TOKEN` et écrit les images et `portfolio.json` dans un dossier temporaire ;
4. vérifie `portfolio.json` (voir [Confidentialité](#confidentialité)) ;
5. remplace le contenu de la branche [`generated`](https://github.com/aust-1/github-stats/tree/generated) par ces seuls fichiers, puis pousse un commit « Update generated files » signé `github-actions[bot]`.

Une seule exécution tourne à la fois (`concurrency`). L'option **reset_history** du lancement manuel recrée `generated` sans historique (push forcé).

## Configuration

Secrets du dépôt (**Settings** → **Secrets and variables** → **Actions**) :

| Secret | Obligatoire | Rôle |
| ------ | ----------- | ---- |
| `ACCESS_TOKEN` | oui | Jeton personnel **classique** avec les portées `read:user`, `user:email` et `repo`. `read:user` et `repo` servent à lire les métadonnées de l'utilisateur et des dépôts (privés compris), `user:email` à attribuer les commits lors du calcul des lignes modifiées. |
| `EXCLUDE_LANGS` | non | Langages à exclure, séparés par des virgules. Insensible à la casse, motifs glob acceptés (`Jupyter*`). |
| `EXCLUDE_REPOS` | non | Dépôts à exclure (`propriétaire/nom`), séparés par des virgules. Motifs glob acceptés (`aust-1/github-*`). |

Les exclusions s'appliquent de la même façon aux images et à `portfolio.json`. Les noms de langages sont ceux de [GitHub Linguist](https://github.com/github-linguist/linguist/blob/main/lib/linguist/languages.yml).

Les autres options du programme (`--help`) peuvent être passées en argument ou en variable d'environnement dans le workflow : par exemple `--max-retries` / `MAX_RETRIES`, ou `--exclude-private` / `EXCLUDE_PRIVATE` pour ignorer les dépôts privés dans les totaux.

## Fichiers produits

Tous les fichiers sont à la racine de la branche `generated` :

| Fichier | URL stable |
| ------- | ---------- |
| `overview.svg` | https://github.com/aust-1/github-stats/blob/generated/overview.svg |
| `languages.svg` | https://github.com/aust-1/github-stats/blob/generated/languages.svg |
| `portfolio.json` | https://raw.githubusercontent.com/aust-1/github-stats/generated/portfolio.json |

Pour afficher les images dans un README, inclure chaque image deux fois, avec `#gh-dark-mode-only` et `#gh-light-mode-only` (voir le début de ce fichier) : sans cible, l'image a un fond clair ; avec la cible `#gh-dark-mode-only`, elle passe en couleurs sombres.

Une copie de `portfolio.json` est encore publiée à l'ancien chemin `generated/portfolio.json`, le temps que `aust-1/portfolio` passe à la nouvelle URL.

## portfolio.json

Le programme l'écrit avec l'option `--public-json-output-file`. Exemple (fichier du 18/11/2026, listes réduites à un élément) :

```json
{
  "generatedAt": "2026-11-18T13:14:19Z",
  "contributions": { "commits": 4202, "prs": 131, "issues": 241, "reviews": 5, "repos": 27 },
  "stars": 112,
  "linesChanged": 5213142,
  "repoCount": 44,
  "languages": [{ "name": "C", "size": 17926, "color": "#555555" }],
  "repositories": [{ "name": "DaVinciBot/cash", "stars": 5 }]
}
```

| Champ | Contenu |
| ----- | ------- |
| `generatedAt` | Date de génération (UTC, ISO 8601). |
| `contributions` | Contributions de tous les temps : commits, pull requests, issues, revues et dépôts créés. |
| `stars`, `linesChanged`, `repoCount` | Totaux sur les dépôts pris en compte (privés compris, hors `EXCLUDE_REPOS`). |
| `languages` | Taille cumulée par langage (hors `EXCLUDE_LANGS`), triée par taille décroissante ; `color` peut être `null`. |
| `repositories` | Nom et étoiles des dépôts **publics** pris en compte. |

Le site le lit sans jeton dans `scripts/github-stats.ts` et le valide avec un schéma strict, qui refuse toute clé en plus.

### Confidentialité

Le dump complet (`JSON_OUTPUT_FILE`), qui contient les adresses e-mail, les vues et les dépôts privés, reste dans le dossier temporaire du runner et n'est jamais publié. Avant toute publication, le workflow vérifie que `portfolio.json` :

- n'a que les clés attendues ;
- ne contient le nom d'aucun dépôt privé ;
- ne contient pas de champ `emails`, `views` ou `user`.

Si l'une de ces vérifications échoue, rien n'est publié.

## Utilisation locale

```bash
zig build
./zig-out/bin/github-stats --help

# Collecte avec un jeton, dump complet pour analyse (ne pas le publier)
./zig-out/bin/github-stats --access-token "$ACCESS_TOKEN" --json-output-file stats.json --debug

# Regénérer les images et portfolio.json à partir d'un dump, sans appel à l'API
./zig-out/bin/github-stats --json-input-file stats.json \
  --overview-output-file overview.svg \
  --languages-output-file languages.svg \
  --public-json-output-file portfolio.json
```

Pour analyser le dump avec [`jq`](https://github.com/jqlang/jq) :

```bash
# Dépôts triés par nombre de vues (remplacer .views par .stars, .lines_changed…)
jq '.repositories | sort_by(.views) | del(.[].languages)' stats.json

# Langages triés par taille cumulée
jq --raw-output '
  [.repositories[].languages // [] | .[]]
    | group_by(.name)
    | sort_by([.[].size] | add)
    | .[]
    | "\(.[0].name): \([.[].size] | add)"
' stats.json
```

## Limites des chiffres

L'API GitHub donne des résultats imprécis dans certains cas :

- **Lignes modifiées** : les fichiers comme `package-lock.json` gonflent le total, et GitHub ne compte rien pour les dépôts de plus de 10 000 commits. [L'API des statistiques de contributeurs n'étant plus fiable](https://github.com/orgs/community/discussions/192970), le programme clone aussi les dépôts et compte les lignes avec `git`, ce qui peut sous-estimer les contributions faites via des PR fusionnées en squash par quelqu'un d'autre ou avec d'anciennes adresses e-mail.
- **Vues** : souvent trop basses ; un dépôt dont on ne peut pas lire les vues compte pour zéro.
- **Dépôts** : seuls les dépôts avec des commits sont comptés.

Les erreurs de l'API GitHub se résolvent souvent d'elles-mêmes en un ou deux jours, sans modification du code.

## Crédits

- [jstrieb/github-stats](https://github.com/jstrieb/github-stats), dont ce dépôt est issu (GPL-3.0).
- Icônes [GitHub Octicons](https://primer.style/octicons/).
