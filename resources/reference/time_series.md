
---

## 📁 File 13: `skills/08_time_series.md`

```markdown
## SKILL-008: Time Series & Data Access

**Feature ID:** DATA-001  
**Domain:** Market Data  
**Scope:** CopyRates, CopyTicks, array management, caching

---

### 8.1 Efficient CopyRates with Caching

```cpp
class CPriceCache
{
private:
   struct RateCache
   {
      string            symbol;
      ENUM_TIMEFRAMES   period;
      MqlRates          rates[];
      datetime          last_update;
      bool              as_series;
   };
   
   RateCache         m_caches[];
   int               m_cache_size;
   
public:
   CPriceCache() : m_cache_size(1000) {}
   
   bool GetRates(string symbol, ENUM_TIMEFRAMES period, int count, MqlRates &rates[])
   {
      //--- Find existing cache
      int idx = FindCache(symbol, period);
      
      if(idx == -1)
      {
         // Create new cache
         idx = CreateCache(symbol, period);
         if(idx == -1) return false;
      }
      
      //--- Check if refresh needed (new bar)
      datetime current_time = iTime(symbol, period, 0);
      bool needs_update = (current_time != m_caches[idx].last_update) ||
                          (ArraySize(m_caches[idx].rates) < count);
      
      if(needs_update)
      {
         ArraySetAsSeries(m_caches[idx].rates, true);
         
         int copied = CopyRates(symbol, period, 0, count, m_caches[idx].rates);
         if(copied < count) return false;
         
         m_caches[idx].last_update = current_time;
      }
      
      //--- Copy to output (shallow copy optimization)
      ArrayResize(rates, count);
      ArrayCopy(rates, m_caches[idx].rates, 0, 0, count);
      ArraySetAsSeries(rates, true);
      
      return true;
   }
   
   bool GetMAValue(int ma_handle, int index, double &value)
   {
      double buffer[];
      ArraySetAsSeries(buffer, true);
      
      if(CopyBuffer(ma_handle, 0, index, 1, buffer) <= 0)
         return false;
         
      value = buffer[0];
      return true;
   }
   
private:
   int FindCache(string symbol, ENUM_TIMEFRAMES period)
   {
      for(int i = 0; i < ArraySize(m_caches); i++)
      {
         if(m_caches[i].symbol == symbol && m_caches[i].period == period)
            return i;
      }
      return -1;
   }
   
   int CreateCache(string symbol, ENUM_TIMEFRAMES period)
   {
      int idx = ArraySize(m_caches);
      ArrayResize(m_caches, idx + 1);
      
      m_caches[idx].symbol = symbol;
      m_caches[idx].period = period;
      m_caches[idx].last_update = 0;
      m_caches[idx].as_series = true;
      
      return idx;
   }
};

---

### 8.2 Tick Data Streaming

```cpp
class CTickProcessor
{
private:
   MqlTick           m_last_tick;
   bool              m_initialized;
   
public:
   bool Update()
   {
      MqlTick tick;
      if(!SymbolInfoTick(_Symbol, tick)) return false;
      
      if(!m_initialized)
      {
         m_last_tick = tick;
         m_initialized = true;
         return true;
      }
      
      //--- Detect tick changes
      if(tick.time != m_last_tick.time || 
         tick.bid != m_last_tick.bid || 
         tick.ask != m_last_tick.ask)
      {
         ProcessTick(tick);
         m_last_tick = tick;
      }
      
      return true;
   }
   
   bool IsNewQuote() const
   {
      return (m_last_tick.flags & TICK_FLAG_BID) || 
             (m_last_tick.flags & TICK_FLAG_ASK);
   }
   
   bool IsNewTrade() const
   {
      return (m_last_tick.flags & TICK_FLAG_LAST);
   }
   
   double GetSpread() const
   {
      return (m_last_tick.ask - m_last_tick.bid) / _Point;
   }
   
private:
   void ProcessTick(const MqlTick &tick)
   {
      // Volume analysis
      if(tick.flags & TICK_FLAG_VOLUME)
      {
         // New volume data available
      }
   }
};

---

### 8.3 Multi-Timeframe Synchronization

```cpp
class CMultiTimeframe
{
public:
   bool GetSyncedData(string symbol, ENUM_TIMEFRAMES periods[], 
                     int count, MqlRates &results[][])
   {
      datetime anchor = iTime(symbol, periods[0], 0);
      
      for(int i = 0; i < ArraySize(periods); i++)
      {
         MqlRates rates[];
         ArraySetAsSeries(rates, true);
         
         int copied = CopyRates(symbol, periods[i], 0, count, rates);
         if(copied < count) return false;
         
         //--- Verify alignment
         if(rates[0].time != anchor)
         {
            // Different bar times - find aligned bar
            int aligned_idx = FindAlignedBar(rates, anchor);
            if(aligned_idx == -1) return false;
            
            // Copy aligned segment
            ArrayResize(rates, count, aligned_idx);
         }
         
         //--- Copy to results matrix
         ArrayCopy(results[i], rates);
      }
      
      return true;
   }
   
   bool IsNewBarAcrossTimeframes(string symbol, ENUM_TIMEFRAMES periods[])
   {
      static datetime last_times[];
      
      if(ArraySize(last_times) != ArraySize(periods))
         ArrayResize(last_times, ArraySize(periods));
      
      for(int i = 0; i < ArraySize(periods); i++)
      {
         datetime current = iTime(symbol, periods[i], 0);
         
         if(current != last_times[i])
         {
            last_times[i] = current;
            return true;
         }
      }
      
      return false;
   }
   
private:
   int FindAlignedBar(MqlRates &rates[], datetime target)
   {
      for(int i = 0; i < ArraySize(rates); i++)
      {
         if(rates[i].time == target) return i;
      }
      return -1;
   }
};

---

### 8.4 Array Series vs Array Linear

```cpp
class CArrayOrientation
{
public:
   //--- Convert series (reverse) to linear (chronological)
   static void SeriesToLinear(double &series_array[], double &linear_array[])
   {
      int size = ArraySize(series_array);
      ArrayResize(linear_array, size);
      
      for(int i = 0; i < size; i++)
      {
         linear_array[i] = series_array[size - 1 - i];
      }
   }
   
   //--- Access by day offset (0 = today, 1 = yesterday)
   static double GetValueByDay(double &rates_high[], datetime &rates_time[], 
                              int days_ago)
   {
      datetime target_date = TimeCurrent() - days_ago * 86400;
      
      for(int i = 0; i < ArraySize(rates_time); i++)
      {
         if(TimeToString(rates_time[i], TIME_DATE) == 
            TimeToString(target_date, TIME_DATE))
         {
            return rates_high[i];
         }
      }
      
      return EMPTY_VALUE;
   }
};