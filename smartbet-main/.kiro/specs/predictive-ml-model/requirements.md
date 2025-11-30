# Requirements Document

## Introduction

Ce projet vise à développer un système de modélisation prédictive basé sur le machine learning pour prédire les résultats de matchs sportifs (Football, NBA, NHL, Tennis). Le système exploitera les données historiques enrichies (xG, ELO, cotes) pour générer des probabilités calibrées et identifier les opportunités de value betting. L'objectif est de surpasser les probabilités implicites des bookmakers en utilisant des features avancées comme les Expected Goals (xG).

## Glossary

- **Predictive Model**: Modèle ML entraîné pour prédire les résultats de matchs sportifs
- **Feature Engineering**: Processus de création de variables prédictives à partir des données brutes
- **Calibration**: Ajustement des probabilités prédites pour qu'elles correspondent aux fréquences observées
- **Value Bet**: Pari où la probabilité estimée par le modèle dépasse la probabilité implicite des cotes
- **Backtesting**: Simulation de paris sur données historiques pour évaluer la performance
- **ROI (Return on Investment)**: Rendement des paris = (gains - mises) / mises
- **Brier Score**: Métrique de calibration = moyenne de (probabilité prédite - résultat réel)²
- **Log Loss**: Métrique de qualité des probabilités = -moyenne(y*log(p) + (1-y)*log(1-p))
- **xG_diff**: Différence entre xG domicile et xG extérieur, indicateur de dominance attendue
- **Rolling Statistics**: Statistiques calculées sur une fenêtre glissante de N matchs précédents
- **Train/Test Split**: Séparation temporelle des données pour éviter le data leakage

## Requirements

### Requirement 1: Feature Engineering Pipeline

**User Story:** As a data scientist, I want to automatically generate predictive features from raw match data, so that I can train accurate prediction models.

#### Acceptance Criteria

1. WHEN processing a match THEN the System SHALL calculate rolling statistics (last 5, 10, 20 games) for each team including win rate, average goals scored, average goals conceded
2. WHEN processing a match THEN the System SHALL compute xG-based features including xG_diff, xG rolling averages, and xG over/under-performance (actual goals - xG)
3. WHEN processing a match THEN the System SHALL extract ELO-based features including ELO difference, ELO momentum (change over last 5 games), and relative ELO rank
4. WHEN processing a match THEN the System SHALL compute odds-derived features including implied probabilities, bookmaker margin, and odds movement indicators
5. WHEN processing a match THEN the System SHALL generate contextual features including home/away indicator, days since last match, and season progress percentage
6. WHEN a team has insufficient history (less than 5 games) THEN the System SHALL use league averages as fallback values

### Requirement 2: Model Training Pipeline

**User Story:** As a data scientist, I want to train and compare multiple ML models, so that I can select the best performing approach for each sport.

#### Acceptance Criteria

1. WHEN training models THEN the System SHALL implement at least three algorithms: Logistic Regression (baseline), XGBoost, and LightGBM
2. WHEN splitting data THEN the System SHALL use temporal split (train on older data, test on newer) to prevent data leakage
3. WHEN training THEN the System SHALL perform hyperparameter tuning using time-series cross-validation with at least 5 folds
4. WHEN training is complete THEN the System SHALL save the trained model with metadata including training date, features used, and performance metrics
5. WHEN training THEN the System SHALL handle class imbalance using appropriate techniques (class weights or SMOTE for draws in football)
6. WHEN training THEN the System SHALL train separate models for each sport to capture sport-specific patterns

### Requirement 3: Probability Calibration

**User Story:** As a bettor, I want well-calibrated probability predictions, so that I can make informed betting decisions.

#### Acceptance Criteria

1. WHEN generating predictions THEN the System SHALL output calibrated probabilities that sum to 1.0 for all outcomes
2. WHEN calibrating THEN the System SHALL apply isotonic regression or Platt scaling to raw model outputs
3. WHEN evaluating calibration THEN the System SHALL compute Brier score with target below 0.20 for binary outcomes
4. WHEN evaluating calibration THEN the System SHALL generate reliability diagrams showing predicted vs actual frequencies
5. WHEN predictions deviate significantly from calibration THEN the System SHALL log a warning with the match details

### Requirement 4: Backtesting Engine

**User Story:** As a bettor, I want to simulate betting strategies on historical data, so that I can evaluate model profitability before risking real money.

#### Acceptance Criteria

1. WHEN backtesting THEN the System SHALL simulate flat betting (fixed stake) on all value bets identified
2. WHEN backtesting THEN the System SHALL calculate ROI, win rate, and profit/loss over the test period
3. WHEN backtesting THEN the System SHALL support configurable value threshold (minimum edge over implied probability)
4. WHEN backtesting THEN the System SHALL track performance by sport, league, and time period
5. WHEN backtesting THEN the System SHALL generate equity curves showing cumulative profit over time
6. WHEN backtesting THEN the System SHALL compute statistical significance of results using bootstrap confidence intervals

### Requirement 5: Model Evaluation and Comparison

**User Story:** As a data scientist, I want comprehensive model evaluation metrics, so that I can compare models and track improvements.

#### Acceptance Criteria

1. WHEN evaluating THEN the System SHALL compute accuracy, precision, recall, and F1-score for each outcome class
2. WHEN evaluating THEN the System SHALL compute log loss as the primary metric for probability quality
3. WHEN evaluating THEN the System SHALL compare model predictions against bookmaker implied probabilities as baseline
4. WHEN evaluating THEN the System SHALL generate confusion matrices for each sport
5. WHEN evaluating THEN the System SHALL track feature importance rankings for tree-based models
6. WHEN comparing models THEN the System SHALL output a summary table with all metrics side by side

### Requirement 6: Prediction API

**User Story:** As a developer, I want to generate predictions for new matches, so that I can integrate the model into the Streamlit application.

#### Acceptance Criteria

1. WHEN predicting THEN the System SHALL accept match details (home team, away team, sport, date) and return probabilities for all outcomes
2. WHEN predicting THEN the System SHALL return confidence intervals for each probability estimate
3. WHEN predicting THEN the System SHALL flag value bets when model probability exceeds implied probability by configurable threshold
4. WHEN predicting THEN the System SHALL include feature values used for the prediction for transparency
5. WHEN the model file is missing or corrupted THEN the System SHALL raise a clear error with recovery instructions

### Requirement 7: Multi-Sport Support

**User Story:** As a user, I want predictions for all four sports, so that I can diversify my betting portfolio.

#### Acceptance Criteria

1. WHEN processing Football THEN the System SHALL predict three outcomes (home win, draw, away win) using multinomial classification
2. WHEN processing NBA, NHL, or Tennis THEN the System SHALL predict two outcomes (home win, away win) using binary classification
3. WHEN processing Tennis THEN the System SHALL include surface type as a categorical feature
4. WHEN processing NHL THEN the System SHALL handle overtime/shootout outcomes appropriately
5. WHEN generating features THEN the System SHALL use sport-specific xG ranges and normalization

