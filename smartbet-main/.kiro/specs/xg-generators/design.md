# Design Document: xG Generators for Multi-Sport Analytics

## Overview

This design document describes the architecture and implementation of state-of-the-art xG (Expected Goals/Points/Games) generators for 4 sports: Football, NBA, NHL, and Tennis. The system transforms the existing dataset columns xG_home and xG_away from empty/partial to 100% complete with realistic, highly predictive values.

### Current Data State

| Sport | Records | xG Status | Target Metric |
|-------|---------|-----------|---------------|
| Football | ~36,891 | 100% empty | xG (Expected Goals) 0.0-6.0 |
| NBA | ~23,235 | 100% empty | xPoints (Expected Points) 85-135 |
| NHL | ~24,319 | Partial (~30% filled) | xG Hockey 1.5-5.0 |
| Tennis | ~52,611 | 100% empty | xGames (Expected Games) 0-40 |

### Design Goals

1. **100% Completeness**: Zero NULL values in xG_home, xG_away, xG_diff
2. **High Correlation**: xG values correlate strongly with actual results (>0.60-0.75 depending on sport)
3. **Realistic Distributions**: Values match real-world xG distributions per sport
4. **No Data Leakage**: Historical xG uses only pre-match data
5. **State-of-the-Art Models**: 2025-level accuracy using proven methodologies

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        xG Generation Pipeline                            │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │  DataLoader  │───▶│  xG Engine   │───▶│  Validator   │              │
│  │  (CSV/DF)    │    │  (Per Sport) │    │  (Quality)   │              │
│  └──────────────┘    └──────────────┘    └──────────────┘              │
│         │                   │                   │                       │
│         ▼                   ▼                   ▼                       │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │ Team Stats   │    │ Sport Models │    │ Calibration  │              │
│  │ Calculator   │    │ (4 engines)  │    │ Reporter     │              │
│  └──────────────┘    └──────────────┘    └──────────────┘              │
└─────────────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────────────┐
                    │      Sport-Specific xG      │
                    │         Generators          │
                    ├─────────────────────────────┤
                    │  ┌─────────┐  ┌─────────┐  │
                    │  │Football │  │  NBA    │  │
                    │  │ xG Gen  │  │xPoints  │  │
                    │  └─────────┘  └─────────┘  │
                    │  ┌─────────┐  ┌─────────┐  │
                    │  │  NHL    │  │ Tennis  │  │
                    │  │ xG Gen  │  │xGames   │  │
                    │  └─────────┘  └─────────┘  │
                    └─────────────────────────────┘
```

## Components and Interfaces

### 1. BaseXGGenerator (Abstract Base Class)

```python
from abc import ABC, abstractmethod
import pandas as pd
import numpy as np

class BaseXGGenerator(ABC):
    """Abstract base class for all xG generators."""
    
    def __init__(self, df: pd.DataFrame, sport: str):
        self.df = df
        self.sport = sport
        self.team_stats = {}
        self.league_averages = {}
    
    @abstractmethod
    def calculate_xg(self, row: pd.Series) -> Tuple[float, float]:
        """Calculate xG_home and xG_away for a single match."""
        pass
    
    @abstractmethod
    def get_valid_range(self) -> Tuple[float, float]:
        """Return valid (min, max) range for this sport's xG."""
        pass
    
    def generate_all(self) -> pd.DataFrame:
        """Generate xG for all matches in dataframe."""
        pass
    
    def calculate_team_stats(self) -> Dict:
        """Calculate rolling team statistics."""
        pass
    
    def get_fallback_xg(self) -> Tuple[float, float]:
        """Return league average xG as fallback."""
        pass
```

### 2. FootballXGGenerator

```python
class FootballXGGenerator(BaseXGGenerator):
    """
    Football xG generator using Poisson-based model.
    
    Model: xG = base_rate * attack_strength * defense_weakness * home_factor
    
    References:
    - Understat xG methodology
    - FBref expected goals model
    - Dixon-Coles model for football prediction
    """
    
    HOME_ADVANTAGE = 1.20  # 20% boost for home team
    VALID_RANGE = (0.0, 6.0)
    
    def calculate_xg(self, row: pd.Series) -> Tuple[float, float]:
        """
        Calculate football xG using team strength model.
        
        Formula:
        xG_home = league_avg * home_attack * away_defense * HOME_ADVANTAGE
        xG_away = league_avg * away_attack * home_defense
        """
        pass
    
    def calculate_attack_strength(self, team: str, date: datetime) -> float:
        """Calculate team's attacking strength relative to league average."""
        pass
    
    def calculate_defense_weakness(self, team: str, date: datetime) -> float:
        """Calculate team's defensive weakness (inverse of strength)."""
        pass
```

### 3. NBAXPointsGenerator

```python
class NBAXPointsGenerator(BaseXGGenerator):
    """
    NBA xPoints generator using offensive/defensive rating model.
    
    Model: xPoints = pace * (off_rating + def_rating_opp) / 200 * home_factor
    
    References:
    - NBA.com Advanced Stats
    - Basketball-Reference ratings
    - Second Spectrum shot quality models
    """
    
    HOME_ADVANTAGE = 3.5  # ~3.5 points home court advantage
    VALID_RANGE = (85.0, 135.0)
    LEAGUE_AVG_PACE = 100.0
    LEAGUE_AVG_POINTS = 110.0
    
    def calculate_xg(self, row: pd.Series) -> Tuple[float, float]:
        """
        Calculate NBA xPoints using team ratings.
        
        Formula:
        xPoints_home = (off_rtg_home + def_rtg_away) / 2 * pace_factor + HOME_ADV
        xPoints_away = (off_rtg_away + def_rtg_home) / 2 * pace_factor
        """
        pass
    
    def calculate_offensive_rating(self, team: str, date: datetime) -> float:
        """Calculate team's points per 100 possessions."""
        pass
    
    def calculate_defensive_rating(self, team: str, date: datetime) -> float:
        """Calculate opponent's points per 100 possessions allowed."""
        pass
```

### 4. NHLXGGenerator

```python
class NHLXGGenerator(BaseXGGenerator):
    """
    NHL xG generator using shot-based model.
    
    Model: xG = shots * shot_quality * (1 - save_pct_opp) * home_factor
    
    References:
    - MoneyPuck xG model
    - Evolving-Hockey expected goals
    - NaturalStatTrick shot quality
    """
    
    HOME_ADVANTAGE = 1.10  # 10% home ice advantage
    VALID_RANGE = (1.5, 5.0)
    LEAGUE_AVG_GOALS = 3.0
    
    def calculate_xg(self, row: pd.Series) -> Tuple[float, float]:
        """
        Calculate NHL xG using shot quality model.
        
        Formula:
        xG_home = shots_home * quality_home * (1 - sv%_away) * HOME_ADV
        xG_away = shots_away * quality_away * (1 - sv%_home)
        """
        pass
    
    def preserve_existing_xg(self, row: pd.Series) -> bool:
        """Check if row has existing valid xG values to preserve."""
        pass
```

### 5. TennisXGamesGenerator

```python
class TennisXGamesGenerator(BaseXGGenerator):
    """
    Tennis xGames generator using service/return model.
    
    Model: xGames = f(serve_pct, return_pct, elo_diff, surface, fatigue)
    
    References:
    - Tennis Abstract xG model
    - Klaassen & Magnus (2001) point-by-point model
    - Jeff Sackmann's tennis ratings
    """
    
    SURFACE_VARIANCE = {
        'Clay': 1.15,      # Higher variance on clay
        'Hard': 1.00,      # Baseline
        'Grass': 0.90,     # Lower variance on grass
        'Carpet': 0.95,
    }
    VALID_RANGE = (0.0, 40.0)  # Max for 5-set match
    
    def calculate_xg(self, row: pd.Series) -> Tuple[float, float]:
        """
        Calculate tennis xGames using Elo and surface model.
        
        Formula:
        win_prob = 1 / (1 + 10^((elo_away - elo_home) / 400))
        xGames_home = expected_games(win_prob, surface_var)
        """
        pass
    
    def calculate_fatigue_factor(self, player: str, date: datetime) -> float:
        """Calculate fatigue based on recent match load."""
        pass
    
    def get_surface_variance(self, surface: str) -> float:
        """Get variance multiplier for surface type."""
        pass
```

### 6. XGValidator

```python
class XGValidator:
    """Validates generated xG values for quality and completeness."""
    
    def validate_bounds(self, df: pd.DataFrame, sport: str) -> bool:
        """Check all xG values are within valid range for sport."""
        pass
    
    def validate_completeness(self, df: pd.DataFrame) -> bool:
        """Check no NULL values in xG columns."""
        pass
    
    def calculate_correlation(self, df: pd.DataFrame, 
                             xg_col: str, actual_col: str) -> float:
        """Calculate Pearson correlation between xG and actual."""
        pass
    
    def generate_report(self, df: pd.DataFrame) -> Dict:
        """Generate comprehensive validation report."""
        pass
```

### 7. XGPipeline (Orchestrator)

```python
class XGPipeline:
    """Main orchestrator for xG generation across all sports."""
    
    def __init__(self, data_dir: str):
        self.data_dir = data_dir
        self.generators = {
            'FOOTBALL': FootballXGGenerator,
            'NBA': NBAXPointsGenerator,
            'NHL': NHLXGGenerator,
            'TENNIS': TennisXGamesGenerator,
        }
    
    def run(self) -> Dict[str, pd.DataFrame]:
        """Run xG generation for all sports."""
        pass
    
    def calculate_xg_diff(self, df: pd.DataFrame) -> pd.DataFrame:
        """Add xG_diff column to dataframe."""
        df['xG_diff'] = df['xG_home'] - df['xG_away']
        return df
    
    def export_results(self, output_dir: str) -> None:
        """Export updated datasets with xG values."""
        pass
```

## Data Models

### Team Statistics Cache

```python
@dataclass
class TeamStats:
    """Rolling team statistics for xG calculation."""
    team: str
    sport: str
    games_played: int
    goals_scored: float      # Or points for NBA
    goals_conceded: float
    attack_strength: float   # Relative to league avg
    defense_strength: float  # Relative to league avg
    home_record: float       # Win % at home
    away_record: float       # Win % away
    last_updated: datetime
```

### xG Configuration

```python
@dataclass
class XGConfig:
    """Configuration for xG generation."""
    sport: str
    min_value: float
    max_value: float
    home_advantage: float
    league_average: float
    rolling_window: int = 10  # Games for rolling stats
    min_games_required: int = 5  # Min games before using team stats
```

### Sport-Specific Configurations

| Sport | Min xG | Max xG | Home Adv | League Avg | Rolling Window |
|-------|--------|--------|----------|------------|----------------|
| Football | 0.0 | 6.0 | 1.20 | 1.35 goals | 10 games |
| NBA | 85.0 | 135.0 | 3.5 pts | 110 pts | 10 games |
| NHL | 1.5 | 5.0 | 1.10 | 3.0 goals | 10 games |
| Tennis | 0.0 | 40.0 | 1.05 | 12 games | 5 matches |

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system-essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Sport-Specific xG Bounds
*For any* generated xG value (xG_home or xG_away), the value must be within the valid range for that sport: Football [0.0, 6.0], NBA [85.0, 135.0], NHL [1.5, 5.0], Tennis [0.0, 40.0].
**Validates: Requirements 1.1, 2.1, 3.1, 4.1, 5.2**

### Property 2: xG_diff Arithmetic Correctness
*For any* match with xG_home and xG_away values, xG_diff must equal exactly (xG_home - xG_away) within floating-point tolerance.
**Validates: Requirements 1.4, 2.4, 3.4, 4.4, 6.1**

### Property 3: No NULL Values in xG Columns
*For any* row in the final dataset, xG_home, xG_away, and xG_diff columns must not contain NULL/NaN values.
**Validates: Requirements 5.1, 6.4**

### Property 4: NHL Existing xG Preservation
*For any* NHL match that has existing valid xG values (non-NULL, within range), the generator must preserve these values unchanged.
**Validates: Requirements 3.5**

### Property 5: Tennis Surface Variance Adjustment
*For any* set of tennis matches, the standard deviation of xG values for clay surface matches should be higher than for grass surface matches (variance ratio > 1.0).
**Validates: Requirements 4.5**

### Property 6: xG_diff Predictive Direction
*For any* sufficiently large sample of matches (n > 100), matches with positive xG_diff should have a higher home win rate than matches with negative xG_diff.
**Validates: Requirements 6.3**

### Property 7: Required Columns Exist
*For any* output dataset, the columns xG_home, xG_away, and xG_diff must exist and be of numeric type.
**Validates: Requirements 6.4**

## Error Handling

### Generation Errors
- **Missing team data**: Use league average as fallback, log warning
- **Invalid date**: Skip match, log error with match details
- **Calculation overflow**: Clamp to valid range, log warning

### Validation Errors
- **Out of bounds xG**: Clamp to valid range, log original value
- **NULL after generation**: Use fallback calculation, raise warning
- **Correlation below threshold**: Log warning, suggest recalibration

### Data Quality Issues
- **Insufficient history**: Use league averages for teams with < 5 games
- **Missing Elo ratings**: Calculate from win/loss record
- **Unknown surface (Tennis)**: Default to 'Hard' surface parameters

## Testing Strategy

### Unit Testing Framework
- **Framework**: pytest
- **Location**: `tests/test_xg_generators/`
- **Coverage target**: >85%

Unit tests will cover:
- Individual generator methods
- Edge cases (new teams, missing data, boundary values)
- Fallback calculations
- Configuration validation

### Property-Based Testing Framework
- **Framework**: Hypothesis (Python)
- **Configuration**: Minimum 100 iterations per property
- **Location**: `tests/test_xg_properties/`

Each correctness property will be implemented as a property-based test:
- Generate random valid inputs using Hypothesis strategies
- Verify the property holds for all generated inputs
- Tag each test with the property number and requirement reference

Example test annotation:
```python
@given(
    xg_home=st.floats(min_value=0.0, max_value=6.0),
    xg_away=st.floats(min_value=0.0, max_value=6.0)
)
def test_xg_diff_arithmetic(xg_home, xg_away):
    """
    **Feature: xg-generators, Property 2: xG_diff Arithmetic Correctness**
    **Validates: Requirements 1.4, 2.4, 3.4, 4.4, 6.1**
    """
    xg_diff = calculate_xg_diff(xg_home, xg_away)
    assert abs(xg_diff - (xg_home - xg_away)) < 1e-9
```

### Integration Tests
- End-to-end pipeline tests with real dataset samples
- Correlation validation against actual results
- Performance benchmarks for large datasets

## Algorithm Details

### Football xG Model (Dixon-Coles Inspired)

```
1. Calculate league average goals per game (home/away separately)
2. For each team, calculate:
   - Attack strength = team_goals_scored / league_avg_goals
   - Defense strength = team_goals_conceded / league_avg_goals
3. For match prediction:
   - xG_home = league_avg_home * attack_home * defense_away * home_factor
   - xG_away = league_avg_away * attack_away * defense_home
4. Apply Poisson smoothing for realistic distribution
```

### NBA xPoints Model (Four Factors Inspired)

```
1. Calculate team offensive/defensive ratings (pts per 100 possessions)
2. Calculate pace factor (possessions per game)
3. For match prediction:
   - expected_possessions = (pace_home + pace_away) / 2
   - xPoints_home = expected_poss * (off_rtg_home + 100 - def_rtg_away) / 200 + home_adv
   - xPoints_away = expected_poss * (off_rtg_away + 100 - def_rtg_home) / 200
```

### NHL xG Model (Shot Quality Based)

```
1. Calculate team shot generation and quality metrics
2. Calculate goaltender save percentages
3. For match prediction:
   - xG_home = shots_home * quality_home * (1 - sv%_away) * home_factor
   - xG_away = shots_away * quality_away * (1 - sv%_home)
4. Preserve existing xG values where available
```

### Tennis xGames Model (Elo + Surface)

```
1. Calculate win probability from Elo difference:
   - win_prob = 1 / (1 + 10^((elo_away - elo_home) / 400))
2. Estimate expected sets based on win probability
3. Calculate expected games per set based on dominance
4. Apply surface variance multiplier
5. Apply fatigue factor for recent match load
```

