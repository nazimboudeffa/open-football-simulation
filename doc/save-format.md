# 💾 Format de sauvegarde

## 1. Vue d'ensemble

- Une sauvegarde est un **unique objet JSON** correspondant à `gameState` (`state.js → saveGame()`).
- Elle est téléchargée par le navigateur sous le nom :

```text
onlinefootballsimulation-save-s{season}r{round}.json
```

- Le chargement (`loadGame()`) lit le fichier, exécute `JSON.parse`, puis remplace directement `gameState`.
- **Il n'y a ni `localStorage`, ni sauvegarde automatique, ni version de format.** Fermer l'onglet sans exporter perd la partie.

## 2. Schéma général

```jsonc
{
  "playerTeamIndex": 0,            // index du club joueur dans teams[]
  "round": 5,                      // journée courante (1 à 35)
  "season": 2,                     // saison courante
  "budget": 7345000,               // budget du club joueur
  "weeklyIncome": 412000,          // revenus de la dernière journée
  "lastIncomeBreakdown": {         // détail affiché dans la modale
    "👕 Shirt Sponsor": 13500,
    "🏟️ North Stand": 5200,
    "🎫 Match Day": 393300
  },
  "teams": [ /* 18 équipes */ ],
  "transferMarket": [ /* joueurs avec askingPrice */ ],
  "sponsors": {
    "shirt": { /* contrat actif ou null */ },
    "north": null,
    "south": null,
    "east": null,
    "west": null
  },
  "sponsorOffers": [ /* 6 offres, roundsRemaining: null */ ],
  "stadium": { "capacity": 15000, "ticketPrice": 25 },
  "kit": { "primary": "#ff0000", "secondary": "#ffffff" },
  "fixtures": [ /* 34 journées de 9 matchs */ ],
  "selectedLineup": [ "a1b2c3d4e", "f6g7h8i9j" ],
  "trainingQueue": [],
  "selectedTrainingType": "elite",
  "messageLog": [ /* 100 entrées maximum, plus récentes en tête */ ]
}
```

## 3. Champs racine

| Champ | Type | Description |
|---|---|---|
| `playerTeamIndex` | `number` | Index du club joueur dans `teams`. Toujours `0` en pratique (le club est choisi à l'index cliqué). |
| `round` | `number` | Journée suivante à jouer. Devient `35` après la dernière journée, puis revient à `1` au changement de saison. |
| `season` | `number` | Saison en cours (commence à 1). |
| `budget` | `number` | Budget du club joueur en euros. |
| `weeklyIncome` | `number` | Total des revenus de la dernière journée simulée. |
| `lastIncomeBreakdown` | `object` | Libellé → montant, pour l'affichage. |
| `teams` | `Team[18]` | Les 18 clubs. |
| `transferMarket` | `Player[]` | Joueurs disponibles à l'achat. |
| `sponsors` | `Sponsors` | Contrats actifs, `null` si l'emplacement est libre. |
| `sponsorOffers` | `Sponsor[]` | Offres en attente. |
| `stadium` | `object` | `{ capacity, ticketPrice }`. |
| `kit` | `object` | `{ primary, secondary }` au format hexadécimal. |
| `fixtures` | `Fixture[][]` | 34 journées, chacune avec 9 matchs. |
| `selectedLineup` | `string[]` | Identifiants des joueurs titulaires (maximum 11). |
| `trainingQueue` | `string[]` | Joueurs cochés mais pas encore envoyés en stage. |
| `selectedTrainingType` | `string \| null` | `basic`, `intensive` ou `elite`. |
| `messageLog` | `Message[]` | 100 entrées maximum, la plus récente en premier. |

## 4. Objets imbriqués

### `Team`

| Champ | Type | Description |
|---|---|---|
| `name` | `string` | Nom du club. |
| `players` | `Player[]` | Effectif (16 au départ). |
| `stats` | `object` | `{ played, won, drawn, lost, goalsFor, goalsAgainst, points }`. |
| `budget` | `number` | Généré (3 à 10 M€) mais **jamais utilisé** par l'IA. |
| `isPlayer` | `boolean` | `true` uniquement pour le club joueur. |

> `isPlayer` sert au filtrage de l'affichage et aux relégations. Le club joueur est identifié par `playerTeamIndex` dans tout le reste du code.

### `Player`

| Champ | Type | Description |
|---|---|---|
| `id` | `string` | Identifiant aléatoire de 9 caractères (base 36). |
| `name` | `string` | Prénom + nom. |
| `position` | `string` | `GK`, `DEF`, `MID` ou `FWD`. |
| `age` | `number` | Âge en années. |
| `energy` | `number` | 0–100. |
| `condition` | `number` | 0–100. |
| `strength` | `number` | 0–99. |
| `value` | `number` | Valeur calculée en euros. |
| `inTraining` | `boolean` | Stage en cours (traité en fin de journée). |
| `trainingBonus` | `number` | Gain de force à appliquer. |
| `trainingEnergyCost` | `number` | Coût énergétique à appliquer. |
| `isYouth` | `boolean` | Présent uniquement sur les joueurs issus de l'académie. |
| `potential` | `number` | Présent uniquement sur les jeunes : plafond de progression. |
| `askingPrice` | `number` | Présent uniquement sur les joueurs du marché des transferts. |
| `retired` | `boolean` | Drapeau **transitoire** posé puis retiré pendant le vieillissement ; ne devrait pas apparaître dans une sauvegarde. |

### `Fixture`

| Champ | Type | Description |
|---|---|---|
| `home` | `number` | Index de l'équipe à domicile dans `teams`. |
| `away` | `number` | Index de l'équipe à l'extérieur. |
| `homeGoals` | `number \| null` | Score, `null` si non joué. |
| `awayGoals` | `number \| null` | Score, `null` si non joué. |
| `played` | `boolean` | `true` une fois le match simulé. |

### `Sponsor` (contrat ou offre)

| Champ | Type | Description |
|---|---|---|
| `id` | `string` | Identifiant de l'offre. |
| `name` | `string` | Nom de la marque. |
| `type` | `string` | `shirt`, `north`, `south`, `east` ou `west`. |
| `amount` | `number` | Montant annuel en euros. |
| `duration` | `number` | Durée en saisons (1 à 3). |
| `roundsRemaining` | `number \| null` | Journées restantes pour un contrat actif, `null` pour une offre. |

### `Message`

| Champ | Type | Description |
|---|---|---|
| `message` | `string` | Texte affiché dans le fil d'actualité. |
| `isError` | `boolean` | Style rouge si `true`. |
| `season` | `number` | Saison au moment du message. |
| `round` | `number` | Journée au moment du message. |
| `timestamp` | `number` | Horodatage `Date.now()` (non utilisé à l'affichage). |

## 5. Exemple minimal

```json
{
  "playerTeamIndex": 0,
  "round": 1,
  "season": 1,
  "budget": 5000000,
  "weeklyIncome": 0,
  "lastIncomeBreakdown": {},
  "teams": [
    {
      "name": "Dynamo Thunderstrike",
      "isPlayer": true,
      "budget": 6200000,
      "stats": { "played": 0, "won": 0, "drawn": 0, "lost": 0, "goalsFor": 0, "goalsAgainst": 0, "points": 0 },
      "players": [
        {
          "id": "k3j9x0a1b",
          "name": "Marco Rossi",
          "position": "GK",
          "age": 24,
          "energy": 94,
          "condition": 88,
          "strength": 71,
          "value": 4615000,
          "inTraining": false,
          "trainingBonus": 0,
          "trainingEnergyCost": 0
        }
      ]
    }
  ],
  "transferMarket": [],
  "sponsors": { "shirt": null, "north": null, "south": null, "east": null, "west": null },
  "sponsorOffers": [],
  "stadium": { "capacity": 15000, "ticketPrice": 25 },
  "kit": { "primary": "#ff0000", "secondary": "#ffffff" },
  "fixtures": [
    [
      { "home": 0, "away": 17, "homeGoals": null, "awayGoals": null, "played": false }
    ]
  ],
  "selectedLineup": ["k3j9x0a1b"],
  "trainingQueue": [],
  "selectedTrainingType": null,
  "messageLog": []
}
```

## 6. Tokens d'identifiants

- Les joueurs sont référencés par `id` dans `selectedLineup`, `trainingQueue` et `sponsors`, jamais par index.
- Les identifiants sont générés par `Math.random().toString(36).substr(2, 9)` : aléatoires, **sans garantie d'unicité**. Une collision (probabilité très faible) casserait la composition.
- Si un joueur vendu ou retraité reste dans `selectedLineup`, `isLineupValid()` peut échouer : `sellPlayer()` nettoie le cas, la retraite non (la composition est vidée en fin de saison).

## 7. Compatibilité et limites

| Limite | Conséquence |
|---|---|
| Absence de version | impossible de détecter un format ancien et de le migrer |
| Aucune validation | un JSON incomplet ou mal typé est accepté tel quel |
| Champs manquants | si `messageLog` est absent, `showNotification()` lève une erreur juste après le chargement |
| `loadGame()` dans un `try` | l'erreur est attrapée et affichée, mais `gameState` a déjà été remplacé : la partie peut se retrouver dans un état incohérent |
| `innerHTML` + noms issus du fichier | un fichier de sauvegarde piégé peut injecter du HTML/JavaScript (XSS) |
| Pas d'autosave | un crash navigateur ou une fermeture d'onglet perd la progression |

## 8. Versionnage recommandé

Pour rendre les sauvegardes durables, il est conseillé d'ajouter :

```json
{
  "saveVersion": 2,
  "gameState": { }
}
```

puis, au chargement :

1. valider la présence de `saveVersion` ;
2. appliquer les migrations nécessaires (compléter les champs manquants avec des valeurs par défaut) ;
3. valider/typer les champs critiques (`teams.length === 18`, `fixtures.length === 34`, tableaux d'identifiants) ;
4. échapper systématiquement les chaînes affichées, ou préférer `textContent` à `innerHTML` ;
5. proposer une sauvegarde automatique dans `localStorage` en complément de l'export de fichier.
