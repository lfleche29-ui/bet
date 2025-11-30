# Implementation Plan

- [x] 1. Restructurer le projet Streamlit




  - [x] 1.1 Créer la structure de dossiers (pages/, components/, services/, utils/)

    - Créer les dossiers et fichiers `__init__.py`
    - Migrer le code existant de app.py vers la nouvelle structure
    - _Requirements: 1.1, 1.2_



  - [x] 1.2 Créer le fichier de configuration centralisé





    - Créer `utils/config.py` avec AppConfig dataclass
    - Créer `streamlit_config/config.toml` pour Streamlit

    - _Requirements: 1.4_

  - [x] 1.3 Créer le module de styling





    - Créer `utils/styling.py` avec CSS custom et thèmes
    - Implémenter toggle dark/light mode
    - _Requirements: 11.1, 11.2_

- [x] 2. Implémenter ModelLoaderService





  - [x] 2.1 Créer le service de chargement de modèles


    - Créer `services/model_loader.py` avec ModelLoaderService
    - Implémenter `get_latest_model()` avec détection automatique
    - Implémenter `get_model_metadata()` pour validation
    - _Requirements: 2.1, 2.3, 2.4_

  - [x] 2.2 Write property test for model loading consistency







    - **Property 5: Model Loading Consistency**
    - **Validates: Requirements 2.1**


  - [x] 2.3 Implémenter la gestion des modèles manquants

    - Afficher warning clair si modèle absent
    - Désactiver prédictions pour le sport concerné
    - _Requirements: 2.2_

- [x] 3. Implémenter DataService





  - [x] 3.1 Créer le service de données


    - Créer `services/data_service.py` avec DataService
    - Implémenter `load_sport_data()` avec cache Streamlit
    - Implémenter `get_team_form()` pour statistiques équipe
    - _Requirements: 12.1, 12.2_

  - [x] 3.2 Implémenter le chargement des matchs récents


    - Implémenter `get_recent_matches()` pour J et J+1
    - Implémenter `get_upcoming_matches()` avec filtrage par date
    - _Requirements: 3.1_

- [x] 4. Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 5. Implémenter PredictionService





  - [x] 5.1 Créer le service de prédiction


    - Créer `services/prediction_service.py` avec PredictionService
    - Intégrer avec `prediction/predictor.py` existant
    - Implémenter `predict_match()` et `predict_batch()`
    - _Requirements: 3.2, 3.3_



  - [ ] 5.2 Implémenter le calcul des fair odds
    - Implémenter `calculate_fair_odds()` (1/probability)
    - Normaliser les probabilités si nécessaire
    - _Requirements: 4.2, 4.3_

  - [x] 5.3 Write property test for fair odds calculation






    - **Property 1: Fair Odds Calculation**
    - **Validates: Requirements 4.2**


  - [ ] 5.4 Write property test for probability normalization


    - **Property 3: Probability Normalization**


    - **Validates: Requirements 4.3**

  - [x] 5.5 Implémenter la détection de value bets





    - Implémenter `identify_value_bets()` avec seuil configurable
    - Calculer edge et Kelly fraction
    - _Requirements: 5.1, 5.4_

  - [x] 5.6 Write property test for value bet edge calculation






    - **Property 2: Value Bet Edge Calculation**
    - **Validates: Requirements 5.1**


  - [x] 5.7 Write property test for Kelly fraction bounds







    - **Property 4: Kelly Fraction Bounds**
    - **Validates: Requirements 5.4**

- [x] 6. Implémenter les composants UI





  - [x] 6.1 Créer le composant MatchCard


    - Créer `components/match_card.py`
    - Afficher équipes, probabilités, cotes, edge
    - Highlighting conditionnel pour value bets
    - _Requirements: 3.2, 3.3, 3.4, 5.2_


  - [x] 6.2 Créer le composant ValueBetTable

    - Créer `components/value_bet_table.py`
    - Table avec highlighting vert pour value bets
    - Colonnes: match, outcome, model prob, implied prob, edge, Kelly
    - _Requirements: 5.2, 5.3_



  - [x] 6.3 Créer le composant XGChart

    - Créer `components/xg_chart.py`
    - Graphique Plotly des xG rolling
    - Afficher over/under-performance

    - _Requirements: 6.1, 6.2, 6.4_

  - [x] 6.4 Créer le composant EquityCurve

    - Créer `components/equity_curve.py`
    - Graphique Plotly du profit cumulé
    - _Requirements: 8.3_

  - [x] 6.5 Créer le composant Filters


    - Créer `components/filters.py`
    - Filtres: sport, league, edge min, odds range
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5_


  - [x] 6.6 Write property test for filter application


    - **Property 7: Filter Application**
    - **Validates: Requirements 9.1, 9.2, 9.3, 9.4**

- [x] 7. Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 8. Créer les pages Streamlit





  - [x] 8.1 Créer la page Live Predictions


    - Créer `pages/1_📊_Live_Predictions.py`
    - Afficher matchs J et J+1 avec prédictions
    - Intégrer MatchCard et ValueBetTable
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5_

  - [x] 8.2 Write property test for date range validity



    - **Property 8: Date Range Validity**
    - **Validates: Requirements 3.1**



  - [x] 8.3 Créer la page Performance





    - Créer `pages/2_📈_Performance.py`
    - Afficher ROI historique, win rate, equity curve
    - Breakdown par sport et league


    - _Requirements: 8.1, 8.2, 8.3, 8.4_

  - [x] 8.4 Créer la page Value Bets





    - Créer `pages/3_🎯_Value_Bets.py`


    - Liste des value bets avec filtres
    - Export CSV
    - _Requirements: 5.1, 5.2, 5.3, 10.1, 10.2, 10.3_



  - [x] 8.5 Créer la page Analysis




    - Créer `pages/4_📉_Analysis.py`
    - Graphiques xG, forme équipes
    - Comparaison modèle vs bookmakers
    - _Requirements: 6.1, 6.2, 6.3, 7.1, 7.2, 7.3, 7.4_

  - [x] 8.6 Créer la page Settings





    - Créer `pages/5_⚙️_Settings.py`
    - Configuration edge threshold, Kelly cap, theme
    - _Requirements: 11.2_

- [x] 9. Implémenter ExportService





  - [x] 9.1 Créer le service d'export

    - Créer `services/export_service.py`
    - Implémenter export CSV des value bets
    - Nommer fichier avec date courante
    - _Requirements: 10.1, 10.2, 10.3, 10.4_


  - [x] 9.2 Write property test for export data completeness


    - **Property 6: Export Data Completeness**
    - **Validates: Requirements 10.2**

- [x] 10. Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.

- [x] 11. Refondre app.py principal






  - [x] 11.1 Mettre à jour app.py comme point d'entrée

    - Configurer multipage avec st.navigation ou pages/
    - Initialiser services dans session_state
    - Appliquer styling global
    - _Requirements: 1.1, 11.1, 11.3_

  - [x] 11.2 Implémenter le cache intelligent


    - Utiliser @st.cache_data pour données
    - Utiliser @st.cache_resource pour modèles
    - _Requirements: 12.1, 12.2, 12.3_

- [x] 12. Tests d'intégration






  - [x] 12.1 Créer tests de chargement des pages


    - Vérifier que toutes les pages chargent sans erreur
    - Vérifier que les modèles se chargent correctement
    - _Requirements: 13.1, 13.2_


  - [x] 12.2 Créer tests du flux complet


    - Test prediction → value bet → export
    - Vérifier cohérence des données entre pages
    - _Requirements: 13.3, 13.4_

- [x] 13. Documentation et finalisation

  - [x] 13.1 Créer README_STREAMLIT.md
    - Instructions d'installation
    - Guide d'utilisation avec screenshots
    - Options de configuration
    - Section troubleshooting
    - _Requirements: 14.1, 14.2, 14.3, 14.4_

  - [x] 13.2 Ajouter captures d'écran



    - Screenshot de chaque page principale
    - Ajouter au README
    - _Requirements: 14.2_

- [x] 14. Final Checkpoint - Ensure all tests pass





  - Ensure all tests pass, ask the user if questions arise.
