# Implementation Plan - Ghost Bet Pan-Europe Strategy v3.0

- [x] 1. Update core configuration and models






  - [x] 1.1 Update `strategies/ghost_bet/config.py` with GhostBetPanEuropeCriteria


    - Define TOP5_LEAGUES = ['E0', 'SP1', 'I1', 'D1', 'F1']
    - Define thresholds: min_away_prob=0.665, min_draw_odds=5.05, max_total_xg=2.78, min_home_goal_diff_last5=1.20, max_away_rank=6
    - Remove old rest_days related config
    - _Requirements: 1.1, 1.2, 1.3, 2.2, 3.1, 4.1_

  - [x] 1.2 Update `strategies/ghost_bet/models.py` with new GhostBetOpportunity dataclass


    - Add xG_home, xG_away, home_goal_diff_last5, away_league_rank fields
    - Remove rest_days field
    - _Requirements: 6.2_

- [x] 2. Implement new Ghost Bet Pan-Europe Filter











  - [x] 2.1 Rewrite `strategies/ghost_bet/filter.py` with GhostBetPanEuropeFilter class



    - Implement is_ghost_bet() with all 6 criteria
    - Implement filter_matches() for batch filtering
    - Remove rest_days logic entirely
    - _Requirements: 1.1, 1.2, 1.3, 2.2, 3.1, 4.1_

  - [x] 2.2 Write property test for filter completeness







    - **Property 1: Ghost Bet Filter Completeness**
    - **Validates: Requirements 1.1, 1.2, 1.3, 2.2, 3.1, 4.1**


  - [x] 2.3 Write property test for filter determinism







    - **Property 6: Filter Determinism**
    - **Validates: Requirements 7.1**

- [x] 3. Checkpoint - Ensure filter tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 4. Update Backtest Engine




  - [x] 4.1 Update `strategies/ghost_bet/backtester.py` for Pan-Europe


    - Update profit calculation: (odds_draw - 1) if draw else -1
    - Keep flat 1% stakes
    - Update breakdown by league (5 leagues only)
    - _Requirements: 5.1, 5.2, 5.3, 5.4_





  - [x] 4.2 Write property test for ROI calculation


    - **Property 2: ROI Calculation Correctness**

    - **Validates: Requirements 5.2**




  - [x] 4.3 Write property test for strike rate calculation


    - **Property 3: Strike Rate Calculation Correctness**


    - **Validates: Requirements 5.2**

  - [x] 4.4 Write property test for equity curve consistency



    - **Property 4: Equity Curve Consistency**
    - **Validates: Requirements 5.3**



  - [x] 4.5 Write property test for breakdown aggregation



    - **Property 5: Breakdown Aggregation Consistency**
    - **Validates: Requirements 5.4**



  - [x] 4.6 Write property test for profit calculation







    - **Property 8: Profit Calculation Correctness**

    - **Validates: Requirements 5.2**

- [x] 5. Update Daily Scanner

  - [x] 5.1 Update `strategies/ghost_bet/scanner.py` for Pan-Europe
    - Use new GhostBetPanEuropeFilter
    - Update format_recommendation() with new fields
    - Handle empty results gracefully
    - _Requirements: 6.1, 6.2, 6.3_

  - [x] 5.2 Write property test for recommendation completeness

    - **Property 7: Recommendation Completeness**
    - **Validates: Requirements 6.2**

- [x] 6. Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 7. Update CLI scripts






  - [x] 7.1 Create `strategies/ghost_bet_pan_europe.py` official script


    - Load FOOTBALL_WITH_XG dataset
    - Apply all 6 filters
    - Export signals to live/ghost_bet_pan_europe_signals.csv
    - _Requirements: 5.1, 6.1_

  - [x] 7.2 Update `scripts/run_ghost_bet_backtest.py` for Pan-Europe


    - Use new filter criteria
    - Output ROI by league breakdown
    - _Requirements: 5.1, 5.2, 5.3, 5.4_


  - [x] 7.3 Update `scripts/scan_ghost_bets.py` for daily recommendations

    - Use new filter criteria
    - Output all required fields
    - _Requirements: 6.1, 6.2, 6.3_

- [x] 8. Clean up deprecated code






  - [x] 8.1 Remove `strategies/ghost_bet/rest_calculator.py` (no longer needed)


    - Delete file entirely
    - Remove imports from __init__.py


  - [x] 8.2 Update `strategies/ghost_bet/__init__.py` exports

    - Export new GhostBetPanEuropeCriteria
    - Export new GhostBetPanEuropeFilter
    - Remove rest_calculator exports


  - [x] 8.3 Update property tests in `tests/test_properties/test_ghost_bet_properties.py`

    - Remove rest_days related tests
    - Update filter completeness test for new criteria

- [x] 9. Final Checkpoint - Ensure all tests pass




  - Ensure all tests pass, ask the user if questions arise.

