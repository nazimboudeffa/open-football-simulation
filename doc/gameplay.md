# 🎮 Règles et systèmes de jeu

Ce document décrit le comportement réel du code, formules comprises. Les valeurs entre parenthèses renvoient aux fonctions concernées.

## 1. Objectif et boucle de jeu

Le joueur dirige un club dans un championnat de 18 équipes. Chaque clic sur **END ROUND** joue les 9 matchs de la journée. Entre deux journées, le joueur peut :

- composer son onze (`lineup.js`) ;
- recruter ou vendre (`transfers.js`) ;
- envoyer des joueurs en stage (`training.js`) ;
- négocier des sponsors (`sponsors.js`) ;
- fixer le prix des billets et agrandir le stade (`stadium.js`).

À la fin des 34 journées, la saison se clôture, les joueurs vieillissent, le club est récompensé selon son classement et une nouvelle saison commence.

## 2. Championnat et calendrier

- 18 clubs, 34 journées, 9 matchs par journée.
- Classement : points, puis différence de buts, puis buts marqués (`getLeaguePosition()`).
- Victoire 3 pts, nul 1 pt, défaite 0 pt.

⚠️ **Le générateur de calendrier est bugué** (`match.js → generateFixtures()`).

L'algorithme utilisé est :

```js
for (let round = 0; round < 34; round++) {
  for (let i = 0; i < 9; i++) {
    const home = (round + i) % 18;
    const away = (round + 17 - i) % 18;
    // …
  }
}
```

Il produit bien 18 équipes distinctes à chaque journée, mais les affiches se répètent : sur 153 rencontres possibles, **seules 81 existent** (18 paires jouées 3 fois, 63 paires jouées 4 fois) et **chaque club n'affronte que 9 adversaires différents sur toute la saison**. Le plan de saison est identique d'une saison à l'autre. Voir [development.md](development.md#bugs-et-limites-connus) pour le correctif proposé.

## 3. Effectifs et joueurs

À la création (`players.js → generateSquad()`), chaque équipe reçoit 16 joueurs :

| Poste | Nombre |
|---|---|
| GK | 2 |
| DEF | 5 |
| MID | 5 |
| FWD | 4 |

Caractéristiques générées :

| Attribut | Plage | Remarque |
|---|---|---|
| Âge | 18 – 34 ans | `18 + floor(random × 17)` |
| Force de base | 40 – 79 | modulée par l'âge : `× 1` si < 28, sinon `× (1 − (âge − 28) × 0.03)` |
| Énergie | 85 – 99 | maximum 100 |
| Condition | 80 – 99 | maximum 100 |
| Valeur | calculée | voir ci-dessous |

Valeur d'un joueur (`calculatePlayerValue()`) :

```text
valeur = floor(force × 50 000 × facteurÂge)

facteurÂge = 1.3  si âge < 26
           = 1.0  si 26 ≤ âge < 30
           = 0.7  si âge ≥ 30
```

Le club joueur démarre avec un bonus : **+10 de force** (plafonnée à 99), énergie 90–99, et le drapeau `isPlayer` (`players.js → boostPlayerTeam()`). Les budgets IA sont générés (3 à 10 M€) mais jamais utilisés.

## 4. Force d'équipe et moteur de match

### Force effective (`match.js → calculateTeamStrength()`)

- Club joueur : uniquement les 11 joueurs de `selectedLineup`.
- Clubs IA : les 11 meilleurs joueurs triés par force.
- Si moins de 11 joueurs sont disponibles, des joueurs fictifs sont ajoutés avec `force = moyenne / 2`, énergie 50, condition 50.

Pour chaque joueur retenu :

```text
forceEffective = force × (0.7 + énergie/100 × 0.3) × (0.8 + condition/100 × 0.2)
```

Les facteurs vont de 0,7 à 1,0 (énergie) et de 0,8 à 1,0 (condition), soit un minimum de 56 % de la force brute. La force d'équipe est la **moyenne** des 11 forces effectives.

### Buts (`match.js → simulateMatch()`)

```text
forceDomicile = forceÉquipeDomicile × 1.1        // avantage du terrain
p = forceDomicile / (forceDomicile + forceExtérieur)

chances = 8 + floor(random × 8)                  // 8 à 15 occasions
pour chaque occasion :
    r = random()
    si r < p × 0.28                    → but domicile
    sinon si r > 1 − (1 − p) × 0.28    → but extérieur
```

Chaque occasion a donc ~28 % de chance de produire un but, et 72 % de ne rien produire. Espérance moyenne avec deux équipes de force égale : environ **1,7 but à domicile et 1,5 à l'extérieur**.

Ensuite, **8 % des matchs** subissent un « upset » qui échange le score :

```js
homeGoals = max(0, awayGoals + floor(random × 2));
awayGoals = max(0, temp - floor(random × 2));   // temp = score domicile initial
```

⚠️ Cet échange peut produire des résultats incohérents (un 5-0 peut devenir une défaite 1-0 de l'équipe dominante, et un 0-0 peut se transformer en 1-0).

### Limites du moteur

- Aucune notion de poste, de tactique, de formation, de buteur ou de passeur.
- Les clubs IA alignent toujours leur meilleur XI et **ne se fatiguent jamais** : seul le club joueur voit son énergie évoluer (`match.js → processPlayerTeamRound()` n'itère que sur `getPlayerTeam()`).
- La force des IA ne dépend donc que des attributs statiques et des retraites.

## 5. Énergie, condition et blessures

Appliqué après chaque journée, uniquement au club joueur (`processPlayerTeamRound()`) :

| Situation | Effet |
|---|---|
| Titulaire | −8 à −14 d'énergie (plancher 25) |
| Remplaçant | +25 d'énergie (plafond 100), +10 de condition (plafond 100) |
| Tous, âge < 25 | +3 de condition (plafond 100) |
| Tous, 25–30 ans | +2 de condition |
| Tous, âge > 30 | +1 de condition |
| Titulaire | 20 % de chance de +1 de force (plafond 99) |
| Jeune titulaire | 30 % de chance de +1 de force jusqu'à son potentiel |

Blessure (`processPlayerTeamRound()`) :

```text
si condition < 50 et random() > condition / 80 :
    perte = 10 + floor(random × 20)     // 10 à 29 points de condition
    condition = max(20, condition − perte)
```

À 40 de condition, la probabilité de blessure par match est de 50 % ; à 49, elle est d'environ 39 %. Le message de blessure apparaît dans le fil d'actualité.

## 6. Composition (lineup)

- `toggleLineup(id)` ajoute/retire un joueur, avec un maximum de 11.
- Aucun contrôle de poste : aligner 11 attaquants est accepté.
- `autoSelectLineup()` (« BEST XI ») trie par **note** (`calculatePlayerRating()` = `force × facteurÉnergie × facteurCondition`) et tente de prendre 1 GK, 4 DEF, 4 MID, 2 FWD.
- `autoSelectFreshXI()` privilégie l'énergie (comparaison par tranches de 10), puis la force.
- `isLineupValid()` exige exactement 11 joueurs.

⚠️ Si l'effectif ne contient pas assez de joueurs aux postes visés, la sélection automatique peut rendre **moins de 11 joueurs**, ce qui bloque `endRound()` sans solution évidente.

## 7. Entraînement

Configuration (`data.js → TRAINING_CONFIG`) :

| Type | Gain de force | Coût énergétique | Prix par joueur |
|---|---|---|---|
| Basic | +3 | −10 | 20 000 € |
| Intensive | +6 | −20 | 60 000 € |
| Elite | +10 | −25 | 150 000 € |

Fonctionnement :

1. Le joueur sélectionne un type puis coche des joueurs (`togglePlayerTraining()`).
2. `sendToTraining()` vérifie le budget, débite le coût total et marque les joueurs (`inTraining`, `trainingBonus`, `trainingEnergyCost`).
3. À la fin de la journée, `processTrainingEffects()` applique le gain (plafond 99) et la perte d'énergie (plancher 30), puis remet les drapeaux à zéro.

⚠️ Un joueur déjà aligné **avant** d'être envoyé en stage reste dans le onze : il joue le match et bénéficie de l'entraînement la même journée.

⚠️ Le rendement est très favorable : +10 de force pour 150 000 €, alors que 10 points de force valent environ 500 000 € sur un joueur de moins de 26 ans. Répété chaque journée, c'est de la création de valeur nette.

## 8. Transferts

| Action | Règle |
|---|---|
| Taille du marché | 10 joueurs générés (`generateTransferMarket()`) |
| Prix demandé | `floor(valeur × (1.1 + random × 0.4))`, soit **110 % à 149 %** de la valeur |
| Achat | `buyPlayer()` débite le budget et retire le joueur du marché |
| Vente | `sellPlayer()` rapporte `floor(valeur × (0.7 + random × 0.2))`, soit **70 % à 89 %** |
| Taille minimale d'effectif | 14 joueurs |
| Rafraîchissement du marché | 100 000 €, régénère 10 joueurs |

⚠️ Le tableau de vente de l'interface affiche `floor(valeur × 0.8)`, un montant **fixe** qui ne correspond pas au prix réellement obtenu (aléatoire entre 70 % et 89 %).

## 9. Sponsors

Cinq emplacements : maillot + 4 tribunes (`shirt`, `north`, `south`, `east`, `west`).

Valeurs de base par saison (`SPONSOR_BASE_VALUES`) :

| Emplacement | Base |
|---|---|
| Maillot | 400 000 € |
| Nord / Sud | 150 000 € |
| Est / Ouest | 120 000 € |

Génération (`generateSponsorOffers()`) : 6 offres, montant = `base + floor(random × base × 0.5)` (donc 100 % à 150 % de la base), durée de 1 à 3 saisons. Les offres sont régénérées à chaque fin de saison et avec 15 % de chance par journée.

Signature (`acceptSponsor()`) :

- le sponsor remplit l'emplacement correspondant s'il est libre ;
- un **bonus de signature de 25 %** du montant annuel est versé immédiatement ;
- `roundsRemaining = durée × 34` ;
- à chaque journée, `processSponsorPayments()` verse `floor(montant / 34)` et décrémente le compteur ; à zéro le contrat s'arrête.

Sur la durée totale, un contrat rapporte donc `1,25 × montant × durée`, soit plus que le montant affiché.

⚠️ Plusieurs offres peuvent viser le même emplacement ; l'interface désactive alors le bouton d'acceptation.

## 10. Stade et billetterie

- Capacité initiale : 15 000 places. Prix initial : 25 €.
- Prix réglable de 10 € à 100 € (`setTicketPrice()`).
- Affluence (`calculateExpectedAttendance()`) :

```text
taux = max(0.4, 1 − (prix − 15) × 0.008)
affluence = floor(capacité × taux)
revenu = affluence × prix          // matchs à domicile uniquement
```

| Prix | Taux d'affluence | Affluence (15 000 places) | Revenu |
|---|---|---|---|
| 10 € | 104 % | 15 600 | 156 000 € |
| 25 € | 92 % | 13 800 | 345 000 € |
| 40 € | 80 % | 12 000 | 480 000 € |
| 70 € | 56 % | 8 400 | 588 000 € |
| 100 € | 40 % (plancher) | 6 000 | 600 000 € |

⚠️ À 10 €, l'affluence dépasse la capacité (le taux n'est pas plafonné à 100 %). ⚠️ Par ailleurs, le plancher de 40 % rend le prix maximal économiquement optimal : monter les prix rapporte toujours plus, ce qui supprime l'arbitrage promis par l'interface.

Extensions (`STADIUM_EXPANSIONS`) :

| Places ajoutées | Coût |
|---|---|
| 1 000 | 500 000 € |
| 2 500 | 1 100 000 € |
| 5 000 | 2 000 000 € |
| 10 000 | 3 500 000 € |

## 11. Jeunes, vieillissement et retraites

Génération de jeunes (`players.js`) :

- 10 % de chance à chaque journée paire, si l'effectif compte moins de 25 joueurs ;
- 40 % de chance supplémentaire à la journée 17 (mi-saison) ;
- âge 16–18 ans, force 25–49, potentiel = force + 20 à 49 ;
- postes pondérés vers le milieu et l'attaque.

Vieillissement annuel (`ageAllPlayers()`) :

- +1 an pour tous ;
- retraite si âge > 34, avec probabilité `(âge − 34) × 0.3` ;
- baisse de force de 0 à 2 points si âge > 32 ;
- les retraités sont retirés de l'effectif (notification pour le club joueur).

⚠️ Aucune équipe IA ne recrute ni ne régénère son effectif : au fil des saisons, leurs joueurs vieillissent, partent à la retraite, et les équipes s'affaiblissent. Seuls les clubs promus repartent avec 16 joueurs neufs. Le championnat devient donc de plus en plus facile pour le joueur.

## 12. Fin de saison et relégation

`processEndOfSeason()` :

| Position finale | Prize money |
|---|---|
| 1er | 5 000 000 € |
| 2e | 3 000 000 € |
| 3e | 2 000 000 € |
| 4e – 6e | 1 000 000 € |
| 7e – 10e | 500 000 € |
| 11e – 18e | 250 000 € |

- Les **2 derniers clubs IA** sont relégués et remplacés par des clubs promus générés aléatoirement (noms piochés dans `PROMOTION_TEAMS`).
- L'énergie et la condition de tous les joueurs sont réinitialisées, les statistiques remises à zéro, un nouveau calendrier, de nouvelles offres et un nouveau marché sont générés.

⚠️ La relégation du club joueur n'a **aucun effet** : `isPlayerRelegated` est calculé mais jamais utilisé. Un joueur peut terminer 18e et continuer en première division. Le club joueur n'est en outre jamais dans la liste des relégués.

⚠️ `ageAllPlayers()` puis `refreshAllPlayers()` : les retraites et la baisse de force des vétérans sont bien conservées, mais l'effectif peut descendre sous 11 joueurs sans recrutement, auquel cas le moteur de match complète avec des joueurs fictifs faibles.

## 13. Événements aléatoires

- 8 % de chance par journée (`triggerRandomEvent()`).
- 24 événements dans `RANDOM_EVENTS` : 8 positifs, 8 négatifs, 8 neutres.
- Effets possibles : budget (±10 000 € à 750 000 €), énergie de tout l'effectif, condition de tout l'effectif, +force sur un joueur aléatoire, ou aucun effet.
- L'événement est mémorisé dans `lastRandomEvent`, affiché dans la modale de fin de journée, puis effacé.

## 14. Synthèse économique

**Recettes du club joueur**

- Sponsors : jusqu'à 5 contrats simultanés, versés par journée (+25 % à la signature).
- Billetterie : uniquement les matchs à domicile (~17 par saison).
- Prize money en fin de saison.
- Ventes de joueurs.

**Dépenses**

- Entraînement (20 k€ à 150 k€ par joueur), transferts (110–149 % de la valeur), rafraîchissement du marché (100 k€), extension du stade (0,5 à 3,5 M€).

**Absent**

- Aucun salaire, aucune charge d'exploitation, aucune masse salariale.
- Les budgets des clubs IA ne servent à rien.

Résultat : l'argent s'accumule vite et les décisions économiques manquent de tension. Les pistes d'équilibrage sont listées dans [development.md](development.md#pistes-damélioration).

## 15. Résumé des constantes

| Constante | Valeur |
|---|---|
| Clubs | 18 |
| Journées | 34 |
| Joueurs par effectif | 16 (2 GK, 5 DEF, 5 MID, 4 FWD) |
| Effectif maximum | 25 (jeunes compris) |
| Effectif minimum | 14 (ventes) |
| Budget initial du joueur | 5 000 000 € |
| Avantage du terrain | ×1.1 |
| Upset | 8 % des matchs |
| Blessure | condition < 50 |
| Événement aléatoire | 8 % par journée |
| Offres de sponsors | 6 par génération |
| Marché des transferts | 10 joueurs |
