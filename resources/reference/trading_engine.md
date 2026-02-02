
---

## 📁 File 5: `skills/04_trading_engine.md`

```markdown
## SKILL-004: Trading Engine Core

**Feature ID:** TRD-001  
**Domain:** Order Execution  
**Scope:** Order placement, position management, error recovery

---

### 4.1 Modern Approach: CTrade Class (RECOMMENDED)

```cpp
#include <Trade/Trade.mqh>

class CExecutionManager {
private:
   CTrade            m_trade;
   CSymbolInfo       m_symbol;
   CPositionInfo     m_position;
   ulong             m_magic;
   
public:
   bool              Initialize(string symbol, ulong magic, int deviation_points = 10) {
      if(!m_symbol.Name(symbol)) return false;
      if(!m_symbol.Refresh()) return false;
      
      m_magic = magic;
      
      //--- Configure trade object
      m_trade.SetExpertMagicNumber(magic);
      m_trade.SetDeviationInPoints(deviation_points);
      m_trade.SetTypeFilling(ORDER_FILLING_IOC);
      m_trade.SetAsyncMode(false);  // Synchronous for safety
      
      return true;
   }
   
   bool              OpenMarketOrder(ENUM_ORDER_TYPE type, double lots, 
                                     double sl_price = 0, double tp_price = 0) {
      //--- Preconditions
      if(!m_symbol.Refresh()) return false;
      if(!IsTradeAllowed()) return false;
      
      //--- Normalize lot size
      lots = NormalizeLots(lots);
      if(lots <= 0) {
         Print("Invalid lot size after normalization");
         return false;
      }
      
      //--- Get execution price
      double price = (type == ORDER_TYPE_BUY) ? m_symbol.Ask() : m_symbol.Bid();
      
      //--- Validate stops against stop level
      if(!ValidateStops(type, price, sl_price, tp_price)) {
         return false;
      }
      
      //--- Check margin
      if(!CheckMargin(type, lots, price)) {
         Print("Insufficient margin");
         return false;
      }
      
      //--- Execute
      if(!m_trade.PositionOpen(m_symbol.Name(), type, lots, price, 
                               sl_price, tp_price, "EA Order")) {
         Print("PositionOpen failed: ", GetRetcodeDescription(m_trade.ResultRetcode()));
         return false;
      }
      
      Print("Order executed successfully. Ticket: ", m_trade.ResultOrder());
      return true;
   }
   
   bool              ModifyPosition(ulong ticket, double sl, double tp) {
      if(!m_position.SelectByTicket(ticket)) {
         Print("Position not found: ", ticket);
         return false;
      }
      
      //--- Validate modification
      ENUM_POSITION_TYPE pos_type = m_position.PositionType();
      double open_price = m_position.PriceOpen();
      
      if(!ValidateStops(pos_type, open_price, sl, tp)) {
         return false;
      }
      
      if(!m_trade.PositionModify(ticket, sl, tp)) {
         Print("Modify failed: ", m_trade.ResultRetcode());
         return false;
      }
      
      return true;
   }
   
   bool              ClosePosition(ulong ticket) {
      if(!m_position.SelectByTicket(ticket)) return false;
      
      if(!m_trade.PositionClose(ticket)) {
         if(m_trade.ResultRetcode() == TRADE_RETCODE_INVALID_TICKET) {
            Print("Ticket no longer valid, position may be already closed");
            return true;  // Consider success
         }
         return false;
      }
      return true;
   }
   
private:
   double            NormalizeLots(double lots) {
      double lot_step = m_symbol.LotsStep();
      double min_lot = m_symbol.LotsMin();
      double max_lot = m_symbol.LotsMax();
      
      //--- Step size
      lots = MathFloor(lots / lot_step) * lot_step;
      lots = NormalizeDouble(lots, 2);
      
      //--- Constraints
      if(lots < min_lot) lots = 0;
      if(lots > max_lot) lots = max_lot;
      
      return lots;
   }
   
   bool              ValidateStops(ENUM_ORDER_TYPE type, double price, 
                                   double sl, double tp) {
      int stops_level = (int)m_symbol.StopsLevel();
      
      if(stops_level == 0) return true;
      
      double min_distance = stops_level * m_symbol.Point();
      
      //--- Check SL distance
      if(sl > 0) {
         double sl_distance = (type == ORDER_TYPE_BUY) ? (price - sl) : (sl - price);
         if(sl_distance < min_distance) {
            Print("SL too close to price. Required: ", min_distance, " points");
            return false;
         }
      }
      
      //--- Check TP distance
      if(tp > 0) {
         double tp_distance = (type == ORDER_TYPE_BUY) ? (tp - price) : (price - tp);
         if(tp_distance < min_distance) {
            Print("TP too close to price. Required: ", min_distance, " points");
            return false;
         }
      }
      
      return true;
   }
   
   bool              CheckMargin(ENUM_ORDER_TYPE type, double lots, double price) {
      double margin_required;
      if(!OrderCalcMargin(type, m_symbol.Name(), lots, price, margin_required)) {
         return false;
      }
      
      double free_margin = AccountInfoDouble(ACCOUNT_MARGIN_FREE);
      return free_margin >= margin_required * 1.5;  // 50% buffer
   }
   
   string            GetRetcodeDescription(uint code) {
      switch(code) {
         case TRADE_RETCODE_REQUOTE:       return "Requote";
         case TRADE_RETCODE_REJECT:        return "Rejected";
         case TRADE_RETCODE_CANCEL:        return "Cancelled";
         case TRADE_RETCODE_PLACED:        return "Placed";
         case TRADE_RETCODE_DONE:          return "Done";
         case TRADE_RETCODE_DONE_PARTIAL:  return "Partial";
         case TRADE_RETCODE_ERROR:         return "Error";
         case TRADE_RETCODE_TIMEOUT:       return "Timeout";
         case TRADE_RETCODE_INVALID:       return "Invalid parameters";
         case TRADE_RETCODE_INVALID_VOLUME:return "Invalid volume";
         case TRADE_RETCODE_INVALID_PRICE: return "Invalid price";
         case TRADE_RETCODE_INVALID_STOPS: return "Invalid stops";
         case TRADE_RETCODE_TRADE_DISABLED:return "Trade disabled";
         case TRADE_RETCODE_MARKET_CLOSED: return "Market closed";
         case TRADE_RETCODE_NO_MONEY:      return "No money";
         default:                          return "Unknown: " + IntegerToString(code);
      }
   }
};

---

### 4.2 Legacy Approach: OrderSend (For Advanced Control)

```cpp
bool SendMarketOrder(ENUM_ORDER_TYPE type, double lots, double sl, double tp) {
   MqlTradeRequest request = {};
   MqlTradeResult result = {};
   
   //--- Fill request
   request.action = TRADE_ACTION_DEAL;
   request.symbol = _Symbol;
   request.volume = lots;
   request.type = type;
   request.price = (type == ORDER_TYPE_BUY) ? SymbolInfoDouble(_Symbol, SYMBOL_ASK) 
                                            : SymbolInfoDouble(_Symbol, SYMBOL_BID);
   request.sl = sl;
   request.tp = tp;
   request.deviation = 10;
   request.magic = 123456;
   request.comment = "EA Order";
   request.type_filling = GetFillingMode();  // Detect valid filling mode
   request.type_time = ORDER_TIME_GTC;
   
   //--- Send with retry
   for(int attempt = 0; attempt < 3; attempt++) {
      ResetLastError();
      
      if(!OrderSend(request, result)) {
         int err = GetLastError();
         Print("OrderSend error: ", err);
         
         if(result.retcode == TRADE_RETCODE_REQUOTE) {
            // Update price and retry
            RefreshRates();
            request.price = (type == ORDER_TYPE_BUY) ? SymbolInfoDouble(_Symbol, SYMBOL_ASK) 
                                                     : SymbolInfoDouble(_Symbol, SYMBOL_BID);
            Sleep(100);
            continue;
         }
         else if(result.retcode == TRADE_RETCODE_PRICE_OFF) {
            RefreshRates();
            continue;
         }
         else if(result.retcode == TRADE_RETCODE_MARKET_CLOSED ||
                 result.retcode == TRADE_RETCODE_TRADE_DISABLED ||
                 result.retcode == TRADE_RETCODE_NO_MONEY) {
            break;  // Don't retry these
         }
      } else {
         if(result.retcode == TRADE_RETCODE_DONE || 
            result.retcode == TRADE_RETCODE_DONE_PARTIAL) {
            Print("Order successful. Ticket: ", result.order);
            return true;
         }
      }
      
      Sleep(1000);
   }
   
   return false;
}

ENUM_ORDER_TYPE_FILLING GetFillingMode() {
   uint filling = (uint)SymbolInfoInteger(_Symbol, SYMBOL_FILLING_MODE);
   
   if(filling & ORDER_FILLING_FOK)   return ORDER_FILLING_FOK;
   if(filling & ORDER_FILLING_IOC)   return ORDER_FILLING_IOC;
   if(filling & ORDER_FILLING_RETURN)return ORDER_FILLING_RETURN;
   
   return ORDER_FILLING_FOK;  // Default
}

---

### 4.3 Pending Orders Management

```cpp
bool PlacePendingOrder(ENUM_ORDER_TYPE type, double price, double lots, 
                       double sl, double tp, datetime expiration = 0) {
   //--- Validate price
   double current_price = (type == ORDER_TYPE_BUY_LIMIT || type == ORDER_TYPE_BUY_STOP) ?
                          SymbolInfoDouble(_Symbol, SYMBOL_ASK) :
                          SymbolInfoDouble(_Symbol, SYMBOL_BID);
   
   //--- Check if price is valid for order type
   if(type == ORDER_TYPE_BUY_LIMIT && price >= current_price) {
      Print("Buy Limit must be below current price");
      return false;
   }
   if(type == ORDER_TYPE_SELL_LIMIT && price <= current_price) {
      Print("Sell Limit must be above current price");
      return false;
   }
   
   MqlTradeRequest request = {};
   request.action = TRADE_ACTION_PENDING;
   request.symbol = _Symbol;
   request.type = type;
   request.price = NormalizeDouble(price, _Digits);
   request.volume = lots;
   request.sl = sl;
   request.tp = tp;
   request.deviation = 10;
   request.magic = 123456;
   
   if(expiration > 0) {
      request.type_time = ORDER_TIME_SPECIFIED;
      request.expiration = expiration;
   } else {
      request.type_time = ORDER_TIME_GTC;
   }
   
   MqlTradeResult result = {};
   if(!OrderSend(request, result)) {
      Print("Pending order failed: ", result.retcode);
      return false;
   }
   
   return true;
}

bool DeletePendingOrder(ulong ticket) {
   MqlTradeRequest request = {};
   request.action = TRADE_ACTION_REMOVE;
   request.order = ticket;
   
   MqlTradeResult result = {};
   if(!OrderSend(request, result)) {
      Print("Delete order failed: ", result.retcode);
      return false;
   }
   
   return true;
}

---

### 4.4 Position Tracking (Multi-Position EAs)

```cpp
struct PositionData {
   ulong    ticket;
   string   symbol;
   datetime open_time;
   double   open_price;
   double   volume;
   double   sl;
   double   tp;
   double   profit;
   int      magic;
};

class CPositionTracker {
private:
   PositionData      m_positions[];
   CPositionInfo     m_pos_info;
   
public:
   void              Refresh() {
      ArrayResize(m_positions, 0);
      
      for(int i = PositionsTotal() - 1; i >= 0; i--) {
         if(m_pos_info.SelectByIndex(i)) {
            if(m_pos_info.Magic() != 123456) continue;  // Filter by magic
            
            int idx = ArraySize(m_positions);
            ArrayResize(m_positions, idx + 1);
            
            m_positions[idx].ticket = m_pos_info.Ticket();
            m_positions[idx].symbol = m_pos_info.Symbol();
            m_positions[idx].open_time = m_pos_info.Time();
            m_positions[idx].open_price = m_pos_info.PriceOpen();
            m_positions[idx].volume = m_pos_info.Volume();
            m_positions[idx].sl = m_pos_info.StopLoss();
            m_positions[idx].tp = m_pos_info.TakeProfit();
            m_positions[idx].profit = m_pos_info.Profit();
            m_positions[idx].magic = m_pos_info.Magic();
         }
      }
   }
   
   int               Count() { return ArraySize(m_positions); }
   
   bool              HasPosition(string symbol, ENUM_POSITION_TYPE type) {
      for(int i = 0; i < ArraySize(m_positions); i++) {
         if(m_positions[i].symbol == symbol) {
            if(!m_pos_info.SelectByTicket(m_positions[i].ticket)) continue;
            if(m_pos_info.PositionType() == type) return true;
         }
      }
      return false;
   }
   
   bool              ModifyAllStops(double new_sl, double new_tp) {
      for(int i = 0; i < ArraySize(m_positions); i++) {
         m_trade.PositionModify(m_positions[i].ticket, new_sl, new_tp);
      }
   }
   
   double            TotalProfit() {
      double total = 0;
      for(int i = 0; i < ArraySize(m_positions); i++) {
         total += m_positions[i].profit;
      }
      return total;
   }
};