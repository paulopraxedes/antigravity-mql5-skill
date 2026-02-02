
---

## 📁 File 4: `skills/03_event_driven.md`

```markdown
## SKILL-003: Event-Driven Programming

**Feature ID:** EVT-001  
**Domain:** Execution Flow  
**Scope:** Event handlers, tick filtering, timers, trade events

---

### 3.1 Core Event Handlers

**Initialization & Cleanup:**
```cpp
int OnInit() {
   //--- Validation
   if(!CheckTerminalSettings()) {
      Alert("Terminal settings incorrect");
      return INIT_FAILED;
   }
   
   //--- Initialize indicators
   m_maHandle = iMA(_Symbol, PERIOD_CURRENT, 14, 0, MODE_SMA, PRICE_CLOSE);
   if(m_maHandle == INVALID_HANDLE) {
      Print("Failed to create MA handle, error: ", GetLastError());
      return INIT_FAILED;
   }
   
   //--- Initialize timer (optional)
   EventSetTimer(60);  // Every 60 seconds
   
   Print(__FUNCTION__, " initialization succeeded");
   return INIT_SUCCEEDED;
}

void OnDeinit(const int reason) {
   //--- Release resources
   IndicatorRelease(m_maHandle);
   EventKillTimer();  // CRITICAL: Kill timer
   
   //--- Log deinitialization reason
   string reason_str = "";
   switch(reason) {
      case REASON_REMOVE:        reason_str = "Removed from chart"; break;
      case REASON_CHARTCLOSE:    reason_str = "Chart closed"; break;
      case REASON_PARAMETERS:    reason_str = "Parameters changed"; break;
      case REASON_TEMPLATE:      reason_str = "Template applied"; break;
      case REASON_REMOVE:        reason_str = "Program terminated"; break;
      default:                   reason_str = "Unknown"; break;
   }
   Print(__FUNCTION__, " deinitialized. Reason: ", reason_str);
}

---

### 3.2 Tick Event Optimization

CRITICAL: New Bar Detection

```cpp
datetime m_prevTime = 0;

bool IsNewBar() {
   datetime currTime = iTime(_Symbol, PERIOD_CURRENT, 0);
   
   if(currTime == m_prevTime) return false;
   
   m_prevTime = currTime;
   return true;
}

void OnTick() {
   //--- Filter: Only process on new bar
   if(!IsNewBar()) return;
   
   //--- Refresh symbol data
   if(!m_symbol.Refresh()) {
      Print("Failed to refresh symbol data");
      return;
   }
   
   //--- Check trading conditions
   if(!IsTradeAllowed()) return;
   
   //--- Execute strategy
   ProcessTradingLogic();
}

bool IsTradeAllowed() {
   //--- Terminal checks
   if(!IsTradeAllowed()) return false;  // Trading disabled
   if(!IsConnected()) return false;      // No connection
   if(IsStopped()) return false;         // Program stop requested
   
   //--- Symbol checks
   if(!m_symbol.IsTradable()) return false;
   if(m_symbol.Spread() > m_maxSpread * m_symbol.Point()) {
      Print("Spread too high: ", m_symbol.Spread());
      return false;
   }
   
   //--- Account checks
   if(AccountInfoDouble(ACCOUNT_MARGIN_FREE) < m_minMargin) return false;
   
   return true;
}

---

### 3.3 Timer-Based Execution

```cpp
void OnTimer() {
   //--- Hourly tasks
   static int prevHour = -1;
   datetime now = TimeCurrent();
   MqlDateTime dt;
   TimeToStruct(now, dt);
   
   if(dt.hour != prevHour) {
      prevHour = dt.hour;
      
      //--- Hourly maintenance
      SaveStatistics();
      CheckConnectionHealth();
   }
   
   //--- Check for end of trading day
   if(dt.hour == 23 && dt.min >= 55) {
      CloseAllPositions("End of day");
   }
}

---

### 3.4 Trade Transaction Events (Advanced)

```cpp
void OnTradeTransaction(const MqlTradeTransaction& trans,
                       const MqlTradeRequest& request,
                       const MqlTradeResult& result) {
   
   //--- Transaction type analysis
   switch(trans.type) {
      case TRADE_TRANSACTION_ORDER_ADD:
         // New order placed (pending)
         HandleOrderAdded(trans.order);
         break;
         
      case TRADE_TRANSACTION_ORDER_UPDATE:
         // Order modified (SL/TP change)
         HandleOrderUpdated(trans.order);
         break;
         
      case TRADE_TRANSACTION_ORDER_DELETE:
         // Order deleted or filled
         HandleOrderDeleted(trans.order);
         break;
         
      case TRADE_TRANSACTION_HISTORY_ADD:
         // Deal executed (position opened/closed)
         HandleDealExecuted(result.deal);
         SaveTradeToHistory(trans, result);
         break;
         
      case TRADE_TRANSACTION_POSITION:
         // Position volume/price changed
         HandlePositionModified(trans.position);
         break;
   }
}

void SaveTradeToHistory(const MqlTradeTransaction& trans, 
                       const MqlTradeResult& result) {
   //--- Log to file or database
   string log_entry = StringFormat("%s|%d|%s|%s|%f|%f|%s",
      TimeToString(TimeCurrent()),
      result.order,
      trans.symbol,
      EnumToString((ENUM_ORDER_TYPE)trans.order_type),
      result.volume,
      result.price,
      result.comment
   );
   
   //--- Write to CSV
   int handle = FileOpen("trade_history.csv", FILE_WRITE|FILE_CSV|FILE_COMMON);
   if(handle != INVALID_HANDLE) {
      FileWrite(handle, TimeCurrent(), result.order, trans.symbol, 
                result.volume, result.price);
      FileClose(handle);
   }
}

---

### 3.5 Book Event (Market Depth)

```cpp
void OnBookEvent(const string &symbol) {
   MqlBookInfo book[];
   
   if(MarketBookGet(symbol, book)) {
      //--- Analyze DOM (Depth of Market)
      double bid_volume = 0;
      double ask_volume = 0;
      
      for(int i = 0; i < ArraySize(book); i++) {
         if(book[i].type == BOOK_TYPE_SELL) {
            ask_volume += book[i].volume;
         } else {
            bid_volume += book[i].volume;
         }
      }
      
      //--- Detect imbalance
      if(ask_volume > bid_volume * 2.0) {
         // Heavy selling pressure
         m_signal = SIGNAL_SELL_PRESSURE;
      } else if(bid_volume > ask_volume * 2.0) {
         // Heavy buying pressure  
         m_signal = SIGNAL_BUY_PRESSURE;
      }
   }
}

---

### 3.6 Chart Events (User Interaction)

```cpp
void OnChartEvent(const int id, const long &lparam, 
                 const double &dparam, const string &sparam) {
   
   switch(id) {
      case CHARTEVENT_KEYDOWN:
         HandleKeyPress((int)lparam);
         break;
         
      case CHARTEVENT_MOUSE_MOVE:
         // lparam = x coordinate, dparam = y coordinate
         HandleMouseMove((int)lparam, (int)dparam);
         break;
         
      case CHARTEVENT_OBJECT_CLICK:
         // sparam = object name
         HandleObjectClick(sparam);
         break;
         
      case CHARTEVENT_OBJECT_DRAG:
         HandleObjectMoved(sparam);
         break;
         
      case CHARTEVENT_CLICK:
         // Chart click at lparam (x), dparam (y)
         HandleChartClick((int)lparam, (int)dparam);
         break;
         
      case CHARTEVENT_CHANGE:
         // Symbol or period changed
         HandleChartChange();
         break;
   }
}

void HandleKeyPress(int key) {
   switch(key) {
      case KEY_B:
         // Manual buy
         OpenManualOrder(ORDER_TYPE_BUY);
         break;
      case KEY_S:
         // Manual sell
         OpenManualOrder(ORDER_TYPE_SELL);
         break;
      case KEY_DELETE:
         CloseAllPositions();
         break;
   }
}