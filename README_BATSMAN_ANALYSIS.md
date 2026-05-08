# IPL Batsman Analysis - Line-by-Line Explanation

## Overview

This notebook performs detailed analysis of IPL batsman performance using Pandas, Matplotlib, and Seaborn. It includes efficiency analysis, run prediction, and risk-adjusted performance metrics.

---

## Cell 1: Basic Batsman Efficiency Analysis

### Imports
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```

**Explanation:**
- `import pandas as pd` - Imports Pandas library aliased as `pd`. Used for data manipulation and DataFrame operations.
- `import matplotlib.pyplot as plt` - Imports pyplot module from Matplotlib. Used for creating visualizations and plots.
- `import seaborn as sns` - Imports Seaborn library aliased as `sns`. Builds on Matplotlib for enhanced statistical visualizations and themes.

### Load Data
```python
deliveries = pd.read_csv('deliveries.csv')
```

**Explanation:**
- `pd.read_csv()` - Reads CSV file into a Pandas DataFrame
- `'deliveries.csv'` - File path. Must be in the same directory as the notebook
- `deliveries` - Variable storing the loaded data with all ball-by-ball delivery records
- Returns a DataFrame with columns like match_id, batter, batsman_runs, etc.

### Function Definition
```python
def analyze_batsman_efficiency():
```

**Explanation:**
- `def` - Keyword to define a function
- `analyze_batsman_efficiency()` - Function name. Encapsulates the entire analysis workflow
- Allows code reusability and organized structure

### User Input
```python
target_batsman = input("Enter the Batsman Name (e.g., BB McCullum): ")
```

**Explanation:**
- `input()` - Built-in Python function that accepts user input from console
- `"Enter the Batsman Name..."` - Prompt displayed to user
- `target_batsman` - Variable storing the batsman name entered by user
- Example: If user types "V Kohli", then `target_batsman = "V Kohli"`

### Input Validation
```python
if target_batsman not in deliveries['batter'].unique():
    print("Batsman not found in dataset. Please check spelling.")
    return
```

**Explanation:**
- `deliveries['batter']` - Accesses the 'batter' column from the DataFrame
- `.unique()` - Returns array of unique batsman names (removes duplicates)
- `not in` - Checks if `target_batsman` is NOT in the list of unique batsmen
- `if` - Conditional statement. Executes if batter not found
- `print()` - Displays error message to user
- `return` - Exits the function immediately without further execution
- **Purpose:** Prevents errors if user enters non-existent batsman name

### Filter Data for Selected Batsman
```python
df_batter = deliveries[deliveries['batter'] == target_batsman].copy()
```

**Explanation:**
- `deliveries['batter'] == target_batsman` - Boolean mask: True where batter matches, False elsewhere
- `deliveries[...]` - Boolean indexing: selects only rows where condition is True
- `.copy()` - Creates independent copy of filtered data (avoids warnings about modifying original)
- `df_batter` - New DataFrame containing ONLY deliveries by the selected batsman

### Define Match Phases
```python
def get_phase(over):
    if over < 6: return 'Powerplay'
    elif over < 15: return 'Middle'
    else: return 'Death'
```

**Explanation:**
- `def get_phase(over):` - Defines nested function that categorizes overs into phases
- `over` - Parameter: over number (0-19 in dataset)
- `if over < 6:` - Overs 0-5 are powerplay phase (first 6 overs)
- `return 'Powerplay'` - Returns string 'Powerplay' for early overs
- `elif over < 15:` - Overs 6-14 are middle overs (strategy phase)
- `return 'Middle'` - Returns string 'Middle'
- `else:` - Overs 15-19 (last 5 overs, death overs)
- `return 'Death'` - Returns string 'Death'
- **Cricket Context:** Different phases require different batting strategies

### Apply Phase Categorization
```python
df_batter['phase'] = df_batter['over'].apply(get_phase)
```

**Explanation:**
- `df_batter['over']` - Accesses the 'over' column
- `.apply(get_phase)` - Applies the `get_phase()` function to EACH value in the 'over' column
- Creates new column 'phase' with phase labels ('Powerplay', 'Middle', 'Death')
- **Result:** Each delivery row now has a 'phase' classification

### Aggregate Over-wise Statistics
```python
over_stats = df_batter.groupby('over').agg({'batsman_runs': 'sum', 'ball': 'count'})
```

**Explanation:**
- `.groupby('over')` - Groups all deliveries by over number
  - Groups deliveries from over 0 together, over 1 together, etc.
- `.agg()` - Aggregates (summarizes) each group
  - `{'batsman_runs': 'sum'}` - Sums all runs in each over
  - `'ball': 'count'` - Counts total balls faced in each over
- `over_stats` - DataFrame with rows = overs, columns = aggregated statistics
- **Result:** Shows total runs and balls faced for each of the 20 overs

### Calculate Strike Rate
```python
over_stats['strike_rate'] = (over_stats['batsman_runs'] / over_stats['ball']) * 100
```

**Explanation:**
- `over_stats['batsman_runs']` - Series of total runs per over
- `over_stats['ball']` - Series of total balls faced per over
- `over_stats['batsman_runs'] / over_stats['ball']` - Vectorized division: runs per ball
- `* 100` - Converts to percentage (standard cricket metric)
- Creates new column 'strike_rate' with calculated values
- **Formula:** Strike Rate = (Runs / Balls) × 100
- **Example:** If 20 runs in 16 balls → SR = (20/16) × 100 = 125

### Aggregate Phase-wise Statistics
```python
phase_stats = df_batter.groupby('phase').agg({'batsman_runs': 'sum', 'ball': 'count'})
```

**Explanation:**
- `.groupby('phase')` - Groups deliveries by phase ('Powerplay', 'Middle', 'Death')
- `.agg()` - Aggregates runs and ball counts for each phase
- `phase_stats` - DataFrame with rows = phases, columns = aggregated stats
- **Result:** Summary statistics for each match phase

### Calculate Phase-wise Strike Rate
```python
phase_stats['strike_rate'] = (phase_stats['batsman_runs'] / phase_stats['ball']) * 100
```

**Explanation:**
- Same calculation as over-wise, but at phase level
- Creates 'strike_rate' column showing efficiency in each phase
- Allows comparison: Is batsman aggressive in powerplay vs death?

### Reorder Phases Logically
```python
phase_order = ['Powerplay', 'Middle', 'Death']
phase_stats = phase_stats.reindex(phase_order)
```

**Explanation:**
- `phase_order` - List defining desired order of phases
- `.reindex(phase_order)` - Reorders rows to match specified order
- Without this, order might be alphabetical or random
- **Result:** Phases appear in chronological order (Powerplay → Middle → Death)

### Create Visualization Layout
```python
fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(14, 10))
```

**Explanation:**
- `plt.subplots()` - Creates figure and subplots
  - `2, 1` - Creates 2 rows, 1 column of subplots
  - Total: 2 plots stacked vertically
- `figsize=(14, 10)` - Sets figure size to 14 inches wide × 10 inches tall
- `fig` - Figure object (overall canvas)
- `(ax1, ax2)` - Tuple of two axes (individual plot areas)

### Plot 1: Over-wise Heatmap
```python
sns.heatmap(over_stats[['strike_rate']].T, annot=True, fmt=".1f", 
            cmap='RdYlGn', ax=ax1, cbar_kws={'label': 'Strike Rate'})
```

**Explanation Line by Line:**
- `over_stats[['strike_rate']]` - Selects strike_rate column (double brackets keep as DataFrame)
- `.T` - Transposes data: converts rows to columns (20 overs become columns)
- `sns.heatmap()` - Creates color-coded matrix visualization
- `annot=True` - Displays actual values inside cells
- `fmt=".1f"` - Formats numbers with 1 decimal place (e.g., 125.5)
- `cmap='RdYlGn'` - Color map: Red (low) → Yellow (medium) → Green (high)
- `ax=ax1` - Draws on first subplot
- `cbar_kws={'label': 'Strike Rate'}` - Adds label to color bar

**Visual Result:** Shows each over's strike rate with color coding. Green = aggressive, Red = defensive.

### Set Plot 1 Title and Labels
```python
ax1.set_title(f'Over-by-Over Strategic Strike Zone: {target_batsman}', fontsize=15)
ax1.set_xlabel('Over Number')
ax1.set_ylabel('')
```

**Explanation:**
- `f'...'` - F-string allows variable insertion
- `{target_batsman}` - Inserts batsman name into title dynamically
- `fontsize=15` - Sets title font size to 15 points
- `set_xlabel()` - Labels x-axis as 'Over Number'
- `set_ylabel('')` - Removes y-axis label (empty string)

### Plot 2: Phase-wise Heatmap
```python
sns.heatmap(phase_stats[['strike_rate']].T, annot=True, fmt=".1f", 
            cmap='RdYlGn', ax=ax2, cbar_kws={'label': 'Strike Rate'})
```

**Explanation:**
- Same as Plot 1 but for phase data
- Only 3 columns (Powerplay, Middle, Death) instead of 20 (overs)
- Shows aggregated efficiency by phase
- `ax=ax2` - Draws on second subplot

### Set Plot 2 Labels
```python
ax2.set_title(f'Phase-wise Efficiency: {target_batsman}', fontsize=15)
ax2.set_xlabel('Match Phase')
ax2.set_ylabel('')
```

**Explanation:**
- Title shows "Phase-wise Efficiency"
- X-axis labeled as 'Match Phase'
- Y-axis label removed for cleaner look

### Finalize and Display
```python
plt.tight_layout()
plt.show()
```

**Explanation:**
- `plt.tight_layout()` - Automatically adjusts spacing between subplots to prevent overlap
- `plt.show()` - Displays the figure in Jupyter/console
- Without this, visualization wouldn't appear

### Execute Function
```python
analyze_batsman_efficiency()
```

**Explanation:**
- Calls the function to run the entire analysis
- Prompts user for input and executes workflow
- Displays visualizations

---

## Cell 2: Predict Runs in Specific Over

### Imports
```python
import pandas as pd
import numpy as np
```

**Explanation:**
- `pandas` - Data manipulation
- `numpy` - Numerical computations and statistical functions

### Load Data
```python
deliveries = pd.read_csv('deliveries.csv')
```

**Explanation:**
- Same as Cell 1: loads ball-by-ball delivery data

### Function Definition
```python
def predict_runs_in_over():
```

**Explanation:**
- Defines function to predict runs in a specific over based on historical data

### Get User Inputs
```python
batter_name = input("Enter Batsman Name: ")
target_over = int(input("Enter Over Number to predict (0-19): "))
```

**Explanation:**
- `batter_name` - Gets batsman name from user
- `target_over` - Gets over number from user
- `int()` - Converts user input from string to integer (necessary for comparison)

### Filter Historical Data
```python
context_data = deliveries[(deliveries['batter'] == batter_name) & 
                          (deliveries['over'] == target_over)]
```

**Explanation:**
- `deliveries['batter'] == batter_name` - Boolean mask for matching batter
- `deliveries['over'] == target_over` - Boolean mask for matching over
- `&` - AND operator: both conditions must be True
- Selects all historical deliveries where batter faced this specific over
- **Purpose:** Get historical performance in this over context

### Check for Data Availability
```python
if context_data.empty:
    print(f"No historical data available for {batter_name} in over {target_over}.")
    return
```

**Explanation:**
- `.empty` - Returns True if DataFrame has no rows
- If batsman never faced this over, cannot make prediction
- `return` - Exits function early

### Calculate Historical Metrics
```python
total_runs = context_data['batsman_runs'].sum()
total_balls = context_data['ball'].count()
```

**Explanation:**
- `context_data['batsman_runs'].sum()` - Sum of runs in all instances of this over
- `context_data['ball'].count()` - Count of balls faced in all instances of this over
- **Example:** If batsman faced over 10 in 5 different matches, sums across all 5

### Calculate Runs Per Ball
```python
runs_per_ball = total_runs / total_balls
```

**Explanation:**
- Divides total runs by total balls
- Gives average runs per delivery in this over
- **Example:** 15 runs in 18 balls (3 instances × 6 balls) → 0.833 runs/ball

### Calculate Strike Rate
```python
strike_rate = runs_per_ball * 100
```

**Explanation:**
- Converts to percentage
- Strike Rate = (Runs Per Ball) × 100
- **Example:** 0.833 × 100 = 83.3 SR

### Predict Full Over
```python
predicted_runs = runs_per_ball * 6
```

**Explanation:**
- Multiplies runs per ball by 6 (number of balls in an over)
- Predicts total runs if batsman faces all 6 balls
- **Example:** 0.833 × 6 = 5 predicted runs

### Calculate Consistency (Risk Assessment)
```python
match_wise_runs = context_data.groupby('match_id')['batsman_runs'].sum()
consistency_score = match_wise_runs.std()
```

**Explanation:**
- `.groupby('match_id')` - Groups deliveries by match
- `.sum()` - Sums runs per match
- `.std()` - Calculates standard deviation (measure of variability)
- High std = inconsistent (unpredictable), Low std = consistent (predictable)
- **Cricket Insight:** Shows whether performance in this over is reliable or volatile

### Display Results
```python
print(f"\n--- Prediction Report for {batter_name} (Over {target_over}) ---")
print(f"Historical Strike Rate in this over: {strike_rate:.2f}")
print(f"Expected Runs (if facing 6 balls): {predicted_runs:.1f}")
```

**Explanation:**
- `f"..."` - F-strings for variable interpolation
- `{strike_rate:.2f}` - Formats to 2 decimal places
- `{predicted_runs:.1f}` - Formats to 1 decimal place
- `\n` - Newline character for spacing

### Categorize Batting Style
```python
if predicted_runs > 10:
    trend = "Aggressive (High Boundary Risk)"
elif predicted_runs > 6:
    trend = "Steady (Strike Rotation)"
else:
    trend = "Defensive (Pressure Building)"
```

**Explanation:**
- `if predicted_runs > 10:` - If expected > 10 runs, batsman is aggressive
- `elif predicted_runs > 6:` - If between 6-10, steady approach
- `else:` - If < 6, defensive approach
- `trend` - Variable storing categorization

### Print Analysis
```python
print(f"Predicted Batting Style: {trend}")
print(f"Confidence Interval: +/- {consistency_score:.1f} runs")
```

**Explanation:**
- Displays predicted batting style based on runs
- Shows confidence interval (uncertainty range)
- Higher std = wider interval = less certain prediction

### Execute Function
```python
predict_runs_in_over()
```

**Explanation:**
- Runs the prediction function with user inputs

---

## Cell 3: Random Forest Run Prediction

### Imports
```python
import pandas as pd
import numpy as np
from sklearn.ensemble import RandomForestRegressor
from sklearn.preprocessing import LabelEncoder
from sklearn.model_selection import train_test_split
```

**Explanation:**
- `RandomForestRegressor` - Machine learning model for numerical predictions
- `LabelEncoder` - Converts categorical data (names) to numerical codes
- `train_test_split` - Splits data into training/testing sets (not used in this code)

### Load Data
```python
deliveries = pd.read_csv('deliveries.csv')
```

**Explanation:**
- Same as previous: loads delivery data

### Aggregate to Over Level
```python
over_data = deliveries.groupby(['match_id', 'batter', 'batting_team', 'over']).agg({
    'batsman_runs': 'sum'
}).reset_index()
```

**Explanation:**
- `.groupby(['match_id', 'batter', 'batting_team', 'over'])` - Groups by 4 dimensions
- `.agg({'batsman_runs': 'sum'})` - Sums runs for each group
- `.reset_index()` - Converts grouped result back to flat DataFrame
- **Result:** One row per unique combination of (match, batter, team, over)

### Encode Categorical Data - Batter
```python
le_batter = LabelEncoder()
over_data['batter_id'] = le_batter.fit_transform(over_data['batter'])
```

**Explanation:**
- `LabelEncoder()` - Creates encoder object
- `.fit_transform()` - Learns mapping and applies it
  - "V Kohli" → 0, "AB de Villiers" → 1, etc.
- `batter_id` - New column with numeric codes instead of names
- **Why?** Machine learning models require numerical inputs

### Encode Categorical Data - Team
```python
le_team = LabelEncoder()
over_data['team_id'] = le_team.fit_transform(over_data['batting_team'])
```

**Explanation:**
- Same encoding for team names
- "MI" → 0, "CSK" → 1, etc.

### Create Features and Target
```python
X = over_data[['batter_id', 'team_id', 'over']]
y = over_data['batsman_runs']
```

**Explanation:**
- `X` - Features (input variables): batter_id, team_id, over number
- `y` - Target (what we predict): batsman_runs
- Machine learning learns relationship: given batter/team/over, predict runs

### Train Random Forest Model
```python
model = RandomForestRegressor(n_estimators=100, random_state=42)
model.fit(X, y)
```

**Explanation:**
- `RandomForestRegressor()` - Creates model
  - `n_estimators=100` - Uses 100 decision trees
  - `random_state=42` - Seed for reproducibility
- `.fit(X, y)` - Trains model on data
  - Learns patterns from historical data
  - How runs depend on batter, team, and over number

### Define Prediction Function
```python
def predict_batsman_performance():
    print("\n--- Random Forest Run Predictor ---")
    name = input("Enter Batsman Name: ")
    over_num = int(input("Enter Over Number (0-19): "))
```

**Explanation:**
- Gets user input for batter name and over number

### Convert Name to ID
```python
name_id = le_batter.transform([name])[0]
```

**Explanation:**
- `le_batter.transform([name])` - Converts name to numeric ID using trained encoder
- `[0]` - Extracts first (only) value from result
- **Note:** Must use same encoder that trained the model

### Get Team ID
```python
team_id = over_data[over_data['batter'] == name]['team_id'].iloc[0]
```

**Explanation:**
- `over_data[over_data['batter'] == name]` - Filters rows for this batter
- `['team_id']` - Gets team_id column
- `.iloc[0]` - Gets first value
- Assumes batter played for only one team (reasonable simplification)

### Make Prediction
```python
prediction = model.predict([[name_id, team_id, over_num]])
```

**Explanation:**
- `model.predict()` - Uses trained model to predict
- `[[name_id, team_id, over_num]]` - Input features as 2D array
- `prediction` - Array containing predicted runs

### Display Prediction
```python
print(f"\nPrediction for {name} in Over {over_num}:")
print(f"Expected Runs: {prediction[0]:.2f}")
```

**Explanation:**
- `prediction[0]` - Gets first (only) prediction value
- `:.2f` - Formats to 2 decimal places
- Shows predicted runs for user query

### Error Handling
```python
except ValueError:
    print("Error: Batsman name not found in dataset. Please check spelling.")
```

**Explanation:**
- `except ValueError` - Catches error if name not in encoder
- Displays user-friendly error message

### Execute Prediction
```python
predict_batsman_performance()
```

**Explanation:**
- Runs the prediction function

---

## Cell 4: Enhanced Efficiency Analysis (4 Plots)

### Imports and Load Data
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

deliveries = pd.read_csv('deliveries.csv')
```

**Explanation:**
- Same as Cell 1

### Function Definition and Input
```python
def analyze_batsman_efficiency():
    target_batsman = input("Enter the Batsman Name (e.g., BB McCullum): ")
    
    if target_batsman not in deliveries['batter'].unique():
        print("Batsman not found in dataset. Please check spelling.")
        return
```

**Explanation:**
- Same validation as Cell 1

### Data Preparation and Phase Categorization
```python
df_batter = deliveries[deliveries['batter'] == target_batsman].copy()

def get_phase(over):
    if over < 6: return 'Powerplay'
    elif over < 15: return 'Middle'
    else: return 'Death'

df_batter['phase'] = df_batter['over'].apply(get_phase)
```

**Explanation:**
- Same as Cell 1

### Aggregate Over-wise Data with Reset
```python
over_stats = df_batter.groupby('over').agg({'batsman_runs': 'sum', 'ball': 'count'}).reset_index()
```

**Explanation:**
- `.reset_index()` - Converts grouped result to regular DataFrame
- Makes 'over' a column instead of index (better for later operations)
- Result: columns are ['over', 'batsman_runs', 'ball']

### Calculate Over-wise Strike Rate
```python
over_stats['strike_rate'] = (over_stats['batsman_runs'] / over_stats['ball']) * 100
```

**Explanation:**
- Same calculation as before

### Aggregate Phase-wise Data
```python
phase_stats = df_batter.groupby('phase').agg({'batsman_runs': 'sum', 'ball': 'count'})
phase_stats['strike_rate'] = (phase_stats['batsman_runs'] / phase_stats['ball']) * 100

phase_order = ['Powerplay', 'Middle', 'Death']
phase_stats = phase_stats.reindex(phase_order)
```

**Explanation:**
- Same as Cell 1

### Create 3-Subplot Layout
```python
fig, (ax1, ax2, ax3) = plt.subplots(3, 1, figsize=(14, 15))
```

**Explanation:**
- `3, 1` - 3 rows, 1 column
- `figsize=(14, 15)` - Taller figure to accommodate 3 plots

### Plot 1: Over-wise Strike Rate Heatmap
```python
sns.heatmap(over_stats.set_index('over')[['strike_rate']].T, annot=True, fmt=".1f", 
            cmap='RdYlGn', ax=ax1, cbar_kws={'label': 'Strike Rate'})
```

**Explanation:**
- `over_stats.set_index('over')` - Makes 'over' the index (row labels)
- `[['strike_rate']]` - Selects only strike_rate column
- `.T` - Transposes (overs as columns)
- `annot=True` - Shows values in cells
- `fmt=".1f"` - 1 decimal place format
- `cmap='RdYlGn'` - Red-Yellow-Green color map

### Set Plot 1 Labels
```python
ax1.set_title(f'Over-by-Over Strategic Strike Zone: {target_batsman}', fontsize=15)
ax1.set_xlabel('Over Number')
ax1.set_ylabel('')
```

**Explanation:**
- Same as Cell 1

### Plot 2: Balls Faced per Over (NEW)
```python
sns.barplot(data=over_stats, x='over', y='ball', ax=ax2, palette='viridis')
ax2.set_title(f'Volume of Balls Faced per Over: {target_batsman}', fontsize=15)
ax2.set_xlabel('Over Number')
ax2.set_ylabel('Total Balls Faced')
```

**Explanation:**
- `sns.barplot()` - Creates bar chart
- `data=over_stats` - DataFrame source
- `x='over', y='ball'` - Over numbers on x-axis, balls on y-axis
- `palette='viridis'` - Color scheme
- **Purpose:** Shows how many balls batsman faced in each over (higher = more plays in that over)

### Add Value Labels on Bars
```python
for p in ax2.patches:
    ax2.annotate(format(p.get_height(), '.0f'), 
                 (p.get_x() + p.get_width() / 2., p.get_height()), 
                 ha = 'center', va = 'center', 
                 xytext = (0, 9), 
                 textcoords = 'offset points')
```

**Explanation:**
- `for p in ax2.patches:` - Loops through each bar rectangle
- `p.get_height()` - Gets height (value) of bar
- `format(..., '.0f')` - Formats as whole number
- `(p.get_x() + p.get_width() / 2., p.get_height())` - Position: center top of bar
- `ax2.annotate()` - Adds text label
- `ha='center', va='center'` - Horizontal and vertical alignment
- `xytext=(0, 9)` - Offset label 9 points above bar
- **Result:** Numbers appear on top of bars

### Plot 3: Phase-wise Efficiency Heatmap
```python
sns.heatmap(phase_stats[['strike_rate']].T, annot=True, fmt=".1f", 
            cmap='RdYlGn', ax=ax3, cbar_kws={'label': 'Strike Rate'})
ax3.set_title(f'Phase-wise Efficiency: {target_batsman}', fontsize=15)
ax3.set_xlabel('Match Phase')
ax3.set_ylabel('')
```

**Explanation:**
- Same heatmap approach but for 3 phases
- Shows strike rate aggregated by phase

### Finalize Display
```python
plt.tight_layout()
plt.show()

analyze_batsman_efficiency()
```

**Explanation:**
- Adjusts spacing and displays all 3 plots

---

## Cell 5: Risk-Adjusted Efficiency Analysis

### Imports
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
```

**Explanation:**
- All libraries needed for advanced analysis

### Load and Check Data
```python
try:
    deliveries = pd.read_csv('deliveries.csv')
except FileNotFoundError:
    print("Error: 'deliveries.csv' not found...")
    return

if 'player_dismissed' not in deliveries.columns:
    print("Error: Dataset must contain a 'player_dismissed' column...")
    return
```

**Explanation:**
- `try-except` - Handles file not found error gracefully
- `if 'player_dismissed' not in...` - Checks required column exists
- Dismissal data needed to calculate risk metrics

### Function Definition and Input
```python
def analyze_risk_adjusted_efficiency():
    target_batsman = input("Enter the Batsman Name (e.g., V Kohli): ")
    
    if target_batsman not in deliveries['batter'].unique():
        print("Batsman not found in dataset. Please check spelling.")
        return
```

**Explanation:**
- Same input validation

### Data Preparation
```python
df_batter = deliveries[deliveries['batter'] == target_batsman].copy()

df_batter['is_dismissed'] = (df_batter['player_dismissed'] == target_batsman).astype(int)
```

**Explanation:**
- Filters to selected batsman
- `df_batter['player_dismissed'] == target_batsman` - Boolean: True if batsman was out
- `.astype(int)` - Converts True→1, False→0
- `is_dismissed` - New column: 1 = out on this delivery, 0 = not out

### Add Phase Categorization
```python
def get_phase(over):
    if over < 6: return 'Powerplay'
    elif over < 15: return 'Middle'
    else: return 'Death'

df_batter['phase'] = df_batter['over'].apply(get_phase)
```

**Explanation:**
- Same as before

### Aggregate Over-wise with Risk Metrics
```python
over_stats = df_batter.groupby('over').agg(
    runs=('batsman_runs', 'sum'),
    balls=('ball', 'count'),
    dismissals=('is_dismissed', 'sum')
).reset_index()
```

**Explanation:**
- `agg()` with named aggregations (more readable):
  - `runs=('batsman_runs', 'sum')` - Column 'runs' = sum of batsman_runs
  - `balls=('ball', 'count')` - Column 'balls' = count of deliveries
  - `dismissals=('is_dismissed', 'sum')` - Column 'dismissals' = count of outs
- Result: over-wise runs, balls, and dismissals

### Calculate Performance Metrics
```python
over_stats['strike_rate'] = (over_stats['runs'] / over_stats['balls']) * 100
```

**Explanation:**
- Standard strike rate calculation

### Calculate Risk Metrics
```python
over_stats['dismissal_rate'] = over_stats['dismissals'] / over_stats['balls']
over_stats['survival_rate'] = 1 - over_stats['dismissal_rate']
```

**Explanation:**
- `dismissal_rate` - Probability of being out per ball
  - Example: 1 out in 20 balls = 5% dismissal rate
- `survival_rate` - Probability of NOT being out
  - Example: 1 - 0.05 = 0.95 (95% survival)
- **Cricket Insight:** Risk-reward balance

### Calculate Survival-Adjusted Strike Rate (SASR)
```python
over_stats['sasr'] = over_stats['strike_rate'] * over_stats['survival_rate']
```

**Explanation:**
- **SASR = Strike Rate × Survival Rate**
- Combines aggression (strike rate) with reliability (survival)
- High SASR = aggressive AND consistent
- Low SASR = either defensive or risky
- **Example:** SR=150 with 80% survival → SASR = 150 × 0.8 = 120

### Phase-wise Aggregation
```python
phase_stats = df_batter.groupby('phase').agg(
    runs=('batsman_runs', 'sum'),
    balls=('ball', 'count'),
    dismissals=('is_dismissed', 'sum')
)
phase_stats['strike_rate'] = (phase_stats['runs'] / phase_stats['balls']) * 100
phase_stats['survival_rate'] = 1 - (phase_stats['dismissals'] / phase_stats['balls'])
phase_stats['sasr'] = phase_stats['strike_rate'] * phase_stats['survival_rate']

phase_order = ['Powerplay', 'Middle', 'Death']
phase_stats = phase_stats.reindex(phase_order)
```

**Explanation:**
- Same calculations but aggregated by phase
- Shows which phase is riskiest (lowest survival rate)

### Create 4-Plot Visualization
```python
fig, (ax1, ax2, ax3, ax4) = plt.subplots(4, 1, figsize=(15, 24))
```

**Explanation:**
- 4 rows, 1 column
- `figsize=(15, 24)` - Very tall for 4 plots

### Plot 1: Raw Strike Rate
```python
sns.barplot(data=over_stats, x='over', y='strike_rate', hue='over', ax=ax1, palette='mako', legend=False)
ax1.set_title(f'Raw Reward: Standard Strike Rate per Over ({target_batsman})', fontsize=14, fontweight='bold')
ax1.set_xlabel('Over Number')
ax1.set_ylabel('Strike Rate')
ax1.axhline(100, color='grey', linestyle='--', alpha=0.5)
```

**Explanation:**
- Bar chart showing strike rate by over
- `hue='over'` - Each bar gets different color (visual variety)
- `legend=False` - Removes legend (too many colors)
- `ax1.axhline(100)` - Horizontal line at 100 SR benchmark
  - Dotted line (linestyle='--')
  - Semi-transparent (alpha=0.5)

### Plot 2: Dismissal Probability
```python
over_stats['out_prob_pct'] = over_stats['dismissal_rate'] * 100
sns.barplot(data=over_stats, x='over', y='out_prob_pct', hue='over', ax=ax2, palette='Reds', legend=False)
ax2.set_title(f'Cost of Aggression: Probability of Dismissal per Ball (%)', fontsize=14, fontweight='bold')
ax2.set_xlabel('Over Number')
ax2.set_ylabel('Dismissal Probability (%)')
```

**Explanation:**
- Converts dismissal_rate to percentage
- Shows which overs are risky (higher dismissal probability)
- Red palette emphasizes risk

### Add Data Labels
```python
for p in ax2.patches:
    height = p.get_height()
    if height > 0:
        ax2.annotate(f'{height:.1f}%', 
                     (p.get_x() + p.get_width() / 2., height), 
                     ha='center', va='bottom', fontsize=9)
```

**Explanation:**
- Adds percentage labels on top of bars
- `if height > 0:` - Only label non-zero bars
- `f'{height:.1f}%'` - Format as percentage with 1 decimal

### Plot 3: Phase-wise Comparison
```python
phase_melted = phase_stats[['strike_rate', 'sasr']].reset_index().melt(id_vars='phase', var_name='Metric', value_name='Value')
sns.barplot(data=phase_melted, x='phase', y='Value', hue='Metric', ax=ax3, palette=['#3498db', '#2ecc71'])
ax3.set_title(f'Phase-wise Breakdown: Illusion (Raw SR) vs Reality (SASR)', fontsize=14, fontweight='bold')
ax3.set_xlabel('Match Phase')
ax3.set_ylabel('Rating')
```

**Explanation:**
- `.melt()` - Reshapes data for easier comparison
  - Converts columns (strike_rate, sasr) to rows
  - Creates 'Metric' column showing which metric each row is
- `hue='Metric'` - Groups bars by metric
- Allows side-by-side comparison of raw vs adjusted metrics
- **Insight:** Shows how much dismissal risk reduces actual efficiency

### Customize Legend
```python
handles, labels = ax3.get_legend_handles_labels()
ax3.legend(handles=handles, labels=['Raw Strike Rate', 'Survival-Adjusted SR (SASR)'], title='Metric')
```

**Explanation:**
- Gets legend information
- Relabels with more descriptive names
- Explains the difference between metrics

### Plot 4: Master Composite
```python
scatter = sns.scatterplot(data=over_stats, x='over', y='sasr', 
                          size='balls', sizes=(100, 1500), 
                          hue='sasr', palette='RdYlGn', 
                          edgecolor='black', linewidth=1.5,
                          ax=ax4, legend=False)
```

**Explanation:**
- Scatter plot (points instead of bars)
- `x='over', y='sasr'` - Over number vs SASR
- `size='balls'` - Bubble size represents balls faced
  - Larger bubble = batsman faced more deliveries in that over
  - Indicates reliability (played more balls)
- `sizes=(100, 1500)` - Range of bubble sizes
- `hue='sasr'` - Color by SASR value
  - Green = high SASR (efficient and safe)
  - Red = low SASR (risky)
- `edgecolor='black', linewidth=1.5` - Black border around bubbles

### Add Title and Labels
```python
ax4.set_title(f'Composite Efficiency: Survival-Adjusted SR + Experience ({target_batsman})', fontsize=14, fontweight='bold')
ax4.set_xlabel('Over Number', fontsize=12)
ax4.set_ylabel('Survival-Adjusted Strike Rate (SASR)', fontsize=12)
```

**Explanation:**
- Title describes what the plot shows
- Larger font for readability

### Add Career Average Line
```python
overall_sr = (over_stats['runs'].sum() / over_stats['balls'].sum()) * 100
overall_survival = 1 - (over_stats['dismissals'].sum() / over_stats['balls'].sum())
career_sasr = overall_sr * overall_survival

ax4.axhline(career_sasr, color='black', linestyle='-.', label=f'Career Average SASR ({career_sasr:.1f})')
ax4.legend(loc='upper left')
```

**Explanation:**
- Calculates overall career SASR across all overs
- `overall_sr` - Career strike rate
- `overall_survival` - Career survival rate
- `career_sasr` - Overall SASR benchmark
- `ax4.axhline()` - Horizontal reference line
  - `linestyle='-.'` - Dash-dot line
- `ax4.legend()` - Shows what the line represents

### Add Bubble Labels
```python
for i in range(len(over_stats)):
    if over_stats['balls'].iloc[i] > 0:
        ax4.text(over_stats['over'].iloc[i], over_stats['sasr'].iloc[i], 
                 str(over_stats['balls'].iloc[i]), 
                 horizontalalignment='center', verticalalignment='center', 
                 color='black', fontsize=9, weight='bold')
```

**Explanation:**
- Loops through each over
- If batsman faced balls in that over:
  - Adds text showing number of balls faced
  - Positioned at center of bubble
  - Makes bubble size more interpretable

### Execute Function
```python
if __name__ == "__main__":
    analyze_risk_adjusted_efficiency()
```

**Explanation:**
- `if __name__ == "__main__":` - Runs only if script executed directly (not imported)
- Professional practice for reusable code

---

## Cell 6: Analysis with Missing Overs Handling

### Key Addition: Reindex for All Overs
```python
all_overs = pd.Index(range(20), name='over')
over_stats = over_stats.reindex(all_overs).fillna(0).reset_index()
```

**Explanation:**
- `pd.Index(range(20), name='over')` - Creates index for all 20 overs (0-19)
- `.reindex(all_overs)` - Forces DataFrame to include all overs
- `.fillna(0)` - Fills missing overs with 0 (batsman didn't bat in those overs)
- `.reset_index()` - Converts index back to regular column
- **Purpose:** Ensures visualization shows all 20 overs, even if batsman skipped some

### Calculate Strike Rate Safely
```python
over_stats['strike_rate'] = (over_stats['batsman_runs'] / over_stats['ball'] * 100).fillna(0)
```

**Explanation:**
- Calculates strike rate
- `.fillna(0)` - Fills NaN (0/0 = undefined) with 0
- Prevents errors when displaying overs with no data

### Log Transformation for Better Visualization
```python
over_stats['log_sr'] = np.log1p(over_stats['strike_rate'])
```

**Explanation:**
- `np.log1p()` - Natural log of (x + 1)
- Adding 1 prevents log(0) = undefined error
- **Purpose:** Reduces impact of outliers
  - Very high SR (e.g., 300) becomes manageable in visualization
  - Shows variation in normal range better

### Use Log Data for Heatmap Colors
```python
sns.heatmap(over_stats.set_index('over')[['log_sr']].T, 
            annot=over_stats.set_index('over')[['strike_rate']].T, 
            fmt=".1f", cmap='RdYlGn', ax=ax1, cbar=False)
```

**Explanation:**
- `log_sr` for color mapping (normalized scale)
- `strike_rate` for annotations (actual values shown)
- `cbar=False` - Removes color bar (redundant with log scale)
- Shows normalized colors but displays true values

---

## Cell 7: Normalized Heatmap with Advanced Features

### Comprehensive Reindexing
```python
all_overs = pd.DataFrame({'over': range(20)})

over_stats = pd.merge(all_overs, over_stats, on='over', how='left').fillna(0)
```

**Explanation:**
- Creates template DataFrame with all 20 overs
- `pd.merge(..., how='left')` - Left join ensures all overs included
- `fillna(0)` - Fills missing with 0
- More robust than reindex for complex scenarios

### Safe Strike Rate Calculation
```python
over_stats['strike_rate'] = np.where(
    over_stats['total_balls'] > 0, 
    (over_stats['total_runs'] / over_stats['total_balls']) * 100, 
    0
)
```

**Explanation:**
- `np.where(condition, if_true, if_false)` - Conditional assignment
- If balls > 0: calculate strike rate
- If balls = 0: assign 0
- Prevents division by zero errors

### Log Transformation for Color
```python
over_stats['color_intensity'] = np.log1p(over_stats['strike_rate'])
```

**Explanation:**
- Creates color column using log transformation
- Normalizes extreme values

### Advanced Visualization
```python
plt.style.use('seaborn-v0_8-muted')
fig = plt.figure(figsize=(15, 12))
gs = fig.add_gridspec(3, 1, height_ratios=[1.2, 2, 1.2])
```

**Explanation:**
- `plt.style.use()` - Applies professional style theme
- `fig.add_gridspec()` - Creates advanced grid layout
- `height_ratios=[1.2, 2, 1.2]` - Middle plot is 2× taller than top/bottom
- More control than subplots

### Dual Coloring Strategy
```python
sns.heatmap(
    over_stats.set_index('over')[['color_intensity']].T,  # For colors
    annot=over_stats.set_index('over')[['strike_rate']].T, # For text
    ...
)
```

**Explanation:**
- Colors based on log-transformed data (normalized)
- Text shows actual strike rate values
- Best of both worlds: good visualization + true values

---

## Cell 8: True Transformation Visualization

### Demonstrates Data Transformation
```python
deliveries = pd.read_csv('deliveries.csv')

def analyze_true_transformation():
    target_batsman = input("Enter Batsman Name: ")
    
    if target_batsman not in deliveries['batter'].unique():
        print("Batsman not found.")
        return
```

**Explanation:**
- Standard input validation

### Prepare Data with Reindex
```python
df_batter = deliveries[deliveries['batter'] == target_batsman].copy()
stats = df_batter.groupby('over').agg(
    runs=('batsman_runs', 'sum'),
    balls=('ball', 'count')
).reindex(range(20)).fillna(0)

stats['raw_sr'] = (stats['runs'] / stats['balls'] * 100).fillna(0)
```

**Explanation:**
- Filters to batsman data
- Groups by over with named aggregations
- Reindexes to include all 20 overs
- Calculates strike rate

### Create Comparison Visualization
```python
fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(15, 8))
```

**Explanation:**
- Two plots to show before/after transformation

### Plot 1: Before (Unrestricted Scale)
```python
sns.heatmap(stats[['raw_sr']].T, annot=True, fmt=".1f", 
            cmap='RdYlGn', ax=ax1, cbar_kws={'label': 'Full Scale'})
ax1.set_title(f"BEFORE: Raw Data (Outliers dominate the color scale)", fontsize=14)
```

**Explanation:**
- No vmax limit, so color scale stretches to maximum value
- If one over has SR=300, colors saturate
- All normal values (0-150) look similar
- Outliers dominate visualization

### Plot 2: After (Capped Scale)
```python
robust_vmax = 250

sns.heatmap(stats[['raw_sr']].T, annot=True, fmt=".1f", 
            cmap='RdYlGn', ax=ax2, vmax=robust_vmax,
            cbar_kws={'label': f'Capped at {robust_vmax} SR'})
ax2.set_title(f"AFTER: Robust Transformation (...)", fontsize=14)
```

**Explanation:**
- `vmax=robust_vmax` - Caps color scale at 250
- Values > 250 still display but use max color
- Values 0-250 use full color range
- **Result:** Normal variations now visible

---

## Summary of Key Concepts

### Data Manipulation
- **Filtering**: `df[df['column'] == value]`
- **Grouping**: `.groupby()` followed by `.agg()`
- **Reindexing**: `.reindex()` to handle missing values
- **Transformations**: `.apply()` for element-wise operations

### Visualization Techniques
- **Heatmaps**: Color-coded matrix showing patterns
- **Bar Charts**: Comparing categorical values
- **Scatter Plots**: Relationships with variable sizes/colors
- **Dual Mapping**: Different data for colors vs annotations

### Machine Learning
- **Encoding**: Converting categories to numbers
- **Random Forest**: Ensemble method for predictions
- **Training**: `.fit()` learns patterns from data

### Cricket Metrics
- **Strike Rate**: (Runs / Balls) × 100 (batting aggression)
- **Phase Analysis**: Powerplay (0-5), Middle (6-14), Death (15-19)
- **Risk-Reward**: Balance between aggression and survival
- **SASR**: Survival-Adjusted Strike Rate (adjusted for dismissals)






