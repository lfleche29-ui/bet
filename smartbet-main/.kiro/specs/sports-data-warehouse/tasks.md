# Implementation Plan

- [x] 1. Set up project structure and core interfaces






  - [x] 1.1 Create ETL package structure

    - Create `etl/` directory with `__init__.py`, `loader.py`, `cleaner.py`, `schema.py`, `exporter.py`, `reporter.py`
    - Define base interfaces and type hints for all components
    - Add required dependencies to `requirements.txt` (hypothesis, pyarrow, duckdb)
    - _Requirements: 1.1, 3.1, 4.1_


  - [x] 1.2 Create configuration module





    - Define constants for valid ranges (odds: 1.01-1000, dates: 2005-01-01 to today)
    - Define sport-specific column mappings from raw CSV to normalized schema
    - Create logging configuration for ETL pipeline
    - _Requirements: 2.3, 2.4_

- [x] 2. Implement DataLoader component






  - [x] 2.1 Implement CSV loading with date parsing

    - Create `DataLoader.load_csv()` method with ISO 8601 date parsing
    - Handle encoding detection and UTF-8 conversion
    - Implement error logging for malformed rows with row numbers
    - _Requirements: 1.1, 1.2_


  - [x] 2.2 Write property test for date parsing round-trip

    - **Property 1: Date Parsing Consistency**
    - **Validates: Requirements 1.1**



  - [x] 2.3 Implement multi-sport loading
    - Create `DataLoader.load_all_sports()` to load all CSV files from dataset folder
    - Implement record count reporting per sport and file
    - _Requirements: 1.3_

  - [x] 2.4 Write property test for record count preservation


    - **Property 2: Record Count Preservation**
    - **Validates: Requirements 1.3**

  - [x] 2.5 Implement schema validation


    - Create `DataLoader.validate_schema()` to check required columns per sport
    - Raise `SchemaValidationError` with column name and file path for missing columns
    - _Requirements: 1.4_

- [x] 3. Implement DataCleaner component





  - [x] 3.1 Implement name normalization


    - Create `DataCleaner.normalize_names()` for trimming whitespace and UTF-8 encoding
    - Handle special characters and accents consistently
    - _Requirements: 2.1_

  - [x] 3.2 Write property test for name normalization idempotence


    - **Property 4: Name Normalization Idempotence**
    - **Validates: Requirements 2.1**

  - [x] 3.3 Implement odds validation


    - Create `DataCleaner.validate_odds()` to set invalid odds (outside 1.01-1000) to NULL
    - Log conversions with event details
    - _Requirements: 2.3_


  - [x] 3.4 Write property test for odds validation boundary

    - **Property 5: Odds Validation Boundary**
    - **Validates: Requirements 2.3**

  - [x] 3.5 Implement score validation


    - Create `DataCleaner.validate_scores()` to set negative scores to NULL
    - Log anomalies with event details
    - _Requirements: 2.6_


  - [x] 3.6 Implement season parsing

    - Create `DataCleaner.parse_season()` to extract start_year and end_year
    - Handle various season formats (20192020, 2019-2020, 2019)
    - _Requirements: 2.5_


  - [x] 3.7 Write property test for season parsing correctness

    - **Property 6: Season Parsing Correctness**
    - **Validates: Requirements 2.5**

  - [x] 3.8 Implement deduplication


    - Create `DataCleaner.deduplicate()` to remove duplicates (same date, home, away, sport)
    - Keep most recent record, return count of removed duplicates
    - _Requirements: 1.5_


  - [x] 3.9 Write property test for deduplication correctness

    - **Property 3: Duplicate Deduplication Correctness**
    - **Validates: Requirements 1.5**

- [x] 4. Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 5. Implement SchemaManager component





  - [x] 5.1 Create SQLAlchemy models for star schema


    - Define `Sports`, `Participants`, `ParticipantAliases`, `Events`, `Outcomes`, `Odds`, `Metrics` models
    - Set up primary keys, foreign keys, and constraints as per design
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6_


  - [x] 5.2 Implement table creation

    - Create `SchemaManager.create_tables()` to create all tables with proper constraints
    - Support both SQLite and PostgreSQL dialects
    - _Requirements: 3.1, 4.1_


  - [x] 5.3 Implement participant management

    - Create `SchemaManager.get_or_create_participant()` for participant lookup/creation
    - Handle participant aliases for historical name changes
    - _Requirements: 3.2, 7.5_


  - [x] 5.4 Implement event insertion

    - Create `SchemaManager.insert_event()` with related odds, outcomes, metrics
    - Use transactions for data integrity
    - _Requirements: 3.1, 3.3, 3.4, 3.5_

- [x] 6. Implement feature calculations





  - [x] 6.1 Implement implied probability calculation


    - Calculate implied probabilities from odds (1/odds) for all markets
    - Handle NULL odds gracefully
    - _Requirements: 6.1_


  - [x] 6.2 Write property test for implied probability calculation

    - **Property 9: Implied Probability Calculation**
    - **Validates: Requirements 6.1**

  - [x] 6.3 Implement bookmaker margin calculation


    - Calculate margin as (sum of implied probabilities - 1)
    - Handle 2-way (no draw) and 3-way markets
    - _Requirements: 6.2_


  - [x] 6.4 Write property test for bookmaker margin calculation

    - **Property 10: Bookmaker Margin Calculation**
    - **Validates: Requirements 6.2**

  - [x] 6.5 Preserve ELO and probability columns


    - Map source ELO ratings to metrics table
    - Preserve Poisson and hybrid probability columns for football
    - _Requirements: 6.4, 6.5_

- [x] 7. Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 8. Implement DataExporter component





  - [x] 8.1 Implement SQLite export


    - Create `DataExporter.export_sqlite()` to export all tables with foreign key constraints
    - Create single database file with proper schema
    - _Requirements: 4.1_


  - [x] 8.2 Implement Parquet export

    - Create `DataExporter.export_parquet()` with partitioning by sport and year
    - Use snappy compression and include schema metadata
    - _Requirements: 4.2, 4.4_


  - [x] 8.3 Implement export validation

    - Create `DataExporter.validate_export()` to verify row counts match source
    - Raise `DataIntegrityError` on mismatch
    - _Requirements: 4.3_


  - [x] 8.4 Write property test for export row count integrity

    - **Property 7: Export Row Count Integrity**
    - **Validates: Requirements 4.3**



  - [x] 8.5 Write property test for serialization round-trip





    - **Property 8: Serialization Round-Trip**
    - **Validates: Requirements 4.5**

- [x] 9. Implement QualityReporter component





  - [x] 9.1 Implement missing value analysis


    - Create `QualityReporter.analyze_missing()` to calculate missing percentages per column
    - Group results by sport
    - _Requirements: 5.1_


  - [x] 9.2 Write property test for missing value report accuracy

    - **Property 12: Missing Value Report Accuracy**
    - **Validates: Requirements 5.1**


  - [x] 9.3 Implement outlier detection

    - Create `QualityReporter.detect_outliers()` using IQR method
    - Flag values outside [Q1 - 1.5*IQR, Q3 + 1.5*IQR]
    - _Requirements: 5.2_


  - [x] 9.4 Write property test for outlier detection consistency

    - **Property 13: Outlier Detection Consistency**
    - **Validates: Requirements 5.2**


  - [x] 9.5 Implement duplicate and coverage reporting

    - Report duplicates found and removed per sport
    - Calculate data coverage by date range per sport
    - _Requirements: 5.3, 5.4_

  - [x] 9.6 Implement report generation


    - Create `QualityReporter.generate_report()` for JSON and Markdown output
    - Include all quality metrics in reports
    - _Requirements: 5.5_

- [x] 10. Implement multi-sport consistency features



  - [x] 10.1 Implement sport filtering


    - Add sport_id filter support to all query methods
    - Ensure consistent column names across sports
    - _Requirements: 7.1, 7.3_



  - [x] 10.2 Write property test for sport filter isolation
    - **Property 11: Sport Filter Isolation**
    - **Validates: Requirements 7.3**

  - [x] 10.3 Implement sport-specific JSON storage



    - Store sport-specific metrics in JSON column
    - Define JSON schemas for Football, NBA, NHL, Tennis
    - _Requirements: 7.2_

- [x] 11. Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 12. Create main ETL pipeline orchestrator





  - [x] 12.1 Implement pipeline orchestrator


    - Create `ETLPipeline` class to coordinate all components
    - Implement `run()` method for full ETL execution
    - Add CLI interface with argparse for configuration
    - _Requirements: 1.1, 1.3, 4.1, 4.2_



  - [x] 12.2 Implement pipeline logging and monitoring





    - Add progress logging for each pipeline stage
    - Generate summary statistics on completion
    - _Requirements: 1.2, 1.3_

- [x] 13. Integration and end-to-end testing






  - [x] 13.1 Write integration tests for full pipeline


    - Test end-to-end pipeline with sample data
    - Verify foreign key constraints in SQLite export
    - Validate Parquet file structure and partitioning
    - _Requirements: 1.1, 4.1, 4.2, 4.3_

- [x] 14. Final Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.
