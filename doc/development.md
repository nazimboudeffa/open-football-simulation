# 🛠️ Guide de développement

## 1. Prérequis et lancement

Aucune installation n'est nécessaire : le projet est du HTML/CSS/JavaScript pur.

```bash
# Méthode la plus simple
# → double-cliquer sur index.html

# Ou via un petit serveur local (recommandé)
npx serve .
# ou
python -m http.server 8000
```

En cas de modification, il suffit de recharger la page : il n'y a **aucune étape de build**.

## 2. Débogage

- Tout l'état est dans la variable globale `gameState` : dans la console du navigateur, tapez `gameState` ou `gameState.teams[0].players` pour l'inspecter.
- `getPlayerTeam()` renvoie le club joueur ; `gameState.fixtures` contient le calendrier ; `gameState.messageLog` l'historique.
- Les erreurs de chargement de sauvegarde sont tracées via `console.error` dans `state.js → loadGame()`.
- Pour tester une saison rapidement, jouez plusieurs journées en boucle ou modifiez `gameState.round` / `gameState.budget` à la main puis appelez `updateAllUI()`.
- L'avantage du club joueur (+10 de force) est appliqué à `startGame()` ; pour rééquilibrer vos tests, ajustez `players.js → boostPlayerTeam()`.

## 3. Conventions du code

- Français uniquement dans les commentaires d'interface, anglais pour les identifiants.
- Un fichier = un domaine fonctionnel.
- Fonctions globales déclarées avec `function` (obligatoire pour les gestionnaires `onclick` inline).
- Aucune classe, aucun module : toute nouvelle fonction est accessible partout.
- Les messages utilisateur passent par `showNotification(message, isError)`.
- Les montants sont des entiers (pas de décimales, pas de centimes).
- Après une action qui modifie `gameState`, appelez `updateAllUI()` (ou la fonction de rendu ciblée).

## 4. Recettes

### Ajouter une constante de jeu

1. L'ajouter dans `js/data.js`.
2. L'exporter dans le bloc `module.exports` final (par cohérence, même s'il est inutilisé dans le navigateur).

### Ajouter un événement aléatoire

Ajouter une entrée dans `RANDOM_EVENTS` (`data.js`) :

```js
{ type: "positive", title: "🎉 Titre", message: "Description…", effect: { budget: 100000 } }
```

Effets reconnus par `triggerRandomEvent()` : `budget`, `energyAll`, `conditionAll`, `strengthRandom`.

### Ajouter un onglet

1. Ajouter un bouton `.nav-tab` avec `data-panel="mon-panel"` dans `index.html`.
2. Ajouter un `<div class="panel" id="panel-mon-panel">`.
3. Ajouter le rendu dans `updateAllUI()` (`ui.js`).

> L'activation des onglets est générique : `setupEventListeners()` se base sur `data-panel`.

### Ajouter un revenu par journée

1. Le calculer dans `processPlayerTeamRound()` (`match.js`).
2. L'ajouter à `breakdown` et à `income`.
3. L'afficher via `renderIncomeBreakdown()` si le revenu est permanent.

## 5. Bugs et limites connus

| # | Sévérité | Zone | Problème | Où |
|---|---|---|---|---|
| 1 | 🔴 Critique | Calendrier | Seules 81 des 153 affiches existent ; chaque club ne rencontre que 9 adversaires ; le plan est identique chaque saison | `match.js → generateFixtures()` |
| 2 | 🔴 Critique | Boucle de jeu | `endRound()` n'est pas protégé contre le double-clic : deux journées peuvent être simulées | `main.js → endRound()` |
| 3 | 🔴 Critique | Sauvegarde | Aucune validation ni version ; un champ manquant casse le chargement ; injection HTML possible via `innerHTML` | `state.js → loadGame()`, `ui.js` |
| 4 | 🔴 Critique | Composition | La sélection automatique peut produire moins de 11 joueurs et bloquer la journée | `lineup.js → autoSelectLineup()` |
| 5 | 🟠 Majeur | Fin de saison | La relégation du club joueur n'a aucun effet (`isPlayerRelegated` inutilisé) | `match.js → processEndOfSeason()` |
| 6 | 🟠 Majeur | Billetterie | Affluence supérieure à la capacité à 10 € (taux non plafonné à 100 %) ; prix maximum toujours optimal | `stadium.js → calculateExpectedAttendance()` |
| 7 | 🟠 Majeur | Économie | L'entraînement crée de la valeur nette à chaque journée ; pas de salaires ni de charges | `training.js`, `players.js` |
| 8 | 🟠 Majeur | IA | Les clubs IA ne se fatiguent jamais (seul le club joueur est mis à jour) | `match.js → processPlayerTeamRound()` |
| 9 | 🟠 Majeur | IA | Les clubs IA ne recrutent ni ne remplacent leurs retraités : le championnat s'affaiblit chaque saison | `players.js → ageAllPlayers()` |
| 10 | 🟡 Mineur | Match | L'« upset » à 8 % peut inverser un score de façon incohérente (0-0 devient 1-0, 5-0 devient une défaite) | `match.js → simulateMatch()` |
| 11 | 🟡 Mineur | Campagne | Un joueur aligné peut être envoyé en stage et en bénéficier le même match | `lineup.js`, `training.js` |
| 12 | 🟡 Mineur | Composition | Aucun contrôle de poste : 11 attaquants sont acceptés | `lineup.js → toggleLineup()` |
| 13 | 🟡 Mineur | UI | Le tri de la colonne « # » de l'effectif ne fait rien (`getValue(p, i)` appelé sans index) | `ui.js → updateSquadTable()`, `getSortedData()` |
| 14 | 🟡 Mineur | UI | Prix de vente affiché à 80 % fixe alors que le prix réel est aléatoire entre 70 % et 89 % | `ui.js → updateTransferMarket()` vs `transfers.js → sellPlayer()` |
| 15 | 🟡 Mineur | Sponsors | Plusieurs offres peuvent viser le même emplacement ; offres régénérées disparaissent sans avertissement | `sponsors.js → generateSponsorOffers()` |
| 16 | 🔵 Propreté | Dépôt | README obsolète (arborescence `football-manager-v2`, `assets/` absent, URL de clone factice) et titre `index.html` encore « Online Football Simulation » | `README.md`, `index.html` |
| 17 | 🔵 Propreté | Dépendances | Google Fonts et Plausible sont chargés depuis Internet alors que le README annonce « no external API calls » | `index.html` |
| 18 | 🔵 Propreté | Code | `module.exports` inerte dans le navigateur ; `String.prototype.substr` déprécié | tous les `js/*.js`, `players.js → generateId()` |

## 6. Correctifs suggérés

### 6.1 Calendrier (bug #1)

Remplacer `generateFixtures()` par un vrai championnat toutes rondes avec rotation :

```js
function generateFixtures() {
    const n = 18;
    const teams = [...Array(n).keys()];
    const firstHalf = [];

    for (let r = 0; r < n - 1; r++) {
        const round = [];
        for (let i = 0; i < n / 2; i++) {
            const a = teams[i];
            const b = teams[n - 1 - i];
            // Alterner les terrains d'une journée à l'autre
            round.push(r % 2 === 0 ? { home: a, away: b } : { home: b, away: a });
        }
        firstHalf.push(round);
        // Rotation : l'équipe 0 reste fixe, les autres tournent
        teams.splice(1, 0, teams.pop());
    }

    // Seconde moitié : matchs retour
    const secondHalf = firstHalf.map(round =>
        round.map(f => ({ home: f.away, away: f.home }))
    );

    gameState.fixtures = [...firstHalf, ...secondHalf].map(round =>
        round.map(f => ({ ...f, homeGoals: null, awayGoals: null, played: false }))
    );
}
```

Vérifications à faire après correction : 34 journées, 153 paires distinctes jouées 2 fois, 17 matchs à domicile par club, 18 équipes distinctes par journée.

### 6.2 Protection de la journée (bug #2)

Dans `endRound()`, ajouter une garde et désactiver le bouton pendant la simulation :

```js
let roundInProgress = false;

function endRound() {
    if (roundInProgress) return;
    if (isSeasonOver()) { showSeasonEndModal(); return; }
    if (!isLineupValid()) { showNotification('You need 11 players in the lineup!', true); return; }

    roundInProgress = true;
    const btn = document.getElementById('endRoundBtn');
    if (btn) btn.disabled = true;

    // … simulation …

    setTimeout(() => {
        // … à la fin du traitement :
        roundInProgress = false;
        if (btn) btn.disabled = false;
    }, 1800);
}
```

### 6.3 Billetterie (bug #6)

```js
const attendanceRate = Math.min(1, Math.max(0.4, 1 - (price - 15) * 0.008));
```

Pour redonner du sens au prix des billets, revoir le plancher (par exemple `Math.max(0.15, …)`) et/ou rendre le revenu dépendant du classement et de la réputation.

### 6.4 Composition (bug #4)

Compléter la sélection automatique avec les meilleurs joueurs restants lorsque les 11 places ne sont pas toutes pourvues, et afficher un avertissement explicite plutôt que de bloquer silencieusement.

### 6.5 Sauvegarde (bug #3)

Versionner, valider, migrer, échapper. Voir [save-format.md](save-format.md#8-versionnage-recommandé).

### 6.6 Solutions robustes

Remplacer le plus possible `innerHTML` par `textContent` pour les données dynamiques, ou introduire une fonction `escapeHtml()` appliquée aux noms (joueurs, clubs, sponsors, messages).

## 7. Pistes d'amélioration

Par ordre de priorité suggéré :

1. **Fiabilité** — garde de journée, autosave `localStorage`, validation/version des sauvegardes, échappement HTML.
2. **Moteur de match** — buts modélisés (loi de Poisson à partir d'attaque/défense), prise en compte des postes et de la forme, fatigue des IA, suppression de l'upset arbitraire, buteurs.
3. **Économie** — salaires et charges hebdomadaires, rendements décroissants de l'entraînement, limites d'âge/potentiel, cohérence des prix de vente.
4. **Intelligence artificielle** — recrutement et régénération des effectifs, entraînement des clubs IA, tactiques simples.
5. **Calendrier et compétitions** — coupe, calendrier corrigé et varié.
6. **UX et accessibilité** — confirmation des actions, navigation clavier, états vides, responsive/tactile, mode mobile.
7. **Qualité** — tests, lint, CI, documentation à jour.

## 8. Tests suggérés

Les fonctions pures sont testables sans navigateur si on les isole :

| Invariant | Cible |
|---|---|
| 34 journées, 153 paires distinctes ×2, 17 domiciles par club, 18 équipes par journée | `generateFixtures()` |
| Distribution des buts plausible et bornée ; pas d'inversion de score incohérente | `simulateMatch()` |
| Force effective entre 56 % et 100 % de la force brute | `calculateTeamStrength()` |
| Valeur cohérente selon l'âge et la force | `calculatePlayerValue()` |
| Affluence jamais supérieure à la capacité, revenu croissant puis décroissant | `calculateAttendance()` |
| Un cycle complet `save → JSON.parse → load` conserve l'état | `saveGame()` / `loadGame()` |

Exemple de test (Node.js, après extraction de la fonction) :

```js
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { buildFixtures } from '../js/fixtures.js';

test('calendrier : 153 affiches jouées deux fois', () => {
    const fixtures = buildFixtures(18);
    assert.equal(fixtures.length, 34);

    const counts = new Map();
    for (const round of fixtures) {
        assert.equal(new Set(round.flatMap(f => [f.home, f.away])).size, 18);
        for (const f of round) {
            const key = [f.home, f.away].sort((a, b) => a - b).join('-');
            counts.set(key, (counts.get(key) ?? 0) + 1);
        }
    }
    assert.equal(counts.size, 153);
    assert.ok([...counts.values()].every(n => n === 2));
});
```

> Cela suppose d'extraire les fonctions du modèle dans des modules importables (voir ci-dessous) — les fichiers actuels ne sont pas chargeables tels quels sous Node.js.

## 9. Refactor envisageable

Si le projet grossit, une migration progressive vers des modules ES est recommandée :

```text
src/
├── engine/
│   ├── fixtures.js      # buildFixtures, simulateMatch, simulateRound
│   ├── players.js       # génération, vieillissement, valeur (fonctions pures)
│   └── economy.js       # billetterie, sponsors, prize money
├── state/
│   └── store.js         # gameState + save/load (avec version)
├── ui/
│   ├── render.js
│   └── events.js
└── main.js
```

Principes :

- fonctions pures séparées de `gameState` (elles reçoivent leurs données en paramètres) ;
- un seul point d'écriture de l'état ;
- rendu découplé des calculs ;
- `index.html` avec `<script type="module" src="src/main.js"></script>`.

## 10. Déploiement

- Hébergement statique simple (GitHub Pages, Netlify, etc.). Point de déploiement actuel : <https://football.dzair.co>.
- Le suivi d'audience Plausible est configuré sur `index.html`. Pour un déploiement privé, retirez la balise `<script data-domain=…>`.
- Pour un fonctionnement 100 % hors-ligne, hébergez les polices localement au lieu de les charger depuis Google Fonts.

## 11. Checklist avant commit

- [ ] Le jeu se lance sans erreur dans la console.
- [ ] Le calendrier contient bien 34 journées cohérentes (après correction du bug #1).
- [ ] Une journée complète se déroule sans blocage (composition pleine).
- [ ] La sauvegarde puis le rechargement restituent la partie.
- [ ] Aucune donnée issue d'un fichier externe n'est injectée dans `innerHTML`.
- [ ] La documentation `doc/` est mise à jour si une mécanique change.
