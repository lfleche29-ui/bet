# Requirements Document

## Introduction

Ce projet vise à transformer l'application Streamlit existante (prototype v2.3) en un outil de prédiction live professionnel et complet. L'application intégrera le modèle ML entraîné (LightGBM + calibration), les générateurs xG, et le système de backtesting pour fournir des prédictions calibrées en temps réel avec identification automatique des value bets.

L'objectif est de créer une interface utilisateur intuitive permettant aux utilisateurs de visualiser les matchs du jour, les probabilités calibrées du modèle, les xG récents des équipes, et les opportunités de value betting détectées automatiquement.

## Glossary

- **Streamlit App**: Application web Python utilisant le framework Streamlit pour l'interface utilisateur
- **Value Bet**: Pari où la probabilité estimée par le modèle dépasse la probabilité implicite des cotes + marge configurée
- **Calibrated Probability**: Probabilité ajustée via isotonic regression ou Platt scaling pour correspondre aux fréquences observées
- **Fair Odds**: Cotes calculées à partir des probabilités du modèle (1/probabilité)
- **Edge**: Différence entre la probabilité du modèle et la probabilité implicite des cotes
- **xG (Expected Goals)**: Métrique de performance attendue générée par les générateurs xG
- **Model Loader**: Composant qui charge automatiquement le dernier modèle entraîné par sport
- **Kelly Fraction**: Fraction de bankroll recommandée basée sur le critère de Kelly
- **Equity Curve**: Courbe de profit cumulé du modèle sur les données historiques

## Requirements

### Requirement 1: Architecture et Structure du Projet

**User Story:** As a developer, I want a clean project structure with reusable components, so that the codebase is maintainable and extensible.

#### Acceptance Criteria

1. WHEN the application starts THEN the System SHALL load components from organized directories (pages/, components/, utils/)
2. WHEN importing modules THEN the System SHALL use relative imports from a well-defined package structure
3. WHEN adding new features THEN the System SHALL allow extension without modifying core components
4. WHEN loading configuration THEN the System SHALL read settings from a centralized config file

### Requirement 2: Model Loading et Gestion

**User Story:** As a user, I want the app to automatically load the latest trained model for each sport, so that I always get predictions from the most recent model.

#### Acceptance Criteria

1. WHEN the application initializes THEN the System SHALL detect and load the most recent model file for each sport from models/trained/
2. WHEN a model file is missing for a sport THEN the System SHALL display a clear warning and disable predictions for that sport
3. WHEN loading a model THEN the System SHALL validate model metadata (training date, features, metrics)
4. WHEN multiple model versions exist THEN the System SHALL select the one with the most recent training date

### Requirement 3: Page Live Predictions

**User Story:** As a bettor, I want to see today's and tomorrow's matches with model predictions, so that I can make informed betting decisions.

#### Acceptance Criteria

1. WHEN displaying the Live Predictions page THEN the System SHALL show matches for today (J) and tomorrow (J+1)
2. WHEN displaying a match THEN the System SHALL show calibrated probabilities for all outcomes (home, draw for football, away)
3. WHEN displaying a match THEN the System SHALL show fair odds calculated from model probabilities
4. WHEN displaying a match THEN the System SHALL show the edge (model prob - implied prob) for each outcome
5. WHEN a value bet is detected THEN the System SHALL highlight the row in green with the edge percentage

### Requirement 4: Affichage des Probabilités et Fair Odds

**User Story:** As a bettor, I want to see both model probabilities and fair odds, so that I can compare with bookmaker odds.

#### Acceptance Criteria

1. WHEN displaying probabilities THEN the System SHALL show calibrated probabilities as percentages with one decimal
2. WHEN displaying fair odds THEN the System SHALL calculate fair odds as 1/probability for each outcome
3. WHEN probabilities sum differs from 1.0 THEN the System SHALL normalize before displaying
4. WHEN displaying confidence THEN the System SHALL show confidence intervals for each probability

### Requirement 5: Value Bet Detection et Highlighting

**User Story:** As a bettor, I want value bets to be automatically highlighted, so that I can quickly identify profitable opportunities.

#### Acceptance Criteria

1. WHEN model probability exceeds implied probability by configurable threshold THEN the System SHALL mark the outcome as a value bet
2. WHEN displaying a value bet THEN the System SHALL highlight the cell with green background and show edge percentage
3. WHEN filtering predictions THEN the System SHALL allow filtering by minimum edge percentage
4. WHEN a value bet is detected THEN the System SHALL calculate recommended Kelly fraction

### Requirement 6: Graphiques xG et Forme des Équipes

**User Story:** As an analyst, I want to see xG trends and team form, so that I can understand the context behind predictions.

#### Acceptance Criteria

1. WHEN displaying team details THEN the System SHALL show rolling xG averages (last 5, 10 games)
2. WHEN displaying team details THEN the System SHALL show xG over/under-performance (actual - xG)
3. WHEN displaying team form THEN the System SHALL show win/draw/loss streak and recent results
4. WHEN displaying xG trends THEN the System SHALL use line charts with Plotly

### Requirement 7: Comparaison Modèle vs Bookmakers

**User Story:** As a bettor, I want to compare model predictions with multiple bookmakers, so that I can find the best value.

#### Acceptance Criteria

1. WHEN displaying odds comparison THEN the System SHALL show Pinnacle odds as reference
2. WHEN displaying odds comparison THEN the System SHALL show model fair odds alongside bookmaker odds
3. WHEN displaying odds comparison THEN the System SHALL calculate edge against each bookmaker
4. WHEN displaying odds comparison THEN the System SHALL highlight the best value opportunity

### Requirement 8: Backtest Summary Intégré

**User Story:** As a user, I want to see the model's historical performance, so that I can trust its predictions.

#### Acceptance Criteria

1. WHEN displaying performance THEN the System SHALL show historical ROI from backtesting
2. WHEN displaying performance THEN the System SHALL show win rate and total bets
3. WHEN displaying performance THEN the System SHALL show equity curve as a line chart
4. WHEN displaying performance THEN the System SHALL show performance breakdown by sport and league

### Requirement 9: Filtres Avancés

**User Story:** As a user, I want advanced filtering options, so that I can focus on specific opportunities.

#### Acceptance Criteria

1. WHEN filtering THEN the System SHALL allow filtering by sport (Football, NBA, NHL, Tennis)
2. WHEN filtering THEN the System SHALL allow filtering by league/competition
3. WHEN filtering THEN the System SHALL allow filtering by minimum edge percentage
4. WHEN filtering THEN the System SHALL allow filtering by minimum/maximum odds range
5. WHEN filtering THEN the System SHALL allow sorting by edge, probability, or odds

### Requirement 10: Export des Value Bets

**User Story:** As a user, I want to export today's value bets, so that I can track them externally.

#### Acceptance Criteria

1. WHEN exporting THEN the System SHALL generate a CSV file with all value bets
2. WHEN exporting THEN the System SHALL include match details, probabilities, odds, and edge
3. WHEN exporting THEN the System SHALL name the file with current date
4. WHEN exporting THEN the System SHALL provide a download button in the UI

### Requirement 11: Design et UX Professionnels

**User Story:** As a user, I want a professional and clean interface, so that the app is pleasant to use.

#### Acceptance Criteria

1. WHEN displaying the app THEN the System SHALL use a consistent color scheme and typography
2. WHEN displaying the app THEN the System SHALL support dark and light mode toggle
3. WHEN displaying data THEN the System SHALL use clear visual hierarchy with proper spacing
4. WHEN displaying value bets THEN the System SHALL use intuitive color coding (green=value, red=avoid)

### Requirement 12: Performance et Cache

**User Story:** As a user, I want the app to load quickly, so that I can access predictions without delay.

#### Acceptance Criteria

1. WHEN loading data THEN the System SHALL use Streamlit caching for expensive operations
2. WHEN loading models THEN the System SHALL cache loaded models in session state
3. WHEN refreshing predictions THEN the System SHALL only reload changed data
4. WHEN displaying large tables THEN the System SHALL use pagination or virtual scrolling

### Requirement 13: Tests d'Intégration

**User Story:** As a developer, I want automated tests for the Streamlit app, so that I can ensure quality.

#### Acceptance Criteria

1. WHEN testing THEN the System SHALL verify that all pages load without errors
2. WHEN testing THEN the System SHALL verify that model loading works correctly
3. WHEN testing THEN the System SHALL verify that predictions are generated correctly
4. WHEN testing THEN the System SHALL verify that exports produce valid files

### Requirement 14: Documentation et README

**User Story:** As a new user, I want clear documentation, so that I can understand how to use the app.

#### Acceptance Criteria

1. WHEN reading documentation THEN the User SHALL find installation instructions
2. WHEN reading documentation THEN the User SHALL find usage guide with screenshots
3. WHEN reading documentation THEN the User SHALL find configuration options explained
4. WHEN reading documentation THEN the User SHALL find troubleshooting section
