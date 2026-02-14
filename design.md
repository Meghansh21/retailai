# System Design Document: AI-Powered Retail Intelligence Platform

## 1. Executive Summary

This document describes the system design for an AI-powered retail intelligence platform tailored for small retailers in India (Bharat context). The platform addresses the challenge of sparse and noisy retailer sales data by integrating external market signals to generate reliable short-term demand forecasts and actionable inventory insights.

**Core Innovation**: Combining retailer-specific sales patterns with external market signals (seasonal trends, holiday patterns, pricing trends, regional signals, search volume data) to improve forecast stability when retailer data alone is insufficient. The system continuously learns from actual sales results and provides explainable recommendations.

**Design Philosophy**: Simple, modular, privacy-focused, explainable AI, continuous learning, and designed for decision support rather than automation.

## 2. Problem Context and Design Rationale

### 2.1 The Challenge

Small retailers in India face significant inventory planning challenges:

1. **Sparse Sales Data**: Limited transaction history, especially for new stores or seasonal products
2. **Noisy Patterns**: Stockouts create artificial demand signals (zero sales doesn't mean zero demand)
3. **Regional Variability**: Demand patterns vary significantly by location, culture, and local events
4. **Limited Resources**: No access to sophisticated forecasting tools or data science expertise
5. **Market Disconnect**: Difficulty understanding broader market trends and consumer interest shifts

**Key Insight**: Retailer-only sales data is often insufficient for reliable forecasting. External market signals provide stability, context, and early indicators of demand changes.

### 2.2 Solution Approach

**Hybrid Data Strategy**: Combine retailer-specific sales patterns with external market signals to create more robust forecasts.

**External Market Signals Include**:
- Seasonal trends and weather patterns
- Holiday and festival calendars (national and regional)
- Pricing trends and market dynamics
- Regional demographic and economic indicators
- Search volume data indicating consumer interest

**Why Machine Learning?**
- **Pattern Recognition**: ML models can learn complex relationships between sales, seasonality, regional events, and market signals
- **Uncertainty Quantification**: ML provides confidence intervals, not just point estimates
- **Adaptability**: Models improve as more data becomes available (continuous learning)
- **Feature Integration**: ML naturally combines multiple data sources (sales + external signals)
- **Similar Product Analysis**: ML can identify products with similar demand patterns

**Why Not Rule-Based?**
- Rules cannot capture non-linear relationships between demand drivers
- Rules require manual tuning for each region, category, and market condition
- Rules cannot adapt to changing patterns or provide uncertainty estimates
- Rules cannot learn from forecast errors or actual results

**Continuous Learning Loop**: The system compares predictions with actual results, calculates accuracy metrics, and adapts models to improve future forecasts.

## 3. High-Level Architecture

### 3.1 System Overview

The platform follows a modular service-oriented architecture with six core components:

```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                        │
│              Web Dashboard (React.js)                        │
│  [Forecasts] [Insights] [Similar Products] [Analytics]      │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      API Gateway                             │
│     Authentication │ Rate Limiting │ Request Routing        │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌──────────────┐    ┌──────────────────┐    ┌──────────────┐
│ Data         │    │ External Market  │    │ Feature      │
│ Ingestion    │    │ Signal Service   │    │ Engineering  │
│ Service      │    │                  │    │ Service      │
└──────────────┘    └──────────────────┘    └──────────────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              ▼
                    ┌──────────────────┐
                    │ ML Forecasting   │
                    │ Service          │
                    │ (Continuous      │
                    │  Learning)       │
                    └──────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌──────────────┐    ┌──────────────────┐    ┌──────────────┐
│ Insight      │    │ Similar Product  │    │ Product-     │
│ Generation   │    │ Analysis Service │    │ Market Fit   │
│ Service      │    │                  │    │ Service      │
└──────────────┘    └──────────────────┘    └──────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Data Layer                              │
│  [PostgreSQL] [TimescaleDB] [Redis Cache] [Object Storage]  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              External Data Sources (Read-Only)               │
│  [Search Trends API] [Festival Calendar] [Weather Data]     │
│  [Regional Demographics] [Market Price Indices]             │
└─────────────────────────────────────────────────────────────┘
```
                              ↕
┌─────────────────────────────────────────────────────────────┐
│                      API Gateway                             │
│         Authentication | Routing | Validation               │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌──────────────┬──────────────┬──────────────┬───────────────┐
│   Data       │   Regional   │  Forecasting │   Insight     │
│  Ingestion   │   Signal     │   Engine     │  Generation   │
│   Service    │   Service    │   (ML Core)  │   Service     │
└──────────────┴──────────────┴──────────────┴───────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│                       Data Layer                             │
│  [Sales DB] [Regional Signals DB] [Models] [Feature Store]  │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Core Components

**1. Data Ingestion Service**
- Accepts CSV uploads and API submissions
- Validates and normalizes retailer sales data
- Rejects any personal customer information
- Stores category-level sales time-series
- Triggers downstream processing pipeline

**2. External Market Signal Service**
- Integrates external market signals from multiple sources:
  - Seasonal trends and weather patterns
  - Holiday and festival calendars
  - Pricing trends and market indices
  - Regional demographic data
  - Search volume data (consumer interest indicators)
- Maps signals to retailer locations (state/city/district)
- Associates signals with relevant product categories
- Updates periodically from external sources
- Provides contextual features for forecasting

**3. Feature Engineering Service**
- Combines retailer sales data with external market signals
- Generates time-series features (lags, rolling averages, trends)
- Creates demand signal features (days to festival, search volume changes)
- Handles missing data and outliers
- Stores engineered features for model training and inference

**4. ML Forecasting Service (Continuous Learning)**
- Trains ML models combining sales history and external signals
- Generates 7-15 day category-level demand forecasts
- Provides confidence intervals for predictions
- Compares predictions with actual results (feedback loop)
- Calculates forecast accuracy metrics (MAPE, bias)
- Automatically retrains models based on new data and performance
- Adapts to retailer-specific patterns over time

**5. Similar Product Analysis Service**
- Identifies products with similar demand patterns
- Analyzes correlation between product sales
- Considers seasonal alignment, price sensitivity, sales velocity
- Recommends similar products performing well in the region
- Helps retailers discover cross-selling opportunities

**6. Product-Market Fit Service**
- Compares retailer sales performance with regional demand signals
- Identifies products with strong/weak market fit
- Flags opportunities (high regional demand, low retailer sales)
- Detects declining products (sales dropping despite stable demand)
- Uses search volume trends as demand proxy

**7. Insight Generation Service**
- Transforms forecasts into actionable recommendations
- Identifies stockout, overstock, and seasonal spike risks
- Generates trend alerts based on search volume changes
- Provides product-market fit insights
- Prioritizes insights by urgency (high/medium/low)
- Explains reasoning behind each recommendation (explainability)
- Tracks recommendation acknowledgment and actions

**8. Web Dashboard**
- Displays forecasts with confidence bands
- Shows prioritized inventory recommendations
- Provides similar product insights
- Displays product-market fit analysis
- Shows external market signals on timeline
- Provides data upload interface
- Tracks forecast accuracy over time
- Supports bilingual interface (English/Hindi)

### 3.3 Architecture Principles

- **Modularity**: Independent services with clear responsibilities
- **Privacy-by-Design**: No PII collection, data minimization
- **Explainability**: All predictions include reasoning and contributing factors
- **Continuous Learning**: Feedback loop from actual results to model improvement
- **Scalability**: Horizontal scaling for growing user base
- **Simplicity**: Low operational complexity suitable for small businesses
- **Resilience**: Graceful degradation when external data sources are unavailable

## 4. Data Flow

### 4.1 End-to-End Data Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│ STEP 1: Data Collection                                     │
│                                                              │
│ Retailer → CSV Upload → Validation → Sales Database         │
│                                                              │
│ External Sources → Market Signal Service → Signal Database  │
│ • Search Trends API                                          │
│ • Festival/Holiday Calendar                                  │
│ • Weather/Seasonal Data                                      │
│ • Regional Demographics                                      │
│ • Market Price Indices                                       │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 2: Data Enrichment & Feature Engineering              │
│                                                              │
│ Sales Data + External Market Signals → Feature Engineering →│
│ Combined Feature Set                                         │
│                                                              │
│ Features Created:                                            │
│ • Historical sales patterns (lags, rolling averages)        │
│ • Temporal features (day of week, month, season)            │
│ • External context (upcoming festivals, weather)            │
│ • Market trends (search volume changes, price trends)       │
│ • Regional signals (population, urban density)              │
│ • Similar product features (correlation patterns)           │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 3: ML Forecasting (with Continuous Learning)          │
│                                                              │
│ Feature Set → ML Model → Category Forecasts (7-15 days) →   │
│ Confidence Intervals                                         │
│                                                              │
│ Model Output:                                                │
│ • Point forecast for each day                               │
│ • Lower and upper confidence bounds (80%, 95%)              │
│ • Feature importance (what drove the forecast)              │
│ • Forecast explanation (key contributing factors)           │
│                                                              │
│ Feedback Loop:                                               │
│ Actual Sales → Compare with Forecast → Calculate Accuracy → │
│ Update Model Parameters → Improve Future Predictions        │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 4: Analysis & Insight Generation                       │
│                                                              │
│ Forecasts + Market Signals → Multiple Analysis Engines      │
│                                                              │
│ A. Inventory Risk Analysis:                                 │
│    • Stockout risk alerts (high demand expected)            │
│    • Overstock warnings (low demand expected)               │
│    • Seasonal spike alerts (festival approaching)           │
│    • Trend alerts (search volume spike detected)            │
│                                                              │
│ B. Similar Product Analysis:                                │
│    • Identify products with correlated demand               │
│    • Recommend similar products performing well             │
│    • Cross-selling opportunities                            │
│                                                              │
│ C. Product-Market Fit Assessment:                           │
│    • Compare retailer sales vs regional demand              │
│    • Flag strong/weak fit products                          │
│    • Identify missed opportunities                          │
│    • Detect declining products                              │
│                                                              │
│ D. Reorder Recommendations:                                 │
│    • Calculate suggested quantities                         │
│    • Consider lead time and safety stock                    │
│    • Prioritize by urgency (high/medium/low)                │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 5: Presentation & User Interaction                     │
│                                                              │
│ Dashboard → Visualizations + Recommendations → Retailer     │
│                                                              │
│ Display:                                                     │
│ • Forecast charts with confidence bands                     │
│ • External market signals on timeline                       │
│ • Prioritized action items with explanations                │
│ • Similar product insights                                  │
│ • Product-market fit analysis                               │
│ • Forecast accuracy trends                                  │
│                                                              │
│ User Actions:                                                │
│ • Acknowledge recommendations                                │
│ • Mark actions as completed                                 │
│ • Upload new sales data                                     │
│                                                              │
│ Feedback to System:                                          │
│ User actions → Learning signal → Model adaptation           │
│ • Historical accuracy metrics                               │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Data Processing Timeline

**Real-Time** (< 1 minute):
- CSV upload and validation
- Dashboard data retrieval
- User authentication

**Near Real-Time** (5-15 minutes):
- Feature engineering after new data upload
- Forecast generation for updated categories
- Similar product analysis
- Product-market fit assessment
- Insight generation and prioritization

**Batch Processing** (Scheduled):
- Model retraining: Weekly or when accuracy drops below threshold
- External market signal updates: Daily
- Forecast accuracy evaluation: Daily (compare predictions vs actuals)
- Model performance monitoring: Daily
- Report generation: Weekly
- Search volume trend updates: Daily

**Continuous Learning Cycle** (Ongoing):
- Actual sales data arrives → Compare with forecast → Calculate error metrics →
- Identify systematic biases → Adjust model parameters → Improve next forecast

### 4.3 Handling Sparse Data

**Challenge**: New retailers or seasonal categories may have limited historical data.

**Strategies**:
1. **External Signal Reliance**: Use external market signals as primary input when retailer data is sparse
2. **Category Similarity**: Leverage patterns from similar categories within the retailer's inventory
3. **Regional Baselines**: Use regional demand patterns as baseline when retailer data is insufficient
4. **Confidence Adjustment**: Wider confidence intervals for sparse data scenarios
5. **Minimum Data Threshold**: Require at least 4-6 weeks of data before generating forecasts
6. **Fallback Methods**: Use simple moving averages or seasonal naive methods when ML confidence is too low
7. **Search Volume Proxy**: Use search volume trends as demand indicator when sales history is limited

## 5. Component Design Details

### 5.1 Data Ingestion Service

**Purpose**: Accept, validate, and store retailer sales data.

**Technology Stack**:
- Language: Python (FastAPI framework)
- File Processing: Pandas
- Validation: Pydantic schemas
- Storage: TimescaleDB (time-series data), PostgreSQL (metadata)

**Key Responsibilities**:
- CSV file upload and parsing
- Data validation (schema, data types, ranges)
- Privacy filtering (reject any PII fields)
- Category name normalization
- Data quality reporting

**Validation Rules**:
- Required fields: date, category, quantity
- Optional fields: price, store_location
- Reject: Customer names, phone numbers, emails, payment details
- Date range: No future dates, within reasonable historical range
- Quantity: Positive numbers only

**API Endpoints**:
```
POST /api/v1/data/upload          # CSV file upload
POST /api/v1/data/submit          # JSON data submission (future)
GET  /api/v1/data/validation/{id} # Check upload status
GET  /api/v1/data/history         # Retrieve historical data
```

### 5.2 External Market Signal Service

**Purpose**: Collect, maintain, and provide external market signals for forecasting.

**Technology Stack**:
- Language: Python
- Scheduler: Celery Beat or Apache Airflow
- Storage: PostgreSQL
- APIs: Search Trends API, Weather API, Public data sources

**Data Sources**:
1. **Search Volume Data**: Google Trends API or similar for product category search trends
2. **Festival/Holiday Calendar**: Indian government calendar, regional festival databases
3. **Seasonal Patterns**: Weather data, agricultural seasons
4. **Regional Demographics**: Census data, population indicators
5. **Market Price Indices**: Public commodity price data where available

**Key Responsibilities**:
- Fetch and update external signals daily
- Map signals to geographic regions (state/city/district)
- Associate signals with product categories
- Provide API for feature engineering service
- Handle API failures gracefully (use cached data)

**Data Model**:
```sql
CREATE TABLE external_signals (
    id UUID PRIMARY KEY,
    signal_type VARCHAR(50) NOT NULL, -- 'festival', 'season', 'search_trend', 'price', 'demographic'
    signal_name VARCHAR(255) NOT NULL,
    start_date DATE,
    end_date DATE,
    regions TEXT[],
    affected_categories TEXT[],
    signal_value JSONB, -- Flexible storage for different signal types
    source VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_signals_date ON external_signals(start_date, end_date);
CREATE INDEX idx_signals_regions ON external_signals USING GIN(regions);
CREATE INDEX idx_signals_categories ON external_signals USING GIN(affected_categories);
```

**Example Signal Types**:
- Festival: `{name: "Diwali", impact_score: 9, categories: ["sweets", "decorations", "gifts"]}`
- Search Trend: `{category: "umbrellas", trend_change: +45%, period: "last_7_days"}`
- Season: `{name: "Monsoon", start: "2026-06-01", end: "2026-09-30", region: "Maharashtra"}`
- Price: `{category: "rice", price_index: 105, change: +5%}`

### 5.3 Feature Engineering Service

**Purpose**: Transform raw sales data and external signals into ML-ready features.

**Technology Stack**:
- Language: Python
- Libraries: Pandas, NumPy, Scikit-learn
- Storage: PostgreSQL (feature store)

**Feature Categories**:

1. **Temporal Features**:
   - Day of week, day of month, week of year, month, quarter
   - Is weekend, is month-end, is month-start
   - Cyclical encoding (sin/cos for day, month)

2. **Lag Features** (from sales history):
   - Sales lag: t-1, t-7, t-14, t-30 days
   - Rolling averages: 7-day, 14-day, 30-day
   - Rolling standard deviation (volatility)
   - Week-over-week growth rate
   - Month-over-month growth rate

3. **Trend Features**:
   - Linear trend over last 30/60/90 days
   - Exponential moving average
   - Trend direction (increasing/decreasing/stable)

4. **External Signal Features**:
   - Days until next festival
   - Festival impact score (0-10)
   - Is festival week (binary)
   - Seasonal indicator (monsoon/summer/winter/spring)
   - Search volume change (% change from baseline)
   - Search volume trend (increasing/decreasing)
   - Price trend indicator
   - Regional demand multiplier

5. **Category Features**:
   - Category historical volatility
   - Category average sales velocity
   - Category seasonality strength

6. **Similar Product Features**:
   - Correlation with top 3 similar products
   - Similar product average sales trend

**Missing Data Handling**:
- Forward fill for short gaps (< 3 days)
- Linear interpolation for medium gaps (3-7 days)
- Seasonal average for longer gaps
- Flag missing data periods for model awareness

### 5.4 ML Forecasting Service (with Continuous Learning)

**Purpose**: Generate demand forecasts and continuously improve from actual results.

**Technology Stack**:
- Language: Python
- ML Libraries: Scikit-learn, XGBoost, LightGBM, Prophet
- Model Registry: MLflow
- Model Serving: FastAPI with model caching

**Model Architecture**:

**Ensemble Approach** (combining multiple models):

1. **Statistical Models** (baseline):
   - Prophet: Handles seasonality and holidays well
   - SARIMA: Classical time-series forecasting
   - Exponential Smoothing: Simple baseline

2. **Machine Learning Models** (primary):
   - XGBoost/LightGBM: Gradient boosting with engineered features
   - Random Forest: Robust to outliers
   - Linear Regression with regularization: Interpretable baseline

3. **Model Selection Strategy**:
   - New retailers (<3 months data): Prophet + external signals
   - Established retailers (3-6 months): XGBoost with ensemble
   - Mature retailers (6+ months): Full ensemble with deep learning option

**Training Pipeline**:
```
1. Fetch historical sales data (last 12-18 months)
2. Fetch external signals for training period
3. Generate features via Feature Engineering Service
4. Split data: 80% train, 20% validation (time-based split)
5. Train multiple models
6. Evaluate on validation set (MAPE, RMSE, MAE, bias)
7. Select best model or create weighted ensemble
8. Save model to MLflow registry with metadata
9. Generate model card (performance metrics, feature importance)
```

**Inference Pipeline**:
```
1. Receive forecast request (retailer_id, categories, horizon)
2. Load latest trained model from registry
3. Fetch recent sales data (last 90 days)
4. Fetch upcoming external signals (next 30 days)
5. Generate features
6. Generate point forecasts for each day
7. Calculate prediction intervals (80%, 95% confidence)
8. Extract feature importance for explainability
9. Return forecasts with uncertainty and explanations
```

**Continuous Learning Loop**:
```
Daily Process:
1. Fetch actual sales from yesterday
2. Retrieve forecasts made 1-15 days ago for yesterday
3. Calculate forecast errors (actual - predicted)
4. Compute accuracy metrics:
   - MAPE (Mean Absolute Percentage Error)
   - MAE (Mean Absolute Error)
   - Bias (systematic over/under-prediction)
   - Coverage (% of actuals within prediction intervals)
5. Store accuracy metrics in database
6. If accuracy drops below threshold:
   - Trigger model retraining
   - Investigate systematic errors
   - Adjust feature weights or model parameters
7. Update model performance dashboard
```

**Explainability**:
- SHAP values for feature importance
- Top 3 contributing factors for each forecast
- Plain language explanations (e.g., "Diwali in 5 days (+30%), Search volume up 25% (+15%)")

### 5.5 Similar Product Analysis Service

**Purpose**: Identify products with similar demand patterns and provide recommendations.

**Technology Stack**:
- Language: Python
- Libraries: Scikit-learn (clustering, correlation analysis)
- Storage: PostgreSQL

**Analysis Methods**:

1. **Correlation Analysis**:
   - Calculate Pearson correlation between product sales time-series
   - Identify products with correlation > 0.7

2. **Seasonal Pattern Matching**:
   - Extract seasonal components using STL decomposition
   - Compare seasonal patterns across products
   - Group products with similar seasonality

3. **Price Sensitivity Analysis**:
   - Analyze sales response to price changes
   - Group products with similar price elasticity

4. **Clustering**:
   - K-means clustering on sales features (velocity, volatility, seasonality)
   - Identify product clusters with similar behavior

**Recommendations**:
- "Products similar to [X]: [A, B, C]"
- "Customers buying [X] often need [Y]" (based on correlated demand)
- "Consider stocking [Z] - similar products show strong sales in your region"

### 5.6 Product-Market Fit Service

**Purpose**: Assess how well retailer's product mix aligns with regional demand.

**Technology Stack**:
- Language: Python
- Storage: PostgreSQL

**Analysis Logic**:

```python
def assess_product_market_fit(product, retailer_sales, external_signals):
    # Calculate retailer's sales performance
    retailer_velocity = calculate_sales_velocity(retailer_sales)
    retailer_trend = calculate_trend(retailer_sales)
    
    # Get regional demand indicators
    search_volume = get_search_volume(product, retailer_region)
    regional_price_trend = get_price_trend(product, retailer_region)
    seasonal_demand = get_seasonal_demand(product, current_season)
    
    # Calculate fit score (0-100)
    demand_score = normalize(search_volume) * 40
    sales_score = normalize(retailer_velocity) * 30
    trend_alignment = compare_trends(retailer_trend, search_volume_trend) * 30
    
    fit_score = demand_score + sales_score + trend_alignment
    
    # Classify fit
    if fit_score > 70:
        return "STRONG_FIT", "Product performing well, aligned with market demand"
    elif fit_score > 40:
        return "MODERATE_FIT", "Product has potential, monitor trends"
    elif demand_score > 60 and sales_score < 30:
        return "OPPORTUNITY", "High market demand but low sales - consider promotion"
    else:
        return "WEAK_FIT", "Low demand or poor performance - consider alternatives"
```

**Insights Generated**:
- Strong fit products: "Keep stocking, high demand"
- Weak fit products: "Consider reducing inventory"
- Opportunities: "High regional demand, low sales - marketing opportunity"
- Declining products: "Sales dropping despite stable demand - investigate quality/price"

### 5.7 Insight Generation Service

**Purpose**: Transform forecasts and analysis into actionable recommendations.

**Technology Stack**:
- Language: Python
- Rules Engine: Custom business logic
- Storage: PostgreSQL

**Insight Types and Logic**:

1. **Stockout Risk Alert**:
```python
if forecasted_demand > current_sales_rate * 1.5:
    priority = "HIGH" if days_ahead <= 3 else "MEDIUM"
    recommendation = f"Increase stock for {category} - demand expected to rise {increase}%"
    explanation = f"Forecast: {forecast} units/day, Current rate: {current} units/day. Contributing factors: {top_factors}"
```

2. **Overstock Warning**:
```python
if forecasted_demand < current_sales_rate * 0.5:
    priority = "MEDIUM"
    recommendation = f"Reduce orders for {category} - demand expected to slow"
    explanation = f"Forecast shows {decrease}% decline. Factors: {factors}"
```

3. **Seasonal Spike Alert**:
```python
if upcoming_festival and category in festival_categories:
    priority = "HIGH"
    recommendation = f"Prepare for {festival} - {category} demand typically increases {historical_increase}%"
    explanation = f"{festival} in {days} days. Historical data shows {pattern}"
```

4. **Trend Alert** (from search volume):
```python
if search_volume_change > 30%:
    priority = "MEDIUM"
    recommendation = f"Consumer interest in {category} rising - consider increasing stock"
    explanation = f"Search volume up {change}% in last 7 days, indicating growing demand"
```

5. **Product-Market Fit Alert**:
```python
if fit_status == "OPPORTUNITY":
    priority = "MEDIUM"
    recommendation = f"{product} shows high regional demand but low sales - marketing opportunity"
    explanation = f"Search volume: {volume}, Regional sales: {regional_avg}, Your sales: {your_sales}"
```

**Recommendation Prioritization**:
- High: Immediate action needed (0-3 days), potential stockout
- Medium: Plan ahead (4-7 days), optimization opportunity
- Low: Monitor situation (8-15 days), informational

## 6. Data Models

### 6.1 Core Data Entities

**Retailer**:
```sql
CREATE TABLE retailers (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    city VARCHAR(100),
    state VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW(),
    subscription_tier VARCHAR(50),
    is_active BOOLEAN DEFAULT TRUE
);
```

**Store**:
```sql
CREATE TABLE stores (
    id UUID PRIMARY KEY,
    retailer_id UUID REFERENCES retailers(id),
    name VARCHAR(255) NOT NULL,
    city VARCHAR(100),
    state VARCHAR(100),
    pincode VARCHAR(10),
    created_at TIMESTAMP DEFAULT NOW()
);
```

**Sales Data** (TimescaleDB Hypertable):
```sql
CREATE TABLE sales_data (
    time TIMESTAMP NOT NULL,
    store_id UUID NOT NULL,
    category VARCHAR(100) NOT NULL,
    quantity DECIMAL(10,2) NOT NULL,
    price DECIMAL(10,2),
    upload_batch_id UUID,
    PRIMARY KEY (time, store_id, category)
);

SELECT create_hypertable('sales_data', 'time');
CREATE INDEX idx_sales_store_category ON sales_data(store_id, category, time DESC);
```

**Forecast**:
```sql
CREATE TABLE forecasts (
    id UUID PRIMARY KEY,
    store_id UUID REFERENCES stores(id),
    category VARCHAR(100) NOT NULL,
    forecast_date DATE NOT NULL,
    target_date DATE NOT NULL,
    predicted_quantity DECIMAL(10,2) NOT NULL,
    lower_bound DECIMAL(10,2),
    upper_bound DECIMAL(10,2),
    confidence_level DECIMAL(3,2),
    model_version VARCHAR(50),
    explanation JSONB, -- Top contributing factors
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_forecast_store_target ON forecasts(store_id, target_date);
```

**Forecast Accuracy** (for continuous learning):
```sql
CREATE TABLE forecast_accuracy (
    id UUID PRIMARY KEY,
    store_id UUID REFERENCES stores(id),
    category VARCHAR(100) NOT NULL,
    forecast_date DATE NOT NULL,
    target_date DATE NOT NULL,
    predicted_quantity DECIMAL(10,2) NOT NULL,
    actual_quantity DECIMAL(10,2) NOT NULL,
    absolute_error DECIMAL(10,2),
    percentage_error DECIMAL(5,2),
    within_confidence_interval BOOLEAN,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_accuracy_evaluation ON forecast_accuracy(store_id, category, target_date);
```

**Insight**:
```sql
CREATE TABLE insights (
    id UUID PRIMARY KEY,
    store_id UUID REFERENCES stores(id),
    category VARCHAR(100) NOT NULL,
    insight_type VARCHAR(50) NOT NULL, -- 'stockout_risk', 'overstock', 'seasonal_spike', 'trend_alert', 'product_fit'
    priority VARCHAR(20) NOT NULL, -- 'HIGH', 'MEDIUM', 'LOW'
    title VARCHAR(255) NOT NULL,
    description TEXT,
    recommended_action TEXT,
    explanation JSONB, -- Contributing factors with values
    relevant_date DATE,
    is_acknowledged BOOLEAN DEFAULT FALSE,
    acknowledged_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_insights_store_priority ON insights(store_id, priority, created_at DESC);
```

**Similar Products**:
```sql
CREATE TABLE similar_products (
    id UUID PRIMARY KEY,
    store_id UUID REFERENCES stores(id),
    product_category VARCHAR(100) NOT NULL,
    similar_category VARCHAR(100) NOT NULL,
    similarity_score DECIMAL(3,2), -- 0.00 to 1.00
    similarity_type VARCHAR(50), -- 'correlation', 'seasonal', 'price_sensitivity'
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_similar_products_lookup ON similar_products(store_id, product_category, similarity_score DESC);
```

**Product Market Fit**:
```sql
CREATE TABLE product_market_fit (
    id UUID PRIMARY KEY,
    store_id UUID REFERENCES stores(id),
    category VARCHAR(100) NOT NULL,
    fit_score DECIMAL(5,2), -- 0-100
    fit_status VARCHAR(50), -- 'STRONG_FIT', 'MODERATE_FIT', 'OPPORTUNITY', 'WEAK_FIT'
    retailer_sales_velocity DECIMAL(10,2),
    regional_demand_score DECIMAL(5,2),
    search_volume_trend VARCHAR(20), -- 'increasing', 'stable', 'decreasing'
    explanation TEXT,
    evaluated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_market_fit_store ON product_market_fit(store_id, fit_status, evaluated_at DESC);
```

## 7. Responsible AI and Data Privacy

### 7.1 Privacy-by-Design Principles

**Data Minimization**:
- Collect only category-level sales data (quantity, date, category, optional price)
- Explicitly reject PII fields during ingestion
- No customer-level transaction tracking
- No payment information

**Purpose Limitation**:
- Data used solely for forecasting and inventory insights
- No secondary use for marketing, profiling, or advertising
- No data sharing with third parties
- External signals used only for improving forecast accuracy

**Data Retention**:
- Sales data: Retain 18-24 months for model training
- Forecasts: Retain 6 months for accuracy tracking
- External signals: Retain current + 30 days future
- User data: Retain while account is active
- Deletion: Complete data removal within 30 days of account closure

**Encryption**:
- TLS 1.3 for all data in transit
- AES-256 encryption for data at rest
- Encrypted database backups
- Secure API key management

**Access Control**:
- Role-based access control (RBAC)
- Audit logging for all data access
- Multi-factor authentication for sensitive operations
- API rate limiting per user/retailer

### 7.2 Explainability and Transparency

**Explainable AI Principles**:
- Every forecast includes explanation of top contributing factors
- Feature importance displayed for each prediction
- Plain language explanations (avoid technical jargon)
- Visual indicators for confidence levels

**Forecast Explanations Include**:
- Top 3-5 contributing factors with impact percentages
- External signals influencing the forecast (festivals, search trends, seasonality)
- Historical pattern references
- Confidence level with visual indicator

**Example Explanation**:
```
Forecast: 150 units/day (±20 units)
Confidence: High (85%)

Contributing Factors:
• Diwali in 5 days (+35% impact)
• Search volume up 28% in last week (+20% impact)
• Historical pattern: Sales typically increase 40% during Diwali week (+25% impact)
• Recent sales trend: +15% week-over-week (+10% impact)
• Seasonal factor: Festival season (+10% impact)
```

**Model Cards**:
- Document model performance metrics (MAPE, accuracy by category)
- List data requirements and limitations
- Explain model selection rationale
- Provide update frequency and retraining schedule

**Transparency Dashboard**:
- Display forecast accuracy trends over time
- Show model performance by category
- Highlight when external signals are unavailable
- Indicate data quality issues

### 7.3 Model Fairness and Continuous Improvement

**Fairness Considerations**:
- Models trained per retailer (no cross-retailer bias)
- Regular accuracy audits across different regions and categories
- Avoid penalizing retailers with limited historical data (use external signals)
- Equal treatment regardless of store size or location

**Bias Mitigation**:
- Monitor for systematic over/under-prediction by region or category
- Ensure external signals cover all major Indian festivals and regions
- Regular model audits for performance disparities
- Adjust for regional economic differences

**Continuous Improvement**:
- Daily comparison of forecasts vs actuals
- Automatic detection of systematic errors
- Model retraining triggered by accuracy drops
- A/B testing for model improvements
- User feedback incorporation

**Human-in-the-Loop**:
- Forecasts are recommendations, not automated decisions
- Users can override or ignore recommendations
- Feedback mechanism to report inaccurate forecasts
- Manual review of high-impact recommendations

### 7.4 Compliance

**Indian Data Protection**:
- Data residency: All data stored in Indian data centers (AWS Mumbai, Azure India, etc.)
- Compliance with IT Act 2000 and upcoming data protection laws
- User consent for data collection and processing
- Clear privacy policy and terms of service

**Security Standards**:
- ISO 27001 compliance (target for production)
- Regular security audits and penetration testing
- Incident response plan
- Vulnerability management program

**User Rights**:
- Right to access: Users can download their data
- Right to deletion: Complete data removal on request
- Right to portability: Export data in standard formats (CSV, JSON)
- Right to explanation: Understand how forecasts are generated

## 8. Deployment Strategy

### 7.1 Infrastructure

**Cloud Provider**: AWS (or equivalent in India - AWS Mumbai region)

**Compute**:
- ECS/EKS for containerized microservices
- Auto-scaling groups for handling variable load
- Spot instances for batch processing (model training)

**Storage**:
- RDS PostgreSQL for relational data
- TimescaleDB (self-hosted on EC2 or managed) for time-series data
- S3 for object storage (models, reports, backups)

**Networking**:
- VPC with public and private subnets
- Application Load Balancer for traffic distribution
- CloudFront CDN for static assets

**Message Queue**:
- Amazon SQS or RabbitMQ for asynchronous processing

**Monitoring**:
- CloudWatch for infrastructure metrics
- Prometheus + Grafana for application metrics
- ELK Stack (Elasticsearch, Logstash, Kibana) for log aggregation

### 7.2 Deployment Pipeline

**CI/CD**:
```
Code Commit → GitHub Actions/GitLab CI →
Unit Tests → Integration Tests →
Build Docker Images → Push to ECR →
Deploy to Staging → Automated Tests →
Manual Approval → Deploy to Production
```

**Environments**:
- Development: Local Docker Compose
- Staging: Scaled-down production replica
- Production: Full infrastructure with redundancy

**Deployment Strategy**:
- Blue-Green deployment for zero-downtime updates
- Canary releases for gradual rollout
- Automated rollback on error threshold

### 7.3 Scalability Considerations

**Horizontal Scaling**:
- Stateless services scale independently
- Database read replicas for query load
- Caching layer (Redis) for frequently accessed data

**Data Partitioning**:
- Sales data partitioned by retailer_id and time
- Separate database instances for large retailers (future)

**Performance Optimization**:
- API response caching (5-minute TTL for forecasts)
- Lazy loading for dashboard components
- Pagination for large data sets
- Database indexing on frequently queried fields

### 7.4 Disaster Recovery

**Backup Strategy**:
- Automated daily database backups (retained 30 days)
- Point-in-time recovery capability
- Cross-region backup replication

**Recovery Objectives**:
- RTO (Recovery Time Objective): 4 hours
- RPO (Recovery Point Objective): 24 hours

**High Availability**:
- Multi-AZ deployment for databases
- Load balancer health checks and automatic failover
- Redundant service instances

## 8. Technology Stack Summary

| Component | Technology | Justification |
|-----------|-----------|---------------|
| API Gateway | Kong or AWS API Gateway | Rate limiting, authentication, routing |
| Backend Services | Python (FastAPI) | Fast development, rich ML ecosystem |
| Frontend | React.js + TypeScript | Modern, component-based, type-safe |
| Time-Series DB | TimescaleDB | PostgreSQL extension, familiar SQL interface |
| Relational DB | PostgreSQL | Robust, open-source, JSON support |
| Object Storage | AWS S3 | Scalable, cost-effective |
| ML Framework | Scikit-learn, XGBoost, Prophet | Proven, well-documented, efficient |
| Model Registry | MLflow | Open-source, experiment tracking |
| Message Queue | RabbitMQ or AWS SQS | Reliable async processing |
| Caching | Redis | Fast, in-memory, widely supported |
| Monitoring | Prometheus + Grafana | Open-source, flexible, powerful |
| Containerization | Docker + Kubernetes | Industry standard, portable |
| CI/CD | GitHub Actions | Integrated, easy to configure |

## 9. Security Architecture

### 9.1 Security Layers

**Network Security**:
- VPC isolation with security groups
- WAF (Web Application Firewall) for DDoS protection
- Private subnets for databases and internal services

**Application Security**:
- Input validation and sanitization
- SQL injection prevention (parameterized queries)
- XSS protection (Content Security Policy)
- CSRF tokens for state-changing operations

**Authentication & Authorization**:
- JWT with short expiration (15 minutes)
- Refresh tokens (7 days)
- API key authentication for programmatic access
- Rate limiting per user/IP

**Data Security**:
- Encryption at rest (AES-256)
- Encryption in transit (TLS 1.3)
- Secrets management (AWS Secrets Manager)
- Database connection encryption

**Audit & Compliance**:
- Comprehensive audit logs
- Regular security scans (OWASP ZAP, Snyk)
- Dependency vulnerability monitoring
- Annual penetration testing

## 10. Performance Requirements

### 10.1 Response Time Targets

| Operation | Target | Maximum |
|-----------|--------|---------|
| Dashboard load | 2s | 3s |
| API response (simple query) | 200ms | 500ms |
| CSV upload (10K rows) | 10s | 30s |
| Forecast generation | 1min | 2min |
| Report generation | 5s | 15s |

### 10.2 Throughput Targets

- Concurrent users: 1,000
- API requests: 10,000 req/min
- Data uploads: 100 concurrent uploads
- Forecast generation: 500 retailers/hour

## 11. Future Enhancements

### 11.1 Technical Improvements

- Real-time streaming data ingestion (Apache Kafka)
- Advanced deep learning models (Transformers for time-series)
- Automated hyperparameter tuning (Optuna, Ray Tune)
- GraphQL API for flexible data queries
- Mobile native apps (React Native)

### 11.2 Feature Additions

- Item-level (SKU) forecasting
- Supplier integration and price comparison
- Collaborative filtering for new retailers
- Anomaly detection for unusual sales patterns
- What-if scenario analysis

---

**Document Version:** 2.0  
**Last Updated:** February 14, 2026  
**Status:** Updated for Hackathon MVP - External Market Signals & Continuous Learning  
**Target Audience**: Small retailers in India (Bharat context)

## Key Design Highlights

1. **Hybrid Data Approach**: Combines retailer sales data with external market signals (search trends, festivals, pricing, demographics) to improve forecast stability
2. **Continuous Learning**: Daily feedback loop comparing predictions with actuals to improve model accuracy over time
3. **Explainable AI**: Every forecast includes clear explanations of contributing factors in plain language
4. **Similar Product Analysis**: Identifies products with correlated demand patterns for cross-selling opportunities
5. **Product-Market Fit Assessment**: Compares retailer performance with regional demand to identify opportunities
6. **Privacy-by-Design**: No PII collection, category-level aggregation only
7. **Operational Simplicity**: Designed for deployment with limited DevOps resources, suitable for small businesses
