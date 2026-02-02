
---

## 📁 File 12: `skills/07_risk_management.md`

```markdown
## SKILL-007: Risk Management & Position Sizing

**Feature ID:** RSK-001  
**Domain:** Risk Control  
**Scope:** Position sizing, drawdown control, correlation, exposure

---

### 7.1 Kelly Criterion Position Sizing

```cpp
class CPositionSizerKelly
{
private:
   double            m_win_rate;      // 0.0 - 1.0
   double            m_win_loss_ratio; // Average win / Average loss
   
public:
   void SetWinRate(double rate) { m_win_rate = rate; }
   void SetWinLossRatio(double ratio) { m_win_loss_ratio = ratio; }
   
   double CalculateKellyFraction()
   {
      // Kelly % = W - [(1 - W) / R]
      // W = Win rate, R = Win/Loss ratio
      
      if(m_win_loss_ratio <= 0) return 0;
      
      double kelly = m_win_rate - ((1 - m_win_rate) / m_win_loss_ratio);
      
      // Use Half-Kelly for safety
      return kelly * 0.5;
   }
   
   double CalculateLots(double account_balance, double stop_loss_points)
   {
      double fraction = CalculateKellyFraction();
      if(fraction <= 0) return 0;
      
      double risk_amount = account_balance * fraction;
      return RiskAmountToLots(risk_amount, stop_loss_points);
   }
   
private:
   double RiskAmountToLots(double risk_amount, double stop_points)
   {
      double tick_value = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_VALUE);
      double tick_size = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_SIZE);
      double point = SymbolInfoDouble(_Symbol, SYMBOL_POINT);
      
      if(tick_value == 0 || tick_size == 0) return 0;
      
      double ticks_at_risk = stop_points * point / tick_size;
      double lots = risk_amount / (ticks_at_risk * tick_value);
      
      return NormalizeLots(lots);
   }
   
   double NormalizeLots(double lots)
   {
      double lot_step = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_STEP);
      double min_lot = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_MIN);
      
      lots = MathFloor(lots / lot_step) * lot_step;
      lots = NormalizeDouble(lots, 2);
      
      if(lots < min_lot) return 0;
      return lots;
   }
};

---

### 7.2 Drawdown Monitor & Circuit Breaker

```cpp
class CDrawdownMonitor
{
private:
   double            m_initial_equity;
   double            m_peak_equity;
   double            m_max_drawdown_percent;  // e.g., 10.0 for 10%
   bool              m_trading_blocked;
   datetime          m_block_time;
   
public:
   void Initialize(double max_dd_percent)
   {
      m_max_drawdown_percent = max_dd_percent;
      m_initial_equity = AccountInfoDouble(ACCOUNT_EQUITY);
      m_peak_equity = m_initial_equity;
      m_trading_blocked = false;
   }
   
   bool CheckDrawdown()
   {
      if(m_trading_blocked)
      {
         // Check if we can resume (next day)
         if(TimeCurrent() > m_block_time + 86400)
         {
            m_trading_blocked = false;
            m_initial_equity = AccountInfoDouble(ACCOUNT_EQUITY);
            m_peak_equity = m_initial_equity;
            Print("Trading resumed after drawdown recovery period");
         }
         return false;
      }
      
      double current_equity = AccountInfoDouble(ACCOUNT_EQUITY);
      
      // Update peak
      if(current_equity > m_peak_equity)
         m_peak_equity = current_equity;
      
      // Calculate drawdown
      double drawdown = (m_peak_equity - current_equity) / m_peak_equity * 100.0;
      
      if(drawdown >= m_max_drawdown_percent)
      {
         Print("CRITICAL: Max drawdown reached: ", drawdown, "%");
         BlockTrading();
         return false;
      }
      
      return true;
   }
   
   bool IsTradingAllowed() const
   {
      return !m_trading_blocked;
   }
   
   double GetCurrentDrawdown() const
   {
      double current = AccountInfoDouble(ACCOUNT_EQUITY);
      return (m_peak_equity - current) / m_peak_equity * 100.0;
   }
   
private:
   void BlockTrading()
   {
      m_trading_blocked = true;
      m_block_time = TimeCurrent();
      
      // Close all positions
      CloseAllPositions();
      
      // Send alert
      SendNotification("Trading stopped: Max drawdown reached");
   }
   
   void CloseAllPositions()
   {
      CTrade trade;
      CPositionInfo pos;
      
      for(int i = PositionsTotal() - 1; i >= 0; i--)
      {
         if(pos.SelectByIndex(i))
            trade.PositionClose(pos.Ticket());
      }
   }
};

---

### 7.3 Correlation Risk Manager

```cpp
struct SymbolExposure
{
   string            symbol;
   double            direction;  // +1 for buy, -1 for sell
   double            lots;
   double            pip_value;
};

class CCorrelationRiskManager
{
private:
   SymbolExposure    m_exposures[];
   
public:
   void AddPosition(string symbol, ENUM_POSITION_TYPE type, double lots)
   {
      int idx = ArraySize(m_exposures);
      ArrayResize(m_exposures, idx + 1);
      
      m_exposures[idx].symbol = symbol;
      m_exposures[idx].direction = (type == POSITION_TYPE_BUY) ? 1.0 : -1.0;
      m_exposures[idx].lots = lots;
      m_exposures[idx].pip_value = CalculatePipValue(symbol);
   }
   
   double GetTotalExposure()
   {
      double total = 0;
      
      for(int i = 0; i < ArraySize(m_exposures); i++)
      {
         total += m_exposures[i].lots * m_exposures[i].direction * 
                  m_exposures[i].pip_value;
      }
      
      return total;
   }
   
   bool WouldExceedExposure(string new_symbol, double new_lots, 
                           ENUM_ORDER_TYPE type, double max_exposure)
   {
      double additional = new_lots * CalculatePipValue(new_symbol);
      if(type == ORDER_TYPE_SELL) additional *= -1;
      
      return MathAbs(GetTotalExposure() + additional) > max_exposure;
   }
   
   bool AreCorrelated(string sym1, string sym2, double threshold = 0.8)
   {
      // Simplified: check if same base or quote currency
      string base1 = StringSubstr(sym1, 0, 3);
      string quote1 = StringSubstr(sym1, 3, 3);
      string base2 = StringSubstr(sym2, 0, 3);
      string quote2 = StringSubstr(sym2, 3, 3);
      
      double correlation = 0;
      
      if(base1 == base2) correlation = 1.0;  // Same base (EURUSD/EURGBP)
      else if(quote1 == quote2) correlation = -0.5;  // Same quote
      else if(base1 == quote2 || base2 == quote1) correlation = -0.8;  // Inverse
      
      return MathAbs(correlation) >= threshold;
   }
   
private:
   double CalculatePipValue(string symbol)
   {
      double tick_value = SymbolInfoDouble(symbol, SYMBOL_TRADE_TICK_VALUE);
      double tick_size = SymbolInfoDouble(symbol, SYMBOL_TRADE_TICK_SIZE);
      
      if(tick_size == 0) return 0;
      
      // For 5-digit brokers, 1 pip = 10 points
      return tick_value * (0.0001 / tick_size);
   }
};

---

### 7.4 Risk Per Trade Calculator

```cpp
class CRiskPerTrade
{
public:
   enum ENUM_RISK_TYPE
   {
      RISK_FIXED_LOTS,           // Fixed lot size
      RISK_FIXED_AMOUNT,         // Risk fixed $ amount
      RISK_PERCENT_BALANCE,      // Risk % of balance
      RISK_PERCENT_EQUITY,       // Risk % of equity
      RISK_PERCENT_AVAIL_MARGIN  // Risk % of free margin
   };
   
   static double CalculateLots(ENUM_RISK_TYPE type, double risk_value, 
                              double stop_loss_points, double slippage_points = 0)
   {
      if(stop_loss_points <= 0) return 0;
      
      double account_base = 0;
      
      switch(type)
      {
         case RISK_FIXED_LOTS:
            return risk_value;
            
         case RISK_FIXED_AMOUNT:
            return AmountToLots(risk_value, stop_loss_points);
            
         case RISK_PERCENT_BALANCE:
            account_base = AccountInfoDouble(ACCOUNT_BALANCE);
            return AmountToLots(account_base * risk_value / 100.0, stop_loss_points);
            
         case RISK_PERCENT_EQUITY:
            account_base = AccountInfoDouble(ACCOUNT_EQUITY);
            return AmountToLots(account_base * risk_value / 100.0, stop_loss_points);
            
         case RISK_PERCENT_AVAIL_MARGIN:
            account_base = AccountInfoDouble(ACCOUNT_MARGIN_FREE);
            return AmountToLots(account_base * risk_value / 100.0, stop_loss_points);
      }
      
      return 0;
   }
   
private:
   static double AmountToLots(double amount, double stop_points)
   {
      double tick_value = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_VALUE);
      double tick_size = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_SIZE);
      double point = SymbolInfoDouble(_Symbol, SYMBOL_POINT);
      
      if(tick_value == 0 || tick_size == 0 || point == 0) return 0;
      
      // Calculate number of ticks at risk
      double ticks = stop_points * point / tick_size;
      
      // Calculate lot size: Amount / (Ticks * TickValue)
      double lots = amount / (ticks * tick_value);
      
      return NormalizeLots(lots);
   }
   
   static double NormalizeLots(double lots)
   {
      double step = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_STEP);
      double min = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_MIN);
      double max = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_MAX);
      
      lots = MathFloor(lots / step) * step;
      lots = NormalizeDouble(lots, 2);
      
      if(lots < min) return 0;
      if(lots > max) return max;
      
      return lots;
   }
};

---

### 7.5 Volatility-Based Sizing (ATR)

```cpp
class CVolatilitySizer
{
private:
   int               m_atr_handle;
   double            m_risk_percent;
   double            m_atr_multiplier;
   
public:
   bool Initialize(int atr_period, double risk_percent, double atr_mult)
   {
      m_atr_handle = iATR(_Symbol, PERIOD_CURRENT, atr_period);
      if(m_atr_handle == INVALID_HANDLE) return false;
      
      m_risk_percent = risk_percent;
      m_atr_multiplier = atr_mult;
      return true;
   }
   
   void Deinit()
   {
      IndicatorRelease(m_atr_handle);
   }
   
   double CalculateLots()
   {
      double atr[];
      if(CopyBuffer(m_atr_handle, 0, 0, 1, atr) <= 0) return 0;
      
      // Stop loss = ATR * multiplier
      double stop_distance = atr[0] * m_atr_multiplier;
      
      // Convert to points
      double point = SymbolInfoDouble(_Symbol, SYMBOL_POINT);
      double stop_points = stop_distance / point;
      
      if(stop_points <= 0) return 0;
      
      // Calculate based on risk percent
      double balance = AccountInfoDouble(ACCOUNT_BALANCE);
      double risk_amount = balance * m_risk_percent / 100.0;
      
      return RiskToLots(risk_amount, stop_points);
   }
   
private:
   double RiskToLots(double amount, double stop_points)
   {
      double tick_value = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_VALUE);
      double tick_size = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_SIZE);
      double point = SymbolInfoDouble(_Symbol, SYMBOL_POINT);
      
      double ticks = stop_points * point / tick_size;
      double lots = amount / (ticks * tick_value);
      
      // Normalize
      double step = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_STEP);
      return MathFloor(lots / step) * step;
   }
};