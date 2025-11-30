# Requirements Document - Ghost Bet Pan-Europe Strategy v3.0

## Introduction

La stratégie Ghost Bet Pan-Europe exploite un biais systématique du modèle xG calibré : quand le modèle prédit une victoire écrasante (≥66.5%) pour l'équipe visiteuse dans un match à faible xG total, avec une équipe à domicile en forme (goal diff +1.2) face à un top 6, le marché sous-évalue massivement le nul.

Cette version v3.0 remplace définitivement l'ancienne stratégie basée sur les rest_days (inutilisable sans matchs de coupe dans le dataset). Les nouveaux filtres xG + classement + goal diff permettent un ROI de +68% sur les 5 grandes ligues européennes.

**Objectif ROI long terme:** +60% à +75%
**Volume cible:** 25-35 paris par saison complète
**Ligues autorisées:** E0, SP1, I1, D1, F1 (aucune ligue virée)

## Glossary

- **Ghost_Bet_System**: Le système de détection et de backtest de la stratégie "Pari du Fantôme" Pan-Europe
- **prob_final_away**: Probabilité de victoire AWAY calculée par le modèle xG calibré
- **odds_draw**: Cote closing du match nul chez Pinnacle/Betfair
- **xG_home**: Expected Goals de l'équipe à domicile pour ce match
- **xG_away**: Expected Goals de l'équipe visiteuse pour ce match
- **home_goal_diff_last5**: Goal difference moyenne des 5 derniers matchs de l'équipe à domicile
- **away_league_rank**: Classement actuel de l'équipe AWAY dans la ligue (à J-1)
- **Top5_Leagues**: EPL (E0), La Liga (SP1), Serie A (I1), Bundesliga (D1), Ligue 1 (F1)
- **Strike_Rate**: Pourcentage de paris gagnants
- **Flat_Stake**: Mise fixe de 1% de la bankroll par pari

## Requirements

### Requirement 1

**User Story:** As a bettor, I want to identify draw betting opportunities when my model is overconfident on away favorites in low-xG matches, so that I can exploit the model's systematic bias.

#### Acceptance Criteria

1. WHEN the model calculates outcome probabilities THEN the Ghost_Bet_System SHALL identify matches where prob_final_away ≥ 0.665
2. WHEN filtering for draw bets THEN the Ghost_Bet_System SHALL only select matches where odds_draw ≥ 5.05
3. WHEN filtering for low-scoring matches THEN the Ghost_Bet_System SHALL only select matches where (xG_home + xG_away) ≤ 2.78
4. WHEN a match passes all filters THEN the Ghost_Bet_System SHALL flag it as a Ghost Bet opportunity

### Requirement 2

**User Story:** As a bettor, I want to filter matches by home team recent form, so that I can target situations where a strong home team might underperform against expectations.

#### Acceptance Criteria

1. WHEN calculating home team form THEN the Ghost_Bet_System SHALL use the goal difference from the last 5 matches
2. WHEN filtering for home team form THEN the Ghost_Bet_System SHALL only select matches where home_goal_diff_last5 ≥ 1.20
3. WHEN home_goal_diff_last5 data is unavailable THEN the Ghost_Bet_System SHALL exclude the match from consideration

### Requirement 3

**User Story:** As a bettor, I want to filter matches by away team league position, so that I can target matches where top teams visit.

#### Acceptance Criteria

1. WHEN filtering by away team rank THEN the Ghost_Bet_System SHALL only select matches where away_league_rank ≤ 6
2. WHEN away_league_rank data is unavailable THEN the Ghost_Bet_System SHALL exclude the match from consideration
3. WHEN the away team rank changes during the season THEN the Ghost_Bet_System SHALL use the rank at J-1 (day before match)

### Requirement 4

**User Story:** As a bettor, I want to restrict bets to the top 5 European leagues in regular season, so that I can ensure consistent market efficiency and data quality.

#### Acceptance Criteria

1. WHEN filtering by league THEN the Ghost_Bet_System SHALL only include matches from E0, SP1, I1, D1, and F1
2. WHEN filtering by competition type THEN the Ghost_Bet_System SHALL exclude cup matches, playoffs, and international competitions
3. WHEN a match league code is provided THEN the Ghost_Bet_System SHALL validate it against the allowed list

### Requirement 5

**User Story:** As a bettor, I want to backtest the Ghost Bet Pan-Europe strategy on historical data, so that I can validate its profitability before live deployment.

#### Acceptance Criteria

1. WHEN running a backtest THEN the Ghost_Bet_System SHALL simulate flat 1% bankroll stakes on all qualifying bets
2. WHEN calculating results THEN the Ghost_Bet_System SHALL compute ROI, strike rate, total profit, and max drawdown
3. WHEN generating equity curve THEN the Ghost_Bet_System SHALL track cumulative profit over time
4. WHEN analyzing performance THEN the Ghost_Bet_System SHALL break down results by league and season

### Requirement 6

**User Story:** As a bettor, I want to generate daily Ghost Bet recommendations, so that I can act on opportunities in real-time.

#### Acceptance Criteria

1. WHEN scanning upcoming matches THEN the Ghost_Bet_System SHALL apply all 6 Ghost Bet filters
2. WHEN a match qualifies THEN the Ghost_Bet_System SHALL output match details, draw odds, model probabilities, xG values, home_goal_diff_last5, and away_league_rank
3. WHEN no matches qualify THEN the Ghost_Bet_System SHALL report zero opportunities without error

### Requirement 7

**User Story:** As a developer, I want the Ghost Bet logic to be testable with property-based tests, so that I can ensure correctness across all edge cases.

#### Acceptance Criteria

1. WHEN filtering matches THEN the Ghost_Bet_System SHALL produce deterministic results for identical inputs
2. WHEN calculating metrics THEN the Ghost_Bet_System SHALL handle edge cases (empty datasets, missing columns)
3. WHEN validating league codes THEN the Ghost_Bet_System SHALL accept only the 5 allowed codes

