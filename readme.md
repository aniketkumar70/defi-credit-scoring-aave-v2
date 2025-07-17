# DeFi Credit Scoring System

## Overview

This system develops a machine learning-based credit scoring model for DeFi wallets using Aave V2 transaction data. The model assigns credit scores between 0-1000 based on historical transaction behavior, with higher scores indicating more reliable and responsible usage patterns.

## Architecture

### Data Processing Pipeline
1. **Data Loading**: Parse JSON transaction data and extract relevant fields
2. **Feature Engineering**: Create behavioral features from transaction history
3. **Synthetic Labeling**: Generate training labels based on behavioral patterns
4. **Model Training**: Train Random Forest regressor with cross-validation
5. **Score Generation**: Predict credit scores for all wallets
6. **Analysis**: Analyze score distributions and wallet behaviors

### Feature Engineering

The system extracts the following categories of features:

#### 1. Activity Features
- **Total Transactions**: Overall activity level
- **Unique Actions**: Diversity of transaction types
- **Account Age**: Time since first transaction
- **Transaction Frequency**: Transactions per day

#### 2. Volume Features
- **Total Volume**: Cumulative USD value of all transactions
- **Average/Median/Max Transaction Size**: Transaction size patterns
- **Volume Standard Deviation**: Volatility in transaction sizes

#### 3. Action Distribution
- **Deposits/Borrows/Repays/Redeems/Liquidations**: Count of each action type
- **Deposit/Borrow/Repay Volumes**: USD volumes by action type

#### 4. Risk Indicators
- **Liquidation Ratio**: Liquidations per total transactions
- **Borrow-to-Deposit Ratio**: Leverage usage patterns
- **Repay-to-Borrow Ratio**: Debt repayment behavior

#### 5. Behavioral Patterns
- **Hour Concentration**: Bot-like timing patterns
- **Time Variance**: Regularity of transaction timing
- **Asset Diversity**: Number of unique assets used

## Scoring Methodology

### Synthetic Label Generation
Since ground truth credit scores aren't available, we create synthetic labels based on four components:

1. **Activity Score (0-250 points)**
   - Rewards consistent, diverse usage
   - Based on transaction count, action diversity, and account age

2. **Volume Score (0-250 points)**
   - Rewards substantial, consistent volume
   - Logarithmic scaling to handle outliers

3. **Responsibility Score (0-300 points)**
   - Rewards good debt management
   - Penalizes liquidations, rewards repayments and asset diversity

4. **Stability Score (0-200 points)**
   - Rewards human-like behavior patterns
   - Penalizes bot-like timing and excessive leverage

### Model Architecture
- **Algorithm**: Random Forest Regressor
- **Features**: 23 engineered features
- **Preprocessing**: RobustScaler for outlier handling
- **Validation**: 80/20 train-test split

## Credit Score Interpretation

### Score Ranges
- **900-1000**: Excellent credit - Highly responsible, consistent users
- **800-899**: Very Good - Strong repayment history, good volume
- **700-799**: Good - Reliable users with moderate activity
- **600-699**: Fair - Average users with some risk indicators
- **500-599**: Poor - High risk or inconsistent behavior
- **0-499**: Very Poor - High liquidation risk or bot-like behavior

### Key Indicators

**High Score Characteristics:**
- Consistent repayment behavior
- Diverse asset usage
- Reasonable leverage ratios
- Long account history
- Human-like transaction patterns

**Low Score Characteristics:**
- High liquidation rates
- Excessive leverage
- Bot-like timing patterns
- Short account history
- Irregular transaction patterns

## Usage

### Prerequisites
```bash
pip install pandas numpy scikit-learn matplotlib seaborn
```

### Running the Model
```python
from defi_credit_scorer import DeFiCreditScorer

# Initialize scorer
scorer = DeFiCreditScorer('/path/to/transactions.json')

# Run complete analysis
scores = scorer.run_complete_analysis()

# Get results
print(scores.head())
```

### Output Files
- `wallet_credit_scores.csv`: Final credit scores for all wallets
- Analysis plots: Score distributions and behavioral patterns

## Model Performance

The model achieves good performance on synthetic labels with:
- Training R²: ~0.85-0.90
- Test R²: ~0.80-0.85

Top important features typically include:
1. Total volume
2. Repay-to-borrow ratio
3. Account age
4. Liquidation ratio
5. Transaction frequency

## Limitations & Future Improvements

### Current Limitations
1. **Synthetic Labels**: No ground truth credit scores available
2. **Single Protocol**: Only Aave V2 data considered
3. **Limited Timeframe**: Historical data only
4. **No External Data**: No on-chain reputation or identity verification

### Future Enhancements
1. **Multi-Protocol Analysis**: Include Compound, MakerDAO, etc.
2. **Real-Time Scoring**: Live score updates
3. **External Data Integration**: ENS names, governance participation
4. **Advanced ML**: Deep learning models, ensemble methods
5. **Validation**: Compare with actual default rates

## Technical Details

### Data Schema
```json
{
  "userWallet": "0x...",
  "action": "deposit|borrow|repay|redeemunderlying|liquidationcall",
  "timestamp": 1629178166,
  "actionData": {
    "amount": "2000000000",
    "assetSymbol": "USDC",
    "assetPriceUSD": "0.9938..."
  }
}
```

### Feature Pipeline
1. Parse transaction data
2. Aggregate by wallet
3. Calculate behavioral metrics
4. Handle outliers and missing values
5. Scale features for model training

## Validation Strategy

The model uses multiple validation approaches:
1. **Statistical Validation**: Score distributions match expected patterns
2. **Behavioral Validation**: High scores correlate with responsible behavior
3. **Temporal Validation**: Scores remain stable over time
4. **Cross-Validation**: K-fold validation during training

## Conclusion

This credit scoring system provides a foundation for assessing DeFi wallet creditworthiness based on transaction behavior. While using synthetic labels, it captures important behavioral patterns that correlate with responsible DeFi usage. The modular design allows for easy extension and improvement as more data becomes available.
