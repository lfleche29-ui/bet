# Requirements Document

## Introduction

Ce projet vise à transformer les données sportives historiques brutes (Football, Tennis, NBA, NHL) en une base de données normalisée, optimisée pour le machine learning et l'analyse prédictive dans le domaine du sport betting. La nouvelle structure doit permettre des analyses multi-sports, une modélisation prédictive efficace et un passage à l'échelle.

## Glossary

- **Event**: Un match ou une rencontre sportive entre deux participants (équipes ou joueurs)
- **Participant**: Une équipe ou un joueur participant à un événement sportif
- **Odds**: Les cotes proposées par les bookmakers pour un marché donné
- **Market**: Un type de pari (victoire domicile, match nul, victoire extérieur, etc.)
- **Outcome**: Le résultat réel d'un événement ou d'un marché
- **ELO Rating**: Système de classement basé sur les performances relatives
- **xG (Expected Goals)**: Métrique avancée mesurant la qualité des occasions de but
- **Implied Probability**: Probabilité dérivée des cotes (1/cote)
- **Value Bet**: Pari où la probabilité estimée dépasse la probabilité implicite des cotes
- **ETL Pipeline**: Processus Extract-Transform-Load pour le traitement des données
- **Star Schema**: Modèle de données avec une table de faits centrale et des tables de dimensions

## Requirements

### Requirement 1: Data Loading and Validation

**User Story:** As a data engineer, I want to load all raw CSV files into a unified pipeline, so that I can process and validate the data consistently across all sports.

#### Acceptance Criteria

1. WHEN the ETL pipeline starts THEN the System SHALL load all CSV files from the dataset folder and parse dates in ISO 8601 format
2. WHEN a CSV file contains malformed rows THEN the System SHALL log the error with row number and continue processing remaining rows
3. WHEN loading is complete THEN the System SHALL report the count of loaded records per sport and per file
4. WHEN a required column is missing from a CSV file THEN the System SHALL raise a validation error with the column name and file path
5. WHEN duplicate events are detected (same date, home, away, sport) THEN the System SHALL keep the most recent record and log duplicates removed

### Requirement 2: Data Cleaning and Transformation

**User Story:** As a data scientist, I want clean and consistent data across all sports, so that I can build reliable predictive models.

#### Acceptance Criteria

1. WHEN processing team/player names THEN the System SHALL normalize names by trimming whitespace and standardizing encoding to UTF-8
2. WHEN a numeric column contains non-numeric values THEN the System SHALL convert the value to NULL and log the conversion
3. WHEN odds values are outside valid range (1.01 to 1000.0) THEN the System SHALL flag the record as invalid and set odds to NULL
4. WHEN date values are outside valid range (2005-01-01 to current date) THEN the System SHALL flag the record as invalid
5. WHEN processing season values THEN the System SHALL extract start_year and end_year as separate integer columns
6. WHEN score values are negative THEN the System SHALL set the score to NULL and log the anomaly

### Requirement 3: Schema Normalization

**User Story:** As a database architect, I want a normalized star schema, so that the data is optimized for analytical queries and machine learning.

#### Acceptance Criteria

1. WHEN creating the events table THEN the System SHALL include event_id (primary key), sport_id, date, season_start_year, season_end_year, event_type, home_participant_id, away_participant_id, and venue_type
2. WHEN creating the participants table THEN the System SHALL include participant_id (primary key), sport_id, name, participant_type (team/player), and country
3. WHEN creating the odds table THEN the System SHALL include odds_id (primary key), event_id (foreign key), market_type, odds_home, odds_draw (nullable), odds_away, and is_generated flag
4. WHEN creating the outcomes table THEN the System SHALL include outcome_id (primary key), event_id (foreign key), score_home, score_away, result_code (0=away_win, 1=home_win, 2=draw), and overtime_flag
5. WHEN creating the metrics table THEN the System SHALL include metrics_id (primary key), event_id (foreign key), xg_home, xg_away, home_elo, away_elo, and sport-specific metrics as JSON
6. WHEN creating the sports table THEN the System SHALL include sport_id (primary key), sport_name, has_draw_market, and default_score_type

### Requirement 4: Data Export

**User Story:** As a data engineer, I want to export the normalized data to multiple formats, so that the data can be consumed by different systems and tools.

#### Acceptance Criteria

1. WHEN exporting to SQLite THEN the System SHALL create a single database file with all tables and proper foreign key constraints
2. WHEN exporting to Parquet THEN the System SHALL create partitioned files by sport and year for efficient querying
3. WHEN exporting is complete THEN the System SHALL validate row counts match between source and destination
4. WHEN exporting to Parquet THEN the System SHALL use snappy compression and include schema metadata
5. WHEN serializing data to Parquet or SQLite THEN the System SHALL produce output that can be deserialized back to equivalent data structures

### Requirement 5: Data Quality Reporting

**User Story:** As a data analyst, I want comprehensive data quality reports, so that I can understand data completeness and identify issues.

#### Acceptance Criteria

1. WHEN analysis is complete THEN the System SHALL generate a report showing missing value percentages per column per sport
2. WHEN analysis is complete THEN the System SHALL identify and report statistical outliers in numeric columns using IQR method
3. WHEN analysis is complete THEN the System SHALL report duplicate records found and removed per sport
4. WHEN analysis is complete THEN the System SHALL calculate and report data coverage by date range per sport
5. WHEN generating reports THEN the System SHALL output reports in both JSON and Markdown formats

### Requirement 6: Feature Engineering Support

**User Story:** As a data scientist, I want pre-computed features and the ability to calculate advanced metrics, so that I can accelerate model development.

#### Acceptance Criteria

1. WHEN processing events THEN the System SHALL calculate implied probabilities from odds (1/odds) for all markets
2. WHEN processing events THEN the System SHALL calculate bookmaker margin as (sum of implied probabilities - 1)
3. WHEN querying participant history THEN the System SHALL support calculating rolling statistics over N previous events
4. WHEN processing events THEN the System SHALL preserve ELO ratings and probability columns from source data
5. WHEN processing football events THEN the System SHALL preserve Poisson and hybrid probability columns

### Requirement 7: Multi-Sport Consistency

**User Story:** As a machine learning engineer, I want consistent data structures across sports, so that I can train multi-sport models efficiently.

#### Acceptance Criteria

1. WHEN storing events THEN the System SHALL use identical column names and types across all sports for common fields
2. WHEN storing sport-specific data THEN the System SHALL use a flexible JSON column in the metrics table
3. WHEN querying events THEN the System SHALL support filtering by sport_id to isolate single-sport datasets
4. WHEN joining tables THEN the System SHALL use consistent foreign key naming convention (table_id format)
5. WHEN storing participant names THEN the System SHALL maintain a mapping table for historical name changes (team relocations, rebranding)

