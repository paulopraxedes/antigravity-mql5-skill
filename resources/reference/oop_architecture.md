
---

## 📁 File 3: `skills/02_oop_architecture.md`

```markdown
## SKILL-002: Object-Oriented Architecture

**Feature ID:** OOP-001  
**Domain:** Software Architecture  
**Scope:** Classes, inheritance, polymorphism, design patterns

---

### 2.1 Class Design Standards

**Base Structure:**
```cpp
class CBaseManager {
private:
   bool              m_initialized;     // State tracking
   string            m_name;            // Instance identification
   CLogger*          m_logger;          // Dependency injection
   
protected:
   // Protected constructor for abstract classes
   CBaseManager(string name) : m_initialized(false), m_name(name), m_logger(NULL) {}
   
public:
   // Virtual destructor for proper cleanup
   virtual          ~CBaseManager() { Deinit(); }
   
   // Non-virtual interface (NVI pattern)
   bool             Init(CLogger* logger) {
      if(m_initialized) return true;
      m_logger = logger;
      if(!OnInit()) return false;
      m_initialized = true;
      return true;
   }
   
   void             Deinit() {
      if(!m_initialized) return;
      OnDeinit();
      m_initialized = false;
   }
   
   bool             IsInitialized() const { return m_initialized; }
   string           GetName() const { return m_name; }
   
protected:
   // Virtual hooks for derived classes
   virtual bool     OnInit() { return true; }
   virtual void     OnDeinit() {}
};

// Usage
class CTradeManager : public CBaseManager {
public:
   CTradeManager() : CBaseManager("TradeManager") {}
   
protected:
   virtual bool OnInit() override {
      // Specific initialization
      return true;
   }
};

---

### 2.2 Singleton Pattern (Trade Manager)

**Thread-Safe Singleton:**
```cpp
class CTradeManager {
private:
   static CTradeManager* m_instance;
   CTrade*               m_trade;
   
   // Private constructor
   CTradeManager() {
      m_trade = new CTrade();
      m_trade.SetExpertMagicNumber(123456);
      m_trade.SetDeviationInPoints(10);
      m_trade.SetTypeFilling(ORDER_FILLING_IOC);
   }
   
   // Prevent copying
   CTradeManager(const CTradeManager&) = delete;
   CTradeManager& operator=(const CTradeManager&) = delete;
   
public:
   static CTradeManager* Instance() {
      if(m_instance == NULL) {
         m_instance = new CTradeManager();
      }
      return m_instance;
   }
   
   static void Destroy() {
      if(m_instance != NULL) {
         delete m_instance;
         m_instance = NULL;
      }
   }
   
   CTrade* GetTrade() { return m_trade; }
};

// Static member definition
CTradeManager* CTradeManager::m_instance = NULL;

// Usage
void OnTick() {
   CTradeManager::Instance()->GetTrade().PositionOpen(...);
}

void OnDeinit(const int reason) {
   CTradeManager::Destroy();  // Critical cleanup
}

---

### 2.3 Observer Pattern (Signal Aggregation)

```cpp
// Abstract observer
class ISignalObserver {
public:
   virtual void OnSignal(int signal_type, double strength, string symbol) = 0;
   virtual ~ISignalObserver() {}
};

// Concrete observer
class CTradeExecutor : public ISignalObserver {
public:
   virtual void OnSignal(int signal_type, double strength, string symbol) override {
      if(strength > 0.8 && signal_type == SIGNAL_BUY) {
         OpenBuy(symbol);
      }
   }
   
private:
   void OpenBuy(string symbol) {
      // Execution logic
   }
};

// Subject (Signal Generator)
class CSignalAggregator {
private:
   ISignalObserver* m_observers[];
   
public:
   void Attach(ISignalObserver* observer) {
      int size = ArraySize(m_observers);
      ArrayResize(m_observers, size + 1);
      m_observers[size] = observer;
   }
   
   void Notify(int signal, double strength, string symbol) {
      for(int i = 0; i < ArraySize(m_observers); i++) {
         m_observers[i].OnSignal(signal, strength, symbol);
      }
   }
};  

---

### 2.4 Dependency Injection

**WRONG: Hardcoded dependencies:**
```cpp
class CRiskManager {
private:
   CLogger m_logger;  // Concrete class, hard to test
   
public:
   CRiskManager() {
      m_logger = new CLogger();  // Tight coupling
   }
};

**CORRECT: Injected dependencies:**
```cpp
class CRiskManager {
private:
   ILogger* m_logger;  // Interface/abstract
   CSymbolInfo* m_symbol;
   
public:
   // Constructor injection
   CRiskManager(ILogger* logger, CSymbolInfo* symbol) 
      : m_logger(logger), m_symbol(symbol) {
      if(logger == NULL || symbol == NULL) {
         Print("CRiskManager: NULL dependencies!");
      }
   }
   
   // Setter injection (optional dependencies)
   void SetLogger(ILogger* logger) { m_logger = logger; }
};


---

### 2.5 Interface Segregation

```cpp
// Separate concerns into small interfaces
class IPriceProvider {
public:
   virtual double GetBid() = 0;
   virtual double GetAsk() = 0;
   virtual double GetPoint() = 0;
};

class ITradeExecutor {
public:
   virtual bool Open Buy(double lots, double sl, double tp) = 0;
   virtual bool OpenSell(double lots, double sl, double tp) = 0;
   virtual bool ClosePosition(ulong ticket) = 0;
};

class IRiskManager {
public:
   virtual double CalculateLots(double risk_percent, double stop_distance) = 0;
   virtual bool ValidateTrade(double lots, double price) = 0;
};

// Concrete implementation combines interfaces
class CTradingFacade : public IPriceProvider, public ITradeExecutor, public IRiskManager {
   // Implementation...
};

---

### 2.6 RAII (Resource Acquisition Is Initialization)

```cpp
// Auto-releasing indicator handle
class CIndicatorHandle {
private:
   int m_handle;
   
public:
   CIndicatorHandle(int handle) : m_handle(handle) {}
   ~CIndicatorHandle() { 
      if(m_handle != INVALID_HANDLE) {
         IndicatorRelease(m_handle);
      }
   }
   
   int Get() const { return m_handle; }
   bool IsValid() const { return m_handle != INVALID_HANDLE; }
   
   // Prevent copying (handle ownership)
   CIndicatorHandle(const CIndicatorHandle&) = delete;
   CIndicatorHandle& operator=(const CIndicatorHandle&) = delete;
};

// Usage
void OnTick() {
   CIndicatorHandle ma(iMA(_Symbol, 0, 14, 0, MODE_SMA, PRICE_CLOSE));
   
   if(!ma.IsValid()) return;
   
   double values[];
   CopyBuffer(ma.Get(), 0, 0, 10, values);
   // Handle automatically released when scope ends
}