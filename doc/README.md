# 📚 Documentation — Open Football Simulation

Bienvenue dans la documentation du projet. Elle décrit le jeu, son architecture, ses règles internes et son code.

La documentation est rédigée en **français** ; les identifiants du code (fonctions, variables, fichiers) restent en anglais.

## Sommaire

| Document | À lire si vous voulez… |
|---|---|
| [architecture.md](architecture.md) | comprendre comment le code est organisé, l'ordre de chargement, le cycle d'une journée et d'une saison |
| [gameplay.md](gameplay.md) | connaître les règles et formules : match, énergie, économie, transferts, sponsors, stade, jeunes |
| [reference.md](reference.md) | retrouver rapidement une fonction ou une constante du code |
| [save-format.md](save-format.md) | comprendre le format des sauvegardes JSON et leurs limites |
| [development.md](development.md) | lancer, modifier, déboguer le projet et connaître les bugs/chantiers connus |

## Le projet en bref

- **Type** : jeu de management de football solo, nostalgique (inspiré de FIFA Manager / Premier Manager PS1).
- **Stack** : HTML, CSS et JavaScript vanilla. Aucun framework, aucun build, aucune dépendance npm.
- **Exécution** : 100 % côté navigateur. Il suffit d'ouvrir `index.html`.
- **Contenu** : 18 clubs, 34 journées par saison, 16 joueurs par effectif, transferts, entraînement, sponsors, stade, jeunes, retraites, événements aléatoires, sauvegarde JSON et progression multi-saisons.
- **Taille** : environ 2 900 lignes et 110 Ko (HTML + CSS + JS + README).
- **Licence** : MIT.

## Démarrage rapide

1. Ouvrir `index.html` dans un navigateur moderne.
2. Choisir un club et les couleurs du maillot.
3. Cliquer sur **NEW GAME**.
4. Gérer l'effectif, la composition, les transferts, l'entraînement, les sponsors et le stade.
5. Cliquer sur **END ROUND** pour simuler la journée.

Un serveur local est optionnel mais pratique pour éviter certains comportements de `file://` :

```bash
npx serve .
# ou
python -m http.server 8000
```

## Conventions de lecture

- Les chemins sont relatifs à la racine du dépôt.
- Les références au code utilisent le format `fichier.js → fonction()`.
- Les montants sont en euros (entiers, sans décimales).
- Les « rounds » du code correspondent aux **journées** de championnat.
- Les blocs `⚠️` signalent un bug ou une limite connue, pas une fonctionnalité voulue.

## Avertissement

Le projet est un **prototype jouable**, pas un jeu équilibré ou testé automatiquement. Plusieurs mécaniques sont volontairement simples, et certaines comportements sont bugués. La liste à jour se trouve dans [development.md](development.md#bugs-et-limites-connus).
