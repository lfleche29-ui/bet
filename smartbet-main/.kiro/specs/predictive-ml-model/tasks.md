# Implementation Plan

- [x] 1. Set up ML prediction package structure







  - [x] 1.1 Create prediction package and base classes

    - Create `prediction/` directory with `__init__.py`
    - Create `prediction/features.py` with `FeatureEngineer` class skeleton
    - Create `prediction/trainer.py` with `ModelTrainer` class skeleton
    - Create `prediction/calibrator.py` with `ProbabilityCalibrator` class skeleton
    - Create `prediction/predictor.py` with `MatchPredictor` class skeleton
    - Create `prediction/backtest.py` with `BacktestEngine` class skeleton
    - Create `prediction/evaluator.py` with `ModelEvaluator` class skeleton
    - Add dependencies to `requirements.txt` (scikit-learn, xgboost, lightgbm, hypothesis)
    - _Requirements: 2.1, 6.1_


  - [x] 1.2 Create data models and configuration

    - Create `prediction/models.py` with dataclasses (PredictionResult, BacktestResult, ModelMetadata)
    - Create `prediction/config.py` with sport-specific configurations and feature lists
    - Define xG ranges, ELO defaults, and window sizes per sport
    - _Requirements: 7.5_

- [x] 2. Implement FeatureEngineer component






  - [x] 2.1 Implement rolling statistics calculation


    - Implement `compute_rolling_stats()` for win rate, goals scored/conceded
    - Support configurable window sizes (5, 10, 20 games)
    - Handle teams with insufficient history using league averages
    - _Requirements: 1.1, 1.6_


  - [x] 2.2 Write property test for rolling statistics window

    - **Property 3: Rolling Statistics Window Correctness**
    - **Validates: Requirements 1.1**

  - [x] 2.3 Implement xG-based features


    - Implement `compute_xg_features()` for xG_diff, rolling xG averages
    - Calculate over/under-performance (actual goals - xG)
    - _Requirements: 1.2_


  - [x] 2.4 Write property test for xG difference arithmetic

    - **Property 4: xG Difference Arithmetic**
    - **Validates: Requirements 1.2**

  - [x] 2.5 Implement ELO and odds features

    - Implement `compute_elo_features()` for ELO diff, momentum, rank
    - Implement `compute_odds_features()` for implied probabilities, margin
    - _Requirements: 1.3, 1.4_


  - [x] 2.6 Write property test for implied probability calculation

    - **Property 5: Implied Probability Calculation**
    - **Validates: Requirements 1.4**

  - [x] 2.7 Implement context features and full pipeline

    - Implement `compute_context_features()` for home/away, days rest, season progress
    - Implement `generate_features()` to combine all feature types
    - _Requirements: 1.5_

- [x] 3. Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 4. Implement ModelTrainer component






  - [x] 4.1 Implement temporal data splitting


    - Implement `temporal_split()` with configurable test ratio
    - Ensure strict temporal ordering (all test dates > all train dates)
    - _Requirements: 2.2_


  - [x] 4.2 Write property test for temporal split integrity

    - **Property 2: Temporal Split Integrity**
    - **Validates: Requirements 2.2**

  - [x] 4.3 Implement model training with multiple algorithms


    - Implement Logistic Regression baseline
    - Implement XGBoost classifier
    - Implement LightGBM classifier
    - Add hyperparameter tuning with TimeSeriesSplit cross-validation
    - _Requirements: 2.1, 2.3_



  - [x] 4.4 Implement class imbalance handling
    - Add class_weight parameter for imbalanced classes
    - Special handling for Football draws (minority class)
    - _Requirements: 2.5_


  - [x] 4.5 Implement model persistence

    - Implement `save()` with joblib and metadata JSON
    - Implement `load()` with validation
    - Store training date, features, hyperparameters, metrics
    - _Requirements: 2.4, 2.6_

- [x] 5. Implement ProbabilityCalibrator component




  - [x] 5.1 Implement calibration methods


    - Implement isotonic regression calibration
    - Implement Platt scaling as alternative
    - Support multi-class calibration for Football
    - _Requirements: 3.2_


  - [x] 5.2 Implement calibration metrics
    - Implement `compute_brier_score()` for calibration quality
    - Add reliability diagram generation
    - _Requirements: 3.3, 3.4_

  - [x] 5.3 Write property test for Brier score bounds


    - **Property 6: Brier Score Bounds**
    - **Validates: Requirements 3.3**



  - [x] 5.4 Write property test for probability sum constraint

    - **Property 1: Probability Sum Constraint**
    - **Validates: Requirements 3.1**

- [x] 6. Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 7. Implement MatchPredictor component






  - [x] 7.1 Implement prediction interface


    - Implement `predict()` accepting match details
    - Return PredictionResult with probabilities and confidence intervals
    - Include feature values for transparency
    - _Requirements: 6.1, 6.4_


  - [x] 7.2 Write property test for sport-specific outcome count

    - **Property 13: Sport-Specific Outcome Count**
    - **Validates: Requirements 7.1, 7.2**


  - [x] 7.3 Write property test for confidence interval validity
    - **Property 14: Confidence Interval Validity**
    - **Validates: Requirements 6.2**


  - [x] 7.4 Implement value bet identification
    - Implement `identify_value_bets()` with configurable threshold
    - Compare model probability vs implied probability

    - _Requirements: 6.3_

  - [x] 7.5 Write property test for value bet threshold consistency
    - **Property 8: Value Bet Threshold Consistency**
    - **Validates: Requirements 4.3, 6.3**

  - [x] 7.6 Implement error handling


    - Handle missing model files with clear error messages
    - Handle missing features gracefully
    - _Requirements: 6.5_

- [x] 8. Implement BacktestEngine component

  - [x] 8.1 Implement backtest simulation
    - Implement `run()` for flat betting simulation
    - Track all bets with outcomes
    - _Requirements: 4.1_

  - [x] 8.2 Implement ROI and performance metrics
    - Implement `compute_roi()` calculation
    - Track win rate and profit/loss
    - _Requirements: 4.2_

  - [x] 8.3 Write property test for ROI calculation correctness
    - **Property 7: ROI Calculation Correctness**
    - **Validates: Requirements 4.2**

  - [x] 8.4 Implement equity curve generation
    - Implement `compute_equity_curve()` as cumulative profit
    - _Requirements: 4.5_

  - [x] 8.5 Write property test for equity curve monotonic calculation
    - **Property 9: Equity Curve Monotonic Calculation**
    - **Validates: Requirements 4.5**

  - [x] 8.6 Implement performance breakdown and statistical significance

    - Track performance by sport, league, time period
    - Implement `bootstrap_confidence_interval()` for significance
    - _Requirements: 4.4, 4.6_

- [x] 9. Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 10. Implement ModelEvaluator component





  - [x] 10.1 Implement classification metrics

    - Implement `compute_classification_metrics()` for accuracy, precision, recall, F1
    - Support multi-class for Football
    - _Requirements: 5.1_


  - [x] 10.2 Implement log loss calculation
    - Implement `compute_log_loss()` for probability quality
    - _Requirements: 5.2_

  - [x] 10.3 Write property test for log loss calculation


    - **Property 10: Log Loss Calculation**
    - **Validates: Requirements 5.2**


  - [x] 10.4 Implement confusion matrix generation
    - Implement `compute_confusion_matrix()` per sport
    - _Requirements: 5.4_


  - [x] 10.5 Write property test for confusion matrix sum integrity
    - **Property 11: Confusion Matrix Sum Integrity**

    - **Validates: Requirements 5.4**

  - [x] 10.6 Implement feature importance and model comparison
    - Implement `get_feature_importance()` for tree models
    - Implement `compare_models()` summary table
    - _Requirements: 5.5, 5.6_


  - [x] 10.7 Write property test for feature importance normalization


    - **Property 12: Feature Importance Normalization**
    - **Validates: Requirements 5.5**

  - [x] 10.8 Implement baseline comparison
    - Compare model vs bookmaker implied probabilities
    - _Requirements: 5.3_

- [x] 11. Implement multi-sport support






  - [x] 11.1 Implement Football multinomial classification


    - Configure 3-class output (home, draw, away)
    - Handle draw class imbalance
    - _Requirements: 7.1_

  - [x] 11.2 Implement NBA/NHL/Tennis binary classification


    - Configure 2-class output (home, away)
    - _Requirements: 7.2_


  - [x] 11.3 Implement sport-specific features

    - Add surface type for Tennis
    - Handle NHL overtime appropriately
    - Use sport-specific xG normalization
    - _Requirements: 7.3, 7.4, 7.5_

- [x] 12. Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 13. Train and validate models






  - [x] 13.1 Train models for all sports


    - Train Football model on historical data
    - Train NBA model on historical data
    - Train NHL model on historical data
    - Train Tennis model on historical data
    - _Requirements: 2.6_


  - [x] 13.2 Evaluate and compare models

    - Run evaluation on test set (2024-2025 data)
    - Compare XGBoost vs LightGBM vs Logistic Regression
    - Generate comparison report
    - _Requirements: 5.3, 5.6_


  - [x] 13.3 Run backtesting validation

    - Backtest on 2024-2025 data
    - Calculate ROI and statistical significance
    - Generate equity curves
    - _Requirements: 4.1, 4.2, 4.6_


  - [x] 13.4 Write integration tests for full pipeline

    - Test end-to-end: data → features → training → prediction → backtest
    - Verify model files are created correctly
    - Verify predictions match expected format
    - _Requirements: 6.1, 2.4_

- [x] 14. Final Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

