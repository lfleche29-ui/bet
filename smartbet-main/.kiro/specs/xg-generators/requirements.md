# Requirements Document

## Introduction

Ce projet vise à créer des générateurs xG (Expected Goals/Points/Games) state-of-the-art pour les 4 sports du dataset (Football, NBA, NHL, Tennis). Les colonnes xG_home et xG_away doivent être remplies à 100% avec des valeurs réalistes et hautement prédictives. La métrique xG devient une métrique universelle "xScore" représentant la quantité de points/buts/jeux qu'un participant "méritait" selon la qualité de ses actions offensives.

## Glossary

- **xG (Expected Goals)**: Métrique mesurant la probabilité qu'un tir devienne un but, basée sur la position, l'angle, le type de tir et le contexte
- **xPoints (NBA)**: Points attendus basés sur la localisation du tir, le type de tir, la défense et l'historique du tireur
- **xG Hockey (NHL)**: Expected Goals hockey basé sur la distance, l'angle, le type de tir, les rebonds et les situations de jeu
- **xGames (Tennis)**: Jeux attendus basés sur le % de points gagnés au service/retour, la surface, l'Elo et la fatigue
- **xG_diff**: Différentiel xG (xG_home - xG_away), feature hautement prédictive
- **Calibration**: Processus d'ajustement du modèle pour que les prédictions correspondent aux résultats réels
- **Shot Quality Model**: Modèle évaluant la qualité d'une occasion de marquer
- **Poisson Distribution**: Distribution statistique utilisée pour modéliser le nombre de buts/points
- **Surface Effect**: Impact de la surface de jeu (clay, grass, hard) sur les performances tennis
- **Home Advantage Factor**: Facteur d'avantage domicile intégré dans les calculs xG

## Requirements

### Requirement 1: Football xG Generator

**User Story:** As a data scientist, I want realistic xG values for all football matches, so that I can build highly predictive models using expected goals metrics.

#### Acceptance Criteria

1. WHEN generating xG for a football match THEN the System SHALL produce xG_home and xG_away values between 0.0 and 6.0 based on team offensive/defensive strength
2. WHEN calculating football xG THEN the System SHALL use team historical scoring rates, league averages, home advantage factor (1.15-1.25), and opponent defensive strength
3. WHEN xG values are generated THEN the System SHALL ensure correlation with actual goals scored exceeds 0.70 across the dataset
4. WHEN processing football matches THEN the System SHALL calculate xG_diff as (xG_home - xG_away) and store it as a separate column
5. WHEN generating xG for historical matches THEN the System SHALL use only data available before the match date to avoid data leakage

### Requirement 2: NBA xPoints Generator

**User Story:** As a data scientist, I want realistic expected points (xPoints) for all NBA games, so that I can predict game outcomes using shot quality metrics.

#### Acceptance Criteria

1. WHEN generating xPoints for an NBA game THEN the System SHALL produce xG_home and xG_away values representing expected points between 85.0 and 135.0
2. WHEN calculating NBA xPoints THEN the System SHALL use team offensive rating, defensive rating, pace factor, and home court advantage (2.5-4.0 points)
3. WHEN xPoints values are generated THEN the System SHALL ensure correlation with actual points scored exceeds 0.75 across the dataset
4. WHEN processing NBA games THEN the System SHALL calculate xG_diff as (xG_home - xG_away) representing expected point differential
5. WHEN generating xPoints for historical games THEN the System SHALL use rolling team statistics from previous games only

### Requirement 3: NHL xG Generator

**User Story:** As a data scientist, I want realistic hockey xG values for all NHL games, so that I can analyze expected goal performance versus actual results.

#### Acceptance Criteria

1. WHEN generating xG for an NHL game THEN the System SHALL produce xG_home and xG_away values between 1.5 and 5.0 based on shot quality models
2. WHEN calculating NHL xG THEN the System SHALL use team shot generation rate, shot quality metrics, goaltender save percentage, and home ice advantage (1.05-1.15 factor)
3. WHEN xG values are generated THEN the System SHALL ensure correlation with actual goals scored exceeds 0.65 across the dataset
4. WHEN processing NHL games THEN the System SHALL calculate xG_diff as (xG_home - xG_away)
5. WHEN generating xG for historical games THEN the System SHALL preserve existing xG values where present and only fill missing values

### Requirement 4: Tennis xGames Generator

**User Story:** As a data scientist, I want realistic expected games (xGames) for all tennis matches, so that I can predict match outcomes using service and return performance metrics.

#### Acceptance Criteria

1. WHEN generating xGames for a tennis match THEN the System SHALL produce xG_home and xG_away values representing expected games won (typically 0-25 range for best-of-3, 0-40 for best-of-5)
2. WHEN calculating tennis xGames THEN the System SHALL use player Elo ratings, surface-specific performance, head-to-head history, and fatigue factor (matches in last 7 days)
3. WHEN xGames values are generated THEN the System SHALL ensure correlation with actual games won exceeds 0.60 across the dataset
4. WHEN processing tennis matches THEN the System SHALL calculate xG_diff as (xG_home - xG_away)
5. WHEN generating xGames THEN the System SHALL adjust calculations based on surface type (clay increases variance, grass decreases variance)

### Requirement 5: Data Completeness and Validation

**User Story:** As a data engineer, I want 100% complete xG columns across all sports, so that the dataset has no missing values for machine learning.

#### Acceptance Criteria

1. WHEN the xG generation pipeline completes THEN the System SHALL have zero NULL values in xG_home and xG_away columns for all 4 sports
2. WHEN validating xG values THEN the System SHALL verify all values are within sport-specific valid ranges
3. WHEN generating xG values THEN the System SHALL log statistics including mean, std, min, max per sport and per season
4. WHEN xG generation fails for a match THEN the System SHALL use fallback calculation based on league averages and log the fallback usage
5. WHEN all xG values are generated THEN the System SHALL produce a validation report showing correlation with actual results per sport

### Requirement 6: xG_diff Feature Engineering

**User Story:** As a machine learning engineer, I want xG_diff as a pre-computed feature, so that I can use the most predictive xG metric directly in models.

#### Acceptance Criteria

1. WHEN xG values are computed THEN the System SHALL calculate xG_diff column as (xG_home - xG_away) for all matches
2. WHEN validating xG_diff THEN the System SHALL verify correlation with home_won outcome exceeds 0.50 for all sports
3. WHEN xG_diff is positive THEN the System SHALL expect home team to win more often than when xG_diff is negative
4. WHEN generating final dataset THEN the System SHALL include xG_home, xG_away, and xG_diff as mandatory columns

### Requirement 7: Model Calibration and Quality Assurance

**User Story:** As a data scientist, I want calibrated xG models, so that the expected values match observed frequencies over large samples.

#### Acceptance Criteria

1. WHEN calibrating xG models THEN the System SHALL ensure predicted xG averages match actual scoring averages within 5% per league/season
2. WHEN validating model quality THEN the System SHALL compute Brier score for xG-based win probability predictions
3. WHEN xG models are trained THEN the System SHALL use cross-validation with temporal splits to prevent data leakage
4. WHEN model performance degrades THEN the System SHALL log warnings and suggest recalibration
5. WHEN generating xG THEN the System SHALL use sport-specific models that are state-of-the-art for 2025 standards

