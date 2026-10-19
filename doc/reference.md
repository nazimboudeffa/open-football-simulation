# 🔎 Référence du code

Index des fonctions et constantes, par fichier. Les signatures correspondent au code actuel.

## `js/data.js` — constantes

| Constante | Type | Contenu |
|---|---|---|
| `TEAM_NAMES` | `string[]` | 18 noms de clubs |
| `FIRST_NAMES` | `string[]` | 34 prénoms |
| `LAST_NAMES` | `string[]` | 30 noms |
| `SPONSOR_NAMES` | `string[]` | 24 marques |
| `SPONSOR_TYPES` | `string[]` | `["shirt","north","south","east","west"]` |
| `SPONSOR_BASE_VALUES` | `object` | montant annuel de base par emplacement |
| `TRAINING_CONFIG` | `object` | `{ coût, strengthGain, energyCost }` par type |
| `STADIUM_EXPANSIONS` | `object` | places ajoutées → coût |
| `POSITION_COUNTS` | `object` | `{ GK: 2, DEF: 5, MID: 5, FWD: 4 }` |
| `PROMOTION_TEAMS` | `string[]` | 21 noms de clubs promus |
| `RANDOM_EVENTS` | `object[]` | 24 événements `{ type, title, message, effect }` |

## `js/state.js` — état et persistance

| Fonction | Rôle |
|---|---|
| `createInitialState()` | Renvoie l'état initial complet (budget 5 M€, stade 15 000 places, prix 25 €, etc.). |
| `resetGameState()` | Réaffecte `gameState` à un état initial neuf. |
| `getPlayerTeam()` | Renvoie `gameState.teams[gameState.playerTeamIndex]`. |
| `getLeaguePosition(teamIndex)` | Position au classement (points, différence de buts, buts marqués). |
| `saveGame()` | Sérialise `gameState` et télécharge un fichier JSON. |
| `loadGame(file)` | Lit un fichier JSON, remplace `gameState`, masque l'intro et rafraîchit l'UI. |

Variable globale : `gameState`.

## `js/players.js` — joueurs et équipes

| Fonction | Rôle |
|---|---|
| `generateId()` | Identifiant aléatoire de 9 caractères. |
| `calculatePlayerValue(player)` | Valeur = `force × 50 000 × facteur d'âge`. |
| `generatePlayer(position)` | Crée un joueur (18–34 ans, force 40–79, énergie 85–99, condition 80–99). |
| `generateSquad()` | Crée un effectif de 16 joueurs selon `POSITION_COUNTS`. |
| `generateTeams()` | Crée les 18 clubs avec noms mélangés et effectifs générés. |
| `boostPlayerTeam()` | Marque le club joueur et lui donne +10 de force, énergie 90–99. |
| `updateAllPlayerValues()` | Recalcule la valeur de tous les joueurs. |
| `ageAllPlayers()` | +1 an, retraites (> 34 ans), baisse de force (> 32 ans), retire les retraités. |
| `refreshAllPlayers()` | Réinitialise énergie/condition et les drapeaux d'entraînement. |
| `generateYouthPlayer(position)` | Crée un jeune (16–18 ans, force 25–49, potentiel aléatoire, `isYouth`). |
| `tryGenerateYouthPlayer()` | Ajoute un jeune si l'effectif compte moins de 25 joueurs ; sinon renvoie `null`. |

## `js/match.js` — calendrier, simulation, saison

| Fonction | Rôle |
|---|---|
| `generateFixtures()` | Génère 34 journées de 9 matchs. ⚠️ Algorithme bugué (affiches répétées). |
| `calculateTeamStrength(team, isPlayerTeam)` | Force moyenne des 11 joueurs, pondérée par énergie et condition. |
| `simulateMatch(home, away, homeIsPlayer, awayIsPlayer)` | Renvoie `{ homeGoals, awayGoals }`. |
| `processRoundMatches()` | Simule la journée, met à jour les statistiques et renvoie `{ playerMatch, results }`. |
| `processPlayerTeamRound(playerMatch)` | Revenus, entraînement, fatigue, blessures, progression, jeunes, événements. |
| `triggerRandomEvent()` | Applique un événement aléatoire de `RANDOM_EVENTS`. |
| `processEndOfSeason()` | Classement final, relégations, prize money, vieillissement, remise à zéro, nouvelle saison. |
| `isSeasonOver()` | `true` si `gameState.round > 34`. |

## `js/lineup.js` — composition

| Fonction | Rôle |
|---|---|
| `toggleLineup(playerId)` | Ajoute/retire un joueur (maximum 11). |
| `calculatePlayerRating(player)` | Note = `force × facteur énergie × facteur condition`. |
| `autoSelectLineup()` | Meilleur XI (1-4-4-2 par note). |
| `autoSelectFreshXI()` | XI le plus frais (énergie d'abord). |
| `isLineupValid()` | Vérifie que 11 joueurs sont alignés. |
| `getLineupPlayers()` | Renvoie les objets joueurs alignés. |

## `js/training.js` — entraînement

| Fonction | Rôle |
|---|---|
| `selectTrainingType(type)` | Mémorise le stage choisi (`basic`, `intensive`, `elite`). |
| `togglePlayerTraining(playerId)` | Ajoute/retire un joueur de la file d'entraînement. |
| `calculateTrainingCost()` | Coût total = coût unitaire × nombre de joueurs. |
| `updateTrainingCostDisplay()` | Met à jour l'affichage et active/désactive le bouton. |
| `sendToTraining()` | Débite le budget et marque les joueurs (`inTraining`, bonus, coût énergétique). |
| `processTrainingEffects()` | Applique gains et coûts énergétiques en fin de journée, puis réinitialise. |

## `js/transfers.js` — marché

| Fonction | Rôle |
|---|---|
| `generateTransferMarket()` | Génère 10 joueurs avec `askingPrice` (110–149 % de la valeur). |
| `buyPlayer(playerId)` | Débite le budget, ajoute le joueur, retire du marché. |
| `sellPlayer(playerId)` | Vend 70–89 % de la valeur ; refuse sous 14 joueurs ; nettoie la composition. |
| `refreshMarket()` | Régénère le marché pour 100 000 €. |

## `js/sponsors.js` — sponsoring

| Fonction | Rôle |
|---|---|
| `getSponsorTypeName(type)` | Libellé affichable d'un emplacement. |
| `generateSponsorOffers()` | Génère 6 offres (`amount` = base → base × 1,5 ; durée 1–3 saisons). |
| `acceptSponsor(sponsorId)` | Remplit l'emplacement, verse 25 % de bonus, fixe `roundsRemaining`. |
| `processSponsorPayments()` | Verse `amount / 34` par emplacement et gère l'expiration. |
| `calculateSponsorIncome()` | Total des paiements par journée, sans décrémenter. |

## `js/stadium.js` — stade et billetterie

| Fonction | Rôle |
|---|---|
| `calculateExpectedAttendance()` | Affluence = `capacité × max(0.4, 1 − (prix − 15) × 0.008)`. |
| `calculateProjectedRevenue()` | Affluence × prix. |
| `setTicketPrice(price)` | Refuse hors de la plage 10–100 €. |
| `expandStadium(seats)` | Débite le coût de `STADIUM_EXPANSIONS` et augmente la capacité. |
| `updateStadiumProjections()` | Met à jour capacité, affluence et revenu affichés. |

## `js/graphics.js` — rendu SVG

| Fonction | Rôle |
|---|---|
| `getContrastColor(hex)` | Renvoie `#000000` ou `#ffffff` selon la luminance. |
| `renderKitDisplay()` | Dessine les maillots domicile/extérieur avec le nom du sponsor maillot. |
| `updateStadiumGraphics()` | Affiche le sponsor dans chaque tribune (vues `stand-*` et `stand2-*`). |
| `renderIncomeBreakdown()` | Liste les revenus par sponsor + billetterie estimée. |

## `js/ui.js` — interface

| Fonction | Rôle |
|---|---|
| `showNotification(message, isError)` | Affiche une notification, journalise dans `messageLog` (100 max). |
| `updateMessageTicker()` | Rendu du fil d'actualité (via `innerHTML`). |
| `makeTableSortable(tableId, dataFetcher, renderer)` | Déclare une table triable. |
| `sortTable(tableId, columnKey, columnIndex)` | Change la colonne/le sens de tri et relance le rendu. |
| `getSortedData(tableId, data, columns)` | Applique le tri courant. |
| `updateHeaderStats()` | Club, saison, journée, budget, dernier revenu, position. |
| `updateSquadTable()` | Tableau de l'effectif (triable). |
| `updateLineupPanel()` | Cases à cocher de composition + notes. |
| `updateTransferMarket()` | Acheteurs et vendeurs. |
| `updateTrainingPanel()` | Cases à cocher d'entraînement + coût. |
| `updateSponsorsPanel()` | Contrats actifs et offres disponibles. |
| `updateLeagueTable()` | Classement (triable). |
| `updateFixturesPanel()` | Journées autour de la journée courante. |
| `updateAllUI()` | Appelle toutes les fonctions de rendu ci-dessus. |

## `js/main.js` — contrôleur

| Fonction | Rôle |
|---|---|
| `initGame()` | Point d'entrée : génère les équipes, remplit l'écran d'accueil, branche les événements. |
| `populateTeamSelect()` | Construit la grille de choix du club. |
| `selectTeam(index)` | Sélectionne un club et active le bouton de démarrage. |
| `startGame()` | Applique le kit et le bonus, génère calendrier/sponsors/marché, affiche le jeu. |
| `endRound()` | Valide la composition, simule la journée, affiche la modale de résultat. |
| `showSeasonEndModal()` | Calcule et affiche le bilan de fin de saison. |
| `createSeasonEndModal()` | Crée la modale de fin de saison si elle n'existe pas. |
| `closeSeasonEndModal()` | Ferme la modale, relance une sélection de XI. |
| `closeMatchModal()` | Ferme la modale de match, rafraîchit l'UI, enchaîne sur la fin de saison si besoin. |
| `setupEventListeners()` | Branche boutons, onglets, curseur de prix et inputs fichiers. |

## Glossaire

| Terme | Signification |
|---|---|
| Round | Journée de championnat |
| Season | Saison (34 journées) |
| Lineup | Composition des 11 titulaires |
| Shirt sponsor | Sponsor maillot |
| Stand | Tribune (north, south, east, west) |
| Condition | Fraîcheur/integrité physique, influe sur la force effective et le risque de blessure |
| Energy | Endurance, se réduit à chaque match, remonte au repos |
