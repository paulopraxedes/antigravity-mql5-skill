
---

## 📁 File 11: `skills/06_standard_library.md`

```markdown
## SKILL-006: Standard Library Deep Usage

**Feature ID:** LIB-001  
**Domain:** Framework  
**Scope:** CTrade, CPositionInfo, COrderInfo, CSymbolInfo, CAccountInfo

---

### 6.1 CTrade - Complete Configuration

```cpp
#include <Trade/Trade.mqh>

class CTradeManager
{
private:
   CTrade            m_trade;
   bool              m_initialized;
   
public:
   bool Initialize(ulong magic, int deviation, ENUM_ORDER_TYPE_FILLING filling)
   {
      //--- Magic number (CRITICAL)
      if(!m_trade.SetExpertMagicNumber(magic))
         return false;
      
      //--- Slippage/deviation in points (not pips)
      m_trade.SetDeviationInPoints(deviation);
      
      //--- Filling mode
      if(!m_trade.SetTypeFilling(filling))
      {
         // Try fallback modes
         if(!m_trade.SetTypeFilling(ORDER_FILLING_IOC))
            if(!m_trade.SetTypeFilling(ORDER_FILLING_RETURN))
               return false;
      }
      
      //--- Margin check mode
      m_trade.SetAsyncMode(false);  // Wait for result (synchronous)
      
      //--- Log level
      m_trade.SetLogLevel(LOG_LEVEL_ERRORS);
      
      m_initialized = true;
      return true;
   }
   
   //--- Wrapper with full error handling
   bool OpenPosition(ENUM_ORDER_TYPE type, double volume, double price, 
                     double sl, double tp, string comment)
   {
      if(!m_trade.PositionOpen(Symbol(), type, volume, price, sl, tp, comment))
      {
         Print("Trade failed. Retcode: ", m_trade.ResultRetcode(),
               " Deal ticket: ", m_trade.ResultDeal(),
               " Order ticket: ", m_trade.ResultOrder(),
               " Volume: ", m_trade.ResultVolume(),
               " Price: ", m_trade.ResultPrice());
         return false;
      }
      return true;
   }
   
   //--- Bulk close by type
   void CloseAllPositions(ulong magic_filter = 0)
   {
      CPositionInfo pos;
      for(int i = PositionsTotal() - 1; i >= 0; i--)
      {
         if(pos.SelectByIndex(i))
         {
            if(magic_filter != 0 && pos.Magic() != magic_filter)
               continue;
               
            if(!m_trade.PositionClose(pos.Ticket()))
            {
               Print("Failed to close position ", pos.Ticket());
            }
         }
      }
   }
   
   //--- Modify all positions
   void ModifyAllPositions(double new_sl, double new_tp, ulong magic_filter)
   {
      CPositionInfo pos;
      for(int i = PositionsTotal() - 1; i >= 0; i--)
      {
         if(pos.SelectByIndex(i))
         {
            if(magic_filter != 0 && pos.Magic() != magic_filter)
               continue;
               
            if(!m_trade.PositionModify(pos.Ticket(), new_sl, new_tp))
            {
               Print("Failed to modify position ", pos.Ticket());
            }
         }
      }
   }
};

---

### 6.2 CSymbolInfo - Complete Market Data

```cpp
#include <Trade/SymbolInfo.mqh>

class CMarketData
{
private:
   CSymbolInfo       m_symbol;
   datetime          m_last_update;
   
public:
   bool Initialize(string symbol)
   {
      if(!m_symbol.Name(symbol))
         return false;
         
      if(!SelectInMarketWatch())
         return false;
         
      return Refresh();
   }
   
   bool Refresh()
   {
      if(!m_symbol.RefreshRates())
         return false;
         
      m_last_update = TimeCurrent();
      return true;
   }
   
   bool IsTradeAllowed() const
   {
      return m_symbol.IsTradable() && !m_symbol.IsStopsLevelFrozen();
   }
   
   double GetSpreadPoints() const
   {
      return (m_symbol.Ask() - m_symbol.Bid()) / m_symbol.Point();
   }
   
   bool CheckStops(ENUM_ORDER_TYPE type, double price, double sl, double tp) const
   {
      int stops_level = (int)m_symbol.StopsLevel();
      if(stops_level == 0) return true;
      
      double min_dist = stops_level * m_symbol.Point();
      
      if(type == ORDER_TYPE_BUY)
      {
         if(sl > 0 && (price - sl) < min_dist) return false;
         if(tp > 0 && (tp - price) < min_dist) return false;
      }
      else
      {
         if(sl > 0 && (sl - price) < min_dist) return false;
         if(tp > 0 && (price - tp) < min_dist) return false;
      }
      
      return true;
   }
   
   double NormalizeLot(double lots) const
   {
      double lot_step = m_symbol.LotsStep();
      double min_lot = m_symbol.LotsMin();
      double max_lot = m_symbol.LotsMax();
      
      lots = MathFloor(lots / lot_step) * lot_step;
      lots = NormalizeDouble(lots, 2);
      
      if(lots < min_lot) return 0;
      if(lots > max_lot) return max_lot;
      
      return lots;
   }
   
   double CalculateMargin(ENUM_ORDER_TYPE type, double lots, double price) const
   {
      double margin;
      if(!OrderCalcMargin(type, m_symbol.Name(), lots, price, margin))
         return -1;
      return margin;
   }
   
   double GetPipValue() const
   {
      // For 5-digit brokers: 1 pip = 10 points
      double point = m_symbol.Point();
      int digits = m_symbol.Digits();
      double tick_value = m_symbol.TickValue();
      double tick_size = m_symbol.TickSize();
      
      if(digits >= 4) // Forex pairs
         return tick_value * (0.0001 / tick_size);  // 1 pip
      else
         return tick_value;
   }
   
private:
   bool SelectInMarketWatch()
   {
      if(!m_symbol.Select())
         return SymbolSelect(m_symbol.Name(), true);
      return true;
   }
};

---

### 6.3 Account Information Wrapper

```cpp
#include <Trade/AccountInfo.mqh>

class CAccountManager
{
private:
   CAccountInfo      m_account;
   
public:
   string GetInfo()
   {
      string info = "";
      info += "Login: " + IntegerToString(m_account.Login()) + "\n";
      info += "Name: " + m_account.Name() + "\n";
      info += "Server: " + m_account.Server() + "\n";
      info += "Currency: " + m_account.Currency() + "\n";
      info += "Leverage: 1:" + IntegerToString(m_account.Leverage()) + "\n";
      info += "Balance: " + DoubleToString(m_account.Balance(), 2) + "\n";
      info += "Equity: " + DoubleToString(m_account.Equity(), 2) + "\n";
      info += "Margin: " + DoubleToString(m_account.Margin(), 2) + "\n";
      info += "Free Margin: " + DoubleToString(m_account.FreeMargin(), 2) + "\n";
      info += "Margin Level: " + DoubleToString(m_account.MarginLevel(), 2) + "%\n";
      return info;
   }
   
   bool IsValidForTrading(double required_margin = 0)
   {
      if(m_account.TradeAllowed() != ACCOUNT_TRADE_MODE_ALLOWED)
         return false;
         
      if(m_account.MarginLevel() < 100 && m_account.Margin() > 0)  // Margin call
         return false;
         
      if(required_margin > 0 && m_account.FreeMargin() < required_margin)
         return false;
         
      return true;
   }
   
   ENUM_ACCOUNT_TRADE_MODE GetTradeMode() const
   {
      return (ENUM_ACCOUNT_TRADE_MODE)m_account.TradeMode();
   }
   
   bool IsDemo() const
   {
      return GetTradeMode() == ACCOUNT_TRADE_MODE_DEMO;
   }
   
   bool IsReal() const
   {
      return GetTradeMode() == ACCOUNT_TRADE_MODE_REAL;
   }
};


---

### 6.4 CDealInfo - Trade History Analysis

```cpp
#include <Trade/DealInfo.mqh>

class CTradeHistoryAnalyzer
{
private:
   CDealInfo         m_deal;
   CHistoryOrderInfo m_history;
   int               m_total_deals;
   
public:
   void LoadHistory(datetime from, datetime to)
   {
      HistorySelect(from, to);
      m_total_deals = HistoryDealsTotal();
   }
   
   double CalculateProfit(ulong magic_filter = 0)
   {
      double total = 0;
      
      for(int i = 0; i < m_total_deals; i++)
      {
         ulong ticket = HistoryDealGetTicket(i);
         if(ticket == 0) continue;
         
         if(!m_deal.SelectByTicket(ticket))
            continue;
            
         if(magic_filter != 0 && m_deal.Magic() != magic_filter)
            continue;
            
         if(m_deal.Entry() == DEAL_ENTRY_OUT || m_deal.Entry() == DEAL_ENTRY_OUT_BY)
         {
            total += m_deal.Profit() + m_deal.Commission() + m_deal.Swap();
         }
      }
      
      return total;
   }
   
   void GetStatistics(double &profit, int &wins, int &losses, int &total)
   {
      profit = 0;
      wins = 0;
      losses = 0;
      total = 0;
      
      for(int i = 0; i < m_total_deals; i++)
      {
         ulong ticket = HistoryDealGetTicket(i);
         if(ticket == 0) continue;
         
         if(!m_deal.SelectByTicket(ticket))
            continue;
            
         if(m_deal.Entry() != DEAL_ENTRY_OUT) continue;
         
         total++;
         double p = m_deal.Profit();
         profit += p;
         
         if(p > 0) wins++;
         else if(p < 0) losses++;
      }
   }
};

---

### 6.5 Terminal State Detection

```cpp
class CEnvironment
{
public:
   static bool IsBacktesting()
   {
      return IsTesting();
   }
   
   static bool IsOptimization()
   {
      return IsOptimization();
   }
   
   static bool IsVisualMode()
   {
      return IsVisualMode();
   }
   
   static bool IsRealtime()
   {
      return !IsTesting() && !IsOptimization();
   }
   
   static bool IsTradeAllowed()
   {
      return IsTradeAllowed() && IsConnected();
   }
   
   static bool IsStopped()
   {
      return ::IsStopped();
   }
   
   static void CheckMemory()
   {
      long available = TerminalInfoInteger(TERMINAL_MEMORY_PHYSICAL);
      long used = TerminalInfoInteger(TERMINAL_MEMORY_TOTAL);
      
      if(available < 100)  // Less than 100MB
      {
         Print("WARNING: Low memory available: ", available, " MB");
      }
   }
};