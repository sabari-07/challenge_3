# Quick Reference Guide

## 🎯 Which Files Should I Use?

### **For Hackathon Production Code:**

```python
# ✅ RECOMMENDED PRODUCTION STACK

# 1. Data Loading
from src.data.loader import DataLoader

# 2. Data Preprocessing  
from src.data.preprocessor_improved import ImprovedDataPreprocessor

# 3. Forecasting Model
from src.models.prophet_final import FinalProphetForecaster

# 4. AI Insights (optional)
from src.ai.cohere_client import CohereClient
```

---

## 📊 Model Selection Guide

| Model File | When to Use | Accuracy (30d) | Complexity |
|------------|-------------|----------------|------------|
| **`prophet_final.py`** | **Production/Hackathon** ✅ | Google 2.8%, Bing 12.9%, Meta 19.9% | Medium |
| `prophet_improved.py` | Backup/Alternative | ~4.79% aggregated | Medium |
| `xgboost_model.py` | High accuracy needed + manual tuning OK | 3.68% aggregated | High |
| `prophet_model.py` | Don't use (legacy) | 10.57% aggregated | Low |
| `ensemble.py` | Experimental only | Unknown | Very High |
| `intelligent_hybrid.py` | Experimental only | Unknown | Very High |

**Default Choice:** `prophet_final.py`

---

## 🔧 Complete Example

```python
# Step 1: Load data
from src.data.loader import DataLoader
loader = DataLoader("./data")
df = loader.load_all_channels()

# Step 2: Preprocess
from src.data.preprocessor_improved import ImprovedDataPreprocessor
preprocessor = ImprovedDataPreprocessor()
df_clean = preprocessor.prepare_all_channels(df)

# Step 3: Train model (30-day forecast)
from src.models.prophet_final import FinalProphetForecaster

google_data = df_clean[df_clean['channel'] == 'google']
model = FinalProphetForecaster(channel='google', forecast_horizon=30)
model.fit(google_data)

# Step 4: Forecast
forecast = model.predict(periods=30)
summary = model.get_forecast_summary(forecast, periods=30)

print(f"Expected Revenue: ${summary['p50']:,.2f}")
print(f"Range: ${summary['p10']:,.2f} - ${summary['p90']:,.2f}")

# Step 5: Budget simulation
forecast_custom = model.predict_with_custom_spend(periods=30, daily_spend=5000)
summary_custom = model.get_forecast_summary(forecast_custom, periods=30)
print(f"With $5K/day: ${summary_custom['p50']:,.2f}")
```

---

## 📁 File Purpose Summary

### **Data Processing**
- `loader.py` → Loads CSVs, standardizes columns
- `preprocessor_improved.py` → **USE THIS** - Cleans data (gaps, outliers, zeros)
- `preprocessor.py` → Legacy (basic preprocessing)

### **Models**
- `prophet_final.py` → **USE THIS** - Best accuracy + all optimizations
- `prophet_improved.py` → Backup (slightly worse)
- `xgboost_model.py` → Alternative (complex, needs features)
- Others → Experimental/legacy

### **Utilities**
- `generate_features.py` → For XGBoost (if you use it)
- `budget_optimizer.py` → Budget allocation optimization
- `cohere_client.py` → AI insights generation

---

## 🚦 Decision Tree

```
Need to forecast revenue?
    │
    ├─→ Google/Bing/Meta data?
    │   │
    │   ├─→ YES → Use prophet_final.py ✅
    │   │
    │   └─→ NO → Use dynamic_loader.py + prophet_final.py
    │
    └─→ Need extreme accuracy + OK with complexity?
        │
        └─→ Use xgboost_model.py (requires feature engineering)
```

---

## ⚡ Performance Comparison

### **30-Day Forecast**

| Model | Google | Bing | Meta |
|-------|--------|------|------|
| **prophet_final (current)** | ~2.8% | ~12.9% | ~19.9% |
| prophet_original (pre-fixes) | 2.8% | 93.3%* | 56.2% |

*Bing 60d was 93.3% before including all campaign types; now ~8.4%

**Winner:** `prophet_final.py` — correct channel-specific configs, sub-model split for Meta, data-driven preprocessing

---

## 🎯 For Different Horizons

```python
# 30-day forecast (best accuracy)
model_30 = FinalProphetForecaster(channel='google', forecast_horizon=30)

# 60-day forecast
model_60 = FinalProphetForecaster(channel='google', forecast_horizon=60)

# 90-day forecast (wider intervals)
model_90 = FinalProphetForecaster(channel='google', forecast_horizon=90)
```

**Why this matters:**
- Automatically adjusts interval width (30d:80%, 60d:85%, 90d:95%)
- Adds rolling regressors for Meta 60/90-day
- Optimizes changepoint flexibility for longer horizons

---

## 🔍 What Each Preprocessing Step Does

### **Bing:**
```python
# preprocessor_improved.py - prepare_bing()
# 1. All campaign types included (Search + PerformanceMax + Shopping)
# 2. Training start auto-detected via 14-day rolling revenue window
# 3. AOV derived from data: median(revenue / conversions) on active days
# 4. Zero-revenue days filled: conversions × AOV, floor at median revenue
# Result: Bing 60d 93.3% → ~8.4%
```

### **Meta:**
```python
# preprocessor_improved.py - prepare_meta()
# 1. Campaign rows tagged Remarketing/Prospecting via keyword matching
# 2. Each sub-series cleaned independently (gap fill, outlier cap)
# 3. Longest contiguous missing-date gap (≥30 days) detected; training from day after
# 4. conversion column already USD revenue — no AOV scaling
# Result: Meta 30d 56.2% → ~19.9%
```

### **Google:**
```python
# preprocessor_improved.py - prepare_google()
# Minimal processing (data already clean)
# Spend regressor retained for budget simulation
```

---

## 💡 Common Questions

**Q: Should I use ensemble.py?**  
A: No, not tested. Stick with prophet_final.py.

**Q: Why not use xgboost_model.py if it has 3.68% error?**  
A: Lower coverage (78.9% vs 92.2%) and won't generalize to new CSVs.

**Q: Which preprocessor should I use?**  
A: Always use `preprocessor_improved.py`. It has all the fixes.

**Q: Can I forecast at different horizons?**  
A: Yes, just change `forecast_horizon` parameter (30, 60, or 90).

**Q: Do I need to tune parameters?**  
A: No, `prophet_final.py` has channel-specific configs built-in.

---

## 📝 Key Takeaways

✅ **Use:** `loader.py` + `preprocessor_improved.py` + `prophet_final.py`  
✅ **Works for:** Any advertiser's Google Ads, Bing Ads, Meta Ads export (generic, no hardcoded constants)  
✅ **Features:** Budget simulation (Google), uncertainty intervals, auto-preprocessing, Meta sub-model split  
✅ **Regressors:** Google uses spend; Bing uses conversions; Meta has no spend regressor (spend too volatile)  
❌ **Avoid:** `prophet_model.py` (legacy), experimental models

---

**Last Updated:** 2026-07-13  
**Recommended Stack:** loader.py → preprocessor_improved.py → prophet_final.py
