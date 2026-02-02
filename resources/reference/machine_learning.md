
---

## 📁 File 18: `skills/13_machine_learning.md`

```markdown
## SKILL-013: Machine Learning Integration (ONNX)

**Feature ID:** ML-001  
**Domain:** Artificial Intelligence  
**Scope:** ONNX models, inference, feature engineering

---

### 13.1 ONNX Model Manager

```cpp
class COnnxModel
{
private:
   long              m_model;
   string            m_model_path;
   int               m_input_dims[];
   int               m_output_dims[];
   bool              m_loaded;
   
public:
   COnnxModel() : m_model(0), m_loaded(false) {}
   
   ~COnnxModel() { Unload(); }
   
   bool Load(string path, int &input_shape[], int &output_shape[])
   {
      m_model_path = path;
      
      //--- Load model
      m_model = OnnxCreate(path, 0);
      if(m_model == INVALID_HANDLE)
      {
         Print("Failed to load ONNX model: ", path);
         return false;
      }
      
      //--- Configure shapes
      ArrayCopy(m_input_dims, input_shape);
      ArrayCopy(m_output_dims, output_shape);
      
      for(int i = 0; i < ArraySize(input_shape); i++)
      {
         if(!OnnxSetInputShape(m_model, i, input_shape[i]))
         {
            Print("Failed to set input shape");
            Unload();
            return false;
         }
      }
      
      for(int i = 0; i < ArraySize(output_shape); i++)
      {
         if(!OnnxSetOutputShape(m_model, i, output_shape[i]))
         {
            Print("Failed to set output shape");
            Unload();
            return false;
         }
      }
      
      m_loaded = true;
      Print("ONNX model loaded successfully: ", path);
      return true;
   }
   
   bool Predict(const matrix &input_features, matrix &predictions)
   {
      if(!m_loaded) return false;
      
      //--- Run inference
      matrix output;
      if(!OnnxRun(m_model, ONNX_TYPE_DOUBLE, input_features, output))
      {
         Print("ONNX inference failed");
         return false;
      }
      
      predictions.Init(output.Rows(), output.Cols());
      output.CopyTo(predictions);
      
      return true;
   }
   
   void Unload()
   {
      if(m_model != 0)
      {
         OnnxRelease(m_model);
         m_model = 0;
         m_loaded = false;
      }
   }
};

---

### 13.2 Feature Engineering for Price Prediction

```cpp
class CFeatureEngineer
{
public:
   //--- Create feature vector from price history
   static vector CreateFeatures(MqlRates &rates[], int index, int lookback)
   {
      vector features;
      features.Resize(lookback * 5);  // OHLCV
      
      int feature_idx = 0;
      for(int i = index; i < index + lookback && i < ArraySize(rates); i++)
      {
         features[feature_idx++] = rates[i].open;
         features[feature_idx++] = rates[i].high;
         features[feature_idx++] = rates[i].low;
         features[feature_idx++] = rates[i].close;
         features[feature_idx++] = (double)rates[i].tick_volume;
      }
      
      //--- Normalize (Z-score)
      double mean = features.Mean();
      double std = features.Std();
      
      if(std > 0)
      {
         for(int i = 0; i < features.Size(); i++)
         {
            features[i] = (features[i] - mean) / std;
         }
      }
      
      return features;
   }
   
   //--- Technical indicators as features
   static vector CreateTAFeatures(string symbol, ENUM_TIMEFRAMES period)
   {
      vector features;
      features.Resize(10);
      
      int ma_fast = iMA(symbol, period, 10, 0, MODE_SMA, PRICE_CLOSE);
      int ma_slow = iMA(symbol, period, 20, 0, MODE_SMA, PRICE_CLOSE);
      int rsi = iRSI(symbol, period, 14, PRICE_CLOSE);
      int atr = iATR(symbol, period, 14);
      
      double ma_f[], ma_s[], r[], a[];
      CopyBuffer(ma_fast, 0, 0, 1, ma_f);
      CopyBuffer(ma_slow, 0, 0, 1, ma_s);
      CopyBuffer(rsi, 0, 0, 1, r);
      CopyBuffer(atr, 0, 0, 1, a);
      
      features[0] = ma_f[0];
      features[1] = ma_s[0];
      features[2] = r[0];
      features[3] = a[0];
      
      // Cleanup
      IndicatorRelease(ma_fast);
      IndicatorRelease(ma_slow);
      IndicatorRelease(rsi);
      IndicatorRelease(atr);
      
      return features;
   }
};

---

### 13.3 Trading Signal Classifier

```cpp
class CMLSignalGenerator
{
private:
   COnnxModel        m_model;
   CFeatureEngineer  m_features;
   int               m_lookback;
   
public:
   bool Initialize(string model_path, int input_shape[], int output_shape[])
   {
      m_lookback = 20;  // Bars to look back
      return m_model.Load(model_path, input_shape, output_shape);
   }
   
   ENUM_SIGNAL GetSignal()
   {
      //--- Get recent data
      MqlRates rates[];
      ArraySetAsSeries(rates, true);
      if(CopyRates(_Symbol, PERIOD_CURRENT, 0, m_lookback, rates) < m_lookback)
         return SIGNAL_NONE;
      
      //--- Create feature matrix (1 sample, N features)
      matrix input;
      input.Init(1, m_lookback * 5);
      
      vector feats = CFeatureEngineer::CreateFeatures(rates, 0, m_lookback);
      for(int i = 0; i < feats.Size(); i++)
      {
         input[0][i] = feats[i];
      }
      
      //--- Predict
      matrix output;
      if(!m_model.Predict(input, output))
         return SIGNAL_NONE;
      
      //--- Interpret output (assuming softmax: [hold, buy, sell])
      double hold_prob = output[0][0];
      double buy_prob = output[0][1];
      double sell_prob = output[0][2];
      
      if(buy_prob > 0.7 && buy_prob > hold_prob && buy_prob > sell_prob)
         return SIGNAL_BUY;
      else if(sell_prob > 0.7 && sell_prob > hold_prob && sell_prob > buy_prob)
         return SIGNAL_SELL;
      
      return SIGNAL_HOLD;
   }
   
   void Shutdown()
   {
      m_model.Unload();
   }
};

enum ENUM_SIGNAL
{
   SIGNAL_NONE = 0,
   SIGNAL_HOLD = 1,
   SIGNAL_BUY = 2,
   SIGNAL_SELL = 3
};