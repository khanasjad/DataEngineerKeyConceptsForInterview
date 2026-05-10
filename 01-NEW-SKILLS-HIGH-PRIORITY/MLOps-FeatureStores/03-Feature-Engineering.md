# Chapter 03: Feature Engineering

**Creating Powerful Features for Machine Learning**

---

## What is Feature Engineering?

### Simple Definition

**Feature Engineering** is the process of transforming raw data into features (inputs) that better represent the underlying problem to ML models, resulting in improved performance.

### Real-Life Analogy

**Raw data** = Raw ingredients (flour, eggs, sugar)
**Feature engineering** = Combining ingredients into cake mix
**ML model** = Oven that bakes the cake

Better ingredients (features) → Better cake (predictions)

---

## Why Feature Engineering Matters

**Impact on Model Performance:**
```
Same algorithm with:
- Raw features: 70% accuracy
- Engineered features: 92% accuracy

Feature engineering often improves models more than algorithm tuning!
```

---

## Types of Features

### 1. Numerical Features

**Examples:**
- Age, salary, price
- Counts, sums, averages
- Ratios, percentages

**Common Transformations:**

**A. Scaling**
```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler

# StandardScaler (mean=0, std=1)
scaler = StandardScaler()
df['age_scaled'] = scaler.fit_transform(df[['age']])

# MinMaxScaler (range 0-1)
scaler = MinMaxScaler()
df['salary_scaled'] = scaler.fit_transform(df[['salary']])
```

**B. Binning**
```python
# Age groups
df['age_group'] = pd.cut(
    df['age'],
    bins=[0, 18, 35, 50, 65, 100],
    labels=['child', 'young_adult', 'adult', 'middle_aged', 'senior']
)
```

**C. Log Transformation (for skewed data)**
```python
import numpy as np

df['log_income'] = np.log1p(df['income'])  # log(1 + x) handles zeros
```

---

### 2. Categorical Features

**Examples:**
- Gender, country, product category
- Day of week, color, status

**Encoding Methods:**

**A. Label Encoding (for ordinal data)**
```python
from sklearn.preprocessing import LabelEncoder

# Education: high_school < bachelor < master < phd
education_order = ['high_school', 'bachelor', 'master', 'phd']
df['education_encoded'] = df['education'].map({
    'high_school': 0,
    'bachelor': 1,
    'master': 2,
    'phd': 3
})
```

**B. One-Hot Encoding**
```python
# For nominal categories (no order)
df_encoded = pd.get_dummies(df, columns=['country', 'gender'])

# Result:
# country_USA | country_UK | country_Canada | gender_M | gender_F
#     1       |     0      |      0         |    1     |    0
```

**C. Target Encoding (for high cardinality)**
```python
# Replace category with mean of target
target_mean = df.groupby('city')['purchase'].mean()
df['city_target_encoded'] = df['city'].map(target_mean)

# Example:
# City: New York → Average purchase: $150
# City: Los Angeles → Average purchase: $120
```

---

### 3. DateTime Features

**Extract Rich Information:**
```python
df['date'] = pd.to_datetime(df['date'])

# Extract components
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day'] = df['date'].dt.day
df['dayofweek'] = df['date'].dt.dayofweek  # 0=Monday
df['hour'] = df['date'].dt.hour
df['is_weekend'] = df['dayofweek'].isin([5, 6]).astype(int)
df['is_holiday'] = df['date'].isin(holiday_list).astype(int)
df['quarter'] = df['date'].dt.quarter

# Cyclical encoding (for hour, month, day of week)
df['hour_sin'] = np.sin(2 * np.pi * df['hour'] / 24)
df['hour_cos'] = np.cos(2 * np.pi * df['hour'] / 24)
```

**Why cyclical encoding?**
- Hour 23 and hour 0 are close, but 23 and 0 are far in linear encoding
- Sin/cos preserves circular nature

---

### 4. Text Features

**A. Basic Statistics:**
```python
df['text_length'] = df['review'].str.len()
df['word_count'] = df['review'].str.split().str.len()
df['avg_word_length'] = df['text_length'] / df['word_count']
df['capital_letter_count'] = df['review'].str.findall(r'[A-Z]').str.len()
```

**B. TF-IDF (Term Frequency-Inverse Document Frequency):**
```python
from sklearn.feature_extraction.text import TfIdfVectorizer

vectorizer = TfIdfVectorizer(max_features=100)
tfidf_matrix = vectorizer.fit_transform(df['review'])

# Convert to DataFrame
tfidf_df = pd.DataFrame(
    tfidf_matrix.toarray(),
    columns=vectorizer.get_feature_names_out()
)
```

**C. Embeddings (for deep learning):**
```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')
df['text_embedding'] = df['review'].apply(lambda x: model.encode(x))
```

---

## Advanced Feature Engineering

### 1. Interaction Features

**Combine features to capture relationships:**
```python
# Polynomial features
from sklearn.preprocessing import PolynomialFeatures

poly = PolynomialFeatures(degree=2, include_bias=False)
features_poly = poly.fit_transform(df[['age', 'income']])

# Result includes:
# - age
# - income
# - age^2
# - income^2
# - age × income  ← Interaction!
```

**Manual interactions:**
```python
df['price_per_sqft'] = df['price'] / df['square_feet']
df['income_to_debt_ratio'] = df['income'] / (df['debt'] + 1)
df['purchase_frequency'] = df['total_purchases'] / df['days_since_registration']
```

---

### 2. Aggregation Features

**Group statistics:**
```python
# Customer-level aggregates
customer_features = df.groupby('customer_id').agg({
    'purchase_amount': ['mean', 'sum', 'std', 'min', 'max'],
    'purchase_date': ['count', 'min', 'max']
}).reset_index()

# Flatten column names
customer_features.columns = ['_'.join(col).strip('_') for col in customer_features.columns]

# Result:
# customer_id | purchase_amount_mean | purchase_amount_sum | purchase_amount_count | ...
```

**Time-based aggregates:**
```python
# Rolling window features
df['sales_7day_avg'] = df.groupby('store_id')['sales'].transform(
    lambda x: x.rolling(window=7, min_periods=1).mean()
)

df['sales_30day_sum'] = df.groupby('store_id')['sales'].transform(
    lambda x: x.rolling(window=30, min_periods=1).sum()
)
```

---

### 3. Ratio & Percentage Features

```python
# Conversion rate
df['conversion_rate'] = df['purchases'] / df['visits']

# Percentage change
df['price_change_pct'] = (df['current_price'] - df['previous_price']) / df['previous_price']

# Share of total
df['category_share'] = df.groupby('category')['sales'].transform(
    lambda x: x / x.sum()
)
```

---

### 4. Lag Features (Time Series)

```python
# Previous values
df['sales_lag_1'] = df.groupby('store_id')['sales'].shift(1)  # Yesterday
df['sales_lag_7'] = df.groupby('store_id')['sales'].shift(7)  # Last week
df['sales_lag_30'] = df.groupby('store_id')['sales'].shift(30)  # Last month

# Difference from previous
df['sales_diff'] = df['sales'] - df['sales_lag_1']
```

---

### 5. Domain-Specific Features

**E-commerce Example:**
```python
# Customer behavior
df['days_since_last_purchase'] = (pd.to_datetime('today') - df['last_purchase_date']).dt.days
df['avg_days_between_purchases'] = df['total_days'] / df['purchase_count']
df['is_returning_customer'] = (df['purchase_count'] > 1).astype(int)

# Product features
df['is_discounted'] = (df['discount'] > 0).astype(int)
df['discount_percentage'] = (df['original_price'] - df['sale_price']) / df['original_price']
df['price_tier'] = pd.qcut(df['price'], q=5, labels=['budget', 'low', 'mid', 'high', 'premium'])
```

---

## Feature Selection

**Why Select Features?**
- Reduce overfitting
- Faster training
- Better interpretability
- Lower storage/compute costs

### 1. Remove Low Variance Features

```python
from sklearn.feature_selection import VarianceThreshold

selector = VarianceThreshold(threshold=0.01)
X_selected = selector.fit_transform(X)

# Removes features with variance < 0.01
```

---

### 2. Correlation-Based Selection

```python
# Remove highly correlated features
correlation_matrix = df.corr().abs()

# Find pairs with correlation > 0.95
high_corr = (correlation_matrix > 0.95) & (correlation_matrix < 1.0)

# Remove one feature from each pair
to_drop = set()
for col in high_corr.columns:
    if high_corr[col].any():
        correlated_features = high_corr.index[high_corr[col]].tolist()
        to_drop.update(correlated_features[1:])  # Keep first, drop rest

df_reduced = df.drop(columns=list(to_drop))
```

---

### 3. Feature Importance (from models)

```python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier()
model.fit(X_train, y_train)

# Get feature importance
importance = pd.DataFrame({
    'feature': X_train.columns,
    'importance': model.feature_importances_
}).sort_values('importance', ascending=False)

# Select top N features
top_features = importance.head(20)['feature'].tolist()
X_selected = X_train[top_features]
```

---

### 4. Recursive Feature Elimination (RFE)

```python
from sklearn.feature_selection import RFE

model = RandomForestClassifier()
rfe = RFE(estimator=model, n_features_to_select=10)
rfe.fit(X_train, y_train)

# Selected features
selected_features = X_train.columns[rfe.support_]
```

---

## Data Engineering for Features

### Feature Pipeline Example

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.compose import ColumnTransformer

# Define transformations for different column types
numeric_features = ['age', 'income', 'credit_score']
categorical_features = ['gender', 'country', 'employment_type']

numeric_transformer = Pipeline(steps=[
    ('scaler', StandardScaler())
])

categorical_transformer = Pipeline(steps=[
    ('onehot', OneHotEncoder(handle_unknown='ignore'))
])

# Combine transformers
preprocessor = ColumnTransformer(
    transformers=[
        ('num', numeric_transformer, numeric_features),
        ('cat', categorical_transformer, categorical_features)
    ])

# Full pipeline
pipeline = Pipeline(steps=[
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier())
])

# Fit and predict
pipeline.fit(X_train, y_train)
predictions = pipeline.predict(X_test)
```

---

### Storing Features (Batch Processing)

```python
import pyarrow.parquet as pq

def compute_customer_features(df):
    """Compute features for customers"""
    
    features = df.groupby('customer_id').agg({
        'purchase_amount': ['mean', 'sum', 'count'],
        'purchase_date': ['min', 'max']
    }).reset_index()
    
    # Flatten columns
    features.columns = ['_'.join(col).strip('_') for col in features.columns]
    
    # Add computed features
    features['days_as_customer'] = (
        pd.to_datetime(features['purchase_date_max']) -
        pd.to_datetime(features['purchase_date_min'])
    ).dt.days
    
    features['avg_purchase_value'] = (
        features['purchase_amount_sum'] / features['purchase_amount_count']
    )
    
    return features

# Compute features
customer_features = compute_customer_features(transactions_df)

# Save to Parquet (efficient storage)
customer_features.to_parquet('features/customer_features.parquet', index=False)
```

---

## Feature Validation

### Data Quality Checks

```python
def validate_features(df):
    """Validate feature quality"""
    
    issues = []
    
    # 1. Check for nulls
    null_pct = df.isnull().sum() / len(df)
    high_null_cols = null_pct[null_pct > 0.1].index.tolist()
    if high_null_cols:
        issues.append(f"High null percentage: {high_null_cols}")
    
    # 2. Check for inf values
    inf_cols = df.columns[df.isin([np.inf, -np.inf]).any()].tolist()
    if inf_cols:
        issues.append(f"Inf values found: {inf_cols}")
    
    # 3. Check value ranges
    if (df['age'] < 0).any() or (df['age'] > 120).any():
        issues.append("Invalid age values detected")
    
    # 4. Check for data drift
    # Compare distributions with training data
    
    if issues:
        raise ValueError(f"Feature validation failed: {issues}")
    
    return True

# Usage
validate_features(features_df)
```

---

## Best Practices

### 1. Avoid Target Leakage

**BAD:**
```python
# Don't include future information!
df['total_future_purchases'] = df.groupby('customer_id')['purchases'].transform('sum')
# ↑ This includes purchases AFTER the prediction time!
```

**GOOD:**
```python
# Only use past information
df['total_past_purchases'] = df.groupby('customer_id')['purchases'].cumsum() - df['purchases']
```

---

### 2. Handle Missing Values Properly

```python
# Strategies:
# - Numeric: Fill with median/mean
df['age'].fillna(df['age'].median(), inplace=True)

# - Categorical: Fill with mode or 'Unknown'
df['country'].fillna('Unknown', inplace=True)

# - Create indicator for missingness
df['age_missing'] = df['age'].isnull().astype(int)
```

---

### 3. Keep Train/Test Consistent

```python
# Fit on training data only!
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)  # Fit + transform
X_test_scaled = scaler.transform(X_test)  # Transform only (no fit!)
```

---

### 4. Document Features

```python
feature_metadata = {
    'customer_lifetime_value': {
        'description': 'Total revenue from customer',
        'formula': 'SUM(purchase_amount)',
        'data_type': 'float',
        'unit': 'dollars',
        'created_date': '2024-01-15',
        'owner': 'data-team'
    },
    'purchase_frequency': {
        'description': 'Average days between purchases',
        'formula': 'total_days / purchase_count',
        'data_type': 'float',
        'unit': 'days',
        'created_date': '2024-01-15',
        'owner': 'data-team'
    }
}
```

---

## Summary

### Key Takeaways:

✅ **Feature Engineering** often improves models more than algorithm tuning
✅ **Types:** Numerical, categorical, datetime, text
✅ **Techniques:** Scaling, encoding, aggregations, interactions
✅ **Selection:** Remove low-variance, highly correlated features
✅ **Validation:** Check for leakage, missing values, data drift
✅ **Best Practices:** Document features, keep train/test consistent

### For Data Engineers:

- Build scalable feature pipelines
- Store features efficiently (Parquet, Delta Lake)
- Version features alongside data
- Validate feature quality continuously
- Enable feature reuse across models

---

**Continue to Chapter 04 to learn about Feature Stores! 🚀**
