# Implementation Plan

- [x] 1. Set up xG generators package structure






  - [x] 1.1 Create generators package and base classes

    - Create `generators/` directory with `__init__.py`
    - Create `generators/base.py` with `BaseXGGenerator` abstract class
    - Create `generators/config.py` with `XGConfig` dataclass and sport configurations
    - Define common interfaces: `calculate_xg()`, `get_valid_range()`, `generate_all()`
    - _Requirements: 1.1, 2.1, 3.1, 4.1_


  - [x] 1.2 Create team statistics calculator

    - Create `generators/team_stats.py` with `TeamStatsCalculator` class
    - Implement rolling window statistics (attack strength, defense strength)
    - Implement league average calculations per sport/season
    - _Requirements: 1.2, 2.2, 3.2_

- [x] 2. Implement Football xG Generator






  - [x] 2.1 Implement FootballXGGenerator core logic

    - Create `generators/football_xg.py` with `FootballXGGenerator` class
    - Implement Dixon-Coles inspired attack/defense strength model
    - Implement home advantage factor (1.20)
    - Implement Poisson smoothing for realistic distributions
    - _Requirements: 1.1, 1.2, 1.5_

  - [x] 2.2 Write property test for football xG bounds






    - **Property 1: Sport-Specific xG Bounds (Football)**
    - Test all generated values are in [0.0, 6.0] range
    - **Validates: Requirements 1.1**


  - [x] 2.3 Implement football xG generation pipeline

    - Implement `generate_all()` method for batch processing
    - Implement fallback to league averages for new teams
    - Add logging for statistics (mean, std, min, max)
    - _Requirements: 1.3, 5.3, 5.4_

- [x] 3. Implement NBA xPoints Generator




  - [x] 3.1 Implement NBAXPointsGenerator core logic


    - Create `generators/nba_xpoints.py` with `NBAXPointsGenerator` class
    - Implement offensive/defensive rating calculations
    - Implement pace factor adjustments
    - Implement home court advantage (3.5 points)
    - _Requirements: 2.1, 2.2, 2.5_

  - [x] 3.2 Write property test for NBA xPoints bounds





    - **Property 1: Sport-Specific xG Bounds (NBA)**
    - Test all generated values are in [85.0, 135.0] range
    - **Validates: Requirements 2.1**

  - [x] 3.3 Implement NBA xPoints generation pipeline


    - Implement `generate_all()` method for batch processing
    - Implement rolling team statistics from previous games only
    - Add logging for statistics
    - _Requirements: 2.3, 2.5, 5.3_

- [x] 4. Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 5. Implement NHL xG Generator




  - [x] 5.1 Implement NHLXGGenerator core logic


    - Create `generators/nhl_xg.py` with `NHLXGGenerator` class
    - Implement shot quality model (shots × quality × save%)
    - Implement home ice advantage (1.10 factor)
    - Implement existing xG preservation logic
    - _Requirements: 3.1, 3.2, 3.5_

  - [x] 5.2 Write property test for NHL xG bounds

    - **Property 1: Sport-Specific xG Bounds (NHL)**
    - Test all generated values are in [1.5, 5.0] range
    - **Validates: Requirements 3.1**


  - [x] 5.3 Write property test for NHL existing xG preservation






    - **Property 4: NHL Existing xG Preservation**
    - Test existing valid xG values are not overwritten
    - **Validates: Requirements 3.5**

  - [x] 5.4 Implement NHL xG generation pipeline

    - Implement `generate_all()` with preservation of existing values
    - Fill only missing xG values
    - Add logging for preserved vs generated counts
    - _Requirements: 3.3, 3.5, 5.3_

- [x] 6. Implement Tennis xGames Generator



  - [x] 6.1 Implement TennisXGamesGenerator core logic

    - Create `generators/tennis_xgames.py` with `TennisXGamesGenerator` class
    - Implement Elo-based win probability calculation
    - Implement surface variance multipliers (clay=1.15, grass=0.90, hard=1.0)
    - Implement fatigue factor (matches in last 7 days)
    - _Requirements: 4.1, 4.2, 4.5_


  - [x] 6.2 Write property test for Tennis xGames bounds






    - **Property 1: Sport-Specific xG Bounds (Tennis)**
    - Test all generated values are in [0.0, 40.0] range
    - **Validates: Requirements 4.1**



  - [x] 6.3 Write property test for Tennis surface variance


    - **Property 5: Tennis Surface Variance Adjustment**
    - Test clay matches have higher xG variance than grass matches
    - **Validates: Requirements 4.5**

  - [x] 6.4 Implement Tennis xGames generation pipeline


    - Implement `generate_all()` method
    - Handle surface-specific calculations
    - Add logging for statistics per surface
    - _Requirements: 4.3, 4.5, 5.3_

- [x] 7. Checkpoint - Ensure all tests pass



  - Ensure all tests pass, ask the user if questions arise.

- [x] 8. Implement xG_diff calculation and validation






  - [x] 8.1 Implement xG_diff calculation

    - Add `calculate_xg_diff()` method to base class
    - Ensure xG_diff = xG_home - xG_away for all sports
    - Add xG_diff column to all output dataframes
    - _Requirements: 1.4, 2.4, 3.4, 4.4, 6.1_

  - [x] 8.2 Write property test for xG_diff arithmetic



    - **Property 2: xG_diff Arithmetic Correctness**
    - Test xG_diff equals (xG_home - xG_away) within tolerance
    - **Validates: Requirements 1.4, 2.4, 3.4, 4.4, 6.1**


  - [x] 8.3 Implement XGValidator class

    - Create `generators/validator.py` with `XGValidator` class
    - Implement `validate_bounds()` for sport-specific range checking
    - Implement `validate_completeness()` for NULL checking
    - Implement `calculate_correlation()` for quality metrics
    - _Requirements: 5.1, 5.2, 7.1_

  - [x] 8.4 Write property test for no NULL values






    - **Property 3: No NULL Values in xG Columns**
    - Test xG_home, xG_away, xG_diff have no NaN values
    - **Validates: Requirements 5.1, 6.4**

  - [x] 8.5 Write property test for required columns






    - **Property 7: Required Columns Exist**
    - Test output has xG_home, xG_away, xG_diff columns of numeric type
    - **Validates: Requirements 6.4**

- [x] 9. Implement XGPipeline orchestrator




  - [x] 9.1 Create main pipeline orchestrator

    - Create `generators/pipeline.py` with `XGPipeline` class
    - Implement `run()` method to process all 4 sports
    - Implement dataset loading from CSV files
    - Implement result export to updated CSV files
    - _Requirements: 5.1, 5.5_


  - [x] 9.2 Implement validation and reporting

  
    - Generate validation report with correlations per sport
    - Log statistics (mean, std, min, max) per sport/season
    - Implement calibration check (predicted avg vs actual avg within 5%)
    - _Requirements: 5.3, 5.5, 7.1, 7.2_


  - [x] 9.3 Write property test for xG_diff predictive direction






    - **Property 6: xG_diff Predictive Direction**
    - Test positive xG_diff correlates with higher home win rate
    - **Validates: Requirements 6.3**

- [x] 10. Checkpoint - Ensure all tests pass










  - Ensure all tests pass, ask the user if questions arise.

- [x] 11. Integration and dataset generation


  - [x] 11.1 Run full xG generation on all datasets





    - Process Football dataset (~37k matches)
    - Process NBA dataset (~24k matches)
    - Process NHL dataset (~24k matches, preserve existing)
    - Process Tennis dataset (~53k matches)
    - _Requirements: 5.1_

  - [x] 11.2 Validate and export final datasets








    - Verify 100% completeness (zero NULL in xG columns)
    - Verify correlations meet thresholds (Football>0.70, NBA>0.75, NHL>0.65, Tennis>0.60)
    - Export updated FINAL.csv files with xG_home, xG_away, xG_diff
    - Generate final validation report
    - _Requirements: 1.3, 2.3, 3.3, 4.3, 5.1, 5.5_

- [x] 11.3 Write integration tests for full pipeline






  - Test end-to-end pipeline with sample data
  - Verify output schema matches expected columns
  - Verify correlation thresholds are met
  - _Requirements: 5.5, 7.1, 7.2_

- [x] 12. Final Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

