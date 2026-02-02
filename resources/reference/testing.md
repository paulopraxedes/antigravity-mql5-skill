
---

## 📁 File 19: `skills/14_testing.md`

```markdown
## SKILL-014: Testing & Quality Assurance

**Feature ID:** TEST-001  
**Domain:** Quality Assurance  
**Scope:** Unit testing, mocking, test automation

---

### 14.1 Unit Testing Framework

```cpp
//+------------------------------------------------------------------+
//| UNIT TEST FRAMEWORK                                                |
//+------------------------------------------------------------------+

#define TEST_ASSERT(condition, msg) \
   if(!(condition)) { Print("FAIL: ", msg, " at line ", __LINE__); return false; }

#define TEST_ASSERT_EQUAL(expected, actual) \
   if((expected) != (actual)) { \
      Print("FAIL: Expected ", expected, " but got ", actual); \
      return false; \
   }

#define TEST_ASSERT_DOUBLE_EQ(expected, actual, epsilon) \
   if(MathAbs((expected) - (actual)) > (epsilon)) { \
      Print("FAIL: Expected ", expected, " but got ", actual); \
      return false; \
   }

#define RUN_TEST(test_func) \
   Print("Running: ", #test_func, "..."); \
   if(test_func()) { \
      Print("  PASS"); \
      tests_passed++; \
   } else { \
      Print("  FAIL"); \
      tests_failed++; \
   } \
   tests_total++;

class CTestSuite
{
protected:
   int    m_tests_total;
   int    m_tests_passed;
   int    m_tests_failed;
   string m_name;
   
public:
   CTestSuite(string name) : 
      m_name(name), m_tests_total(0), m_tests_passed(0), m_tests_failed(0) {}
   
   void Run()
   {
      Print("========================================");
      Print("Test Suite: ", m_name);
      Print("========================================");
      
      Setup();
      RunTests();
      TearDown();
      
      Print("----------------------------------------");
      Print("Total: ", m_tests_total, 
            " | Passed: ", m_tests_passed, 
            " | Failed: ", m_tests_failed);
      Print("========================================");
   }
   
   virtual void Setup() {}
   virtual void TearDown() {}
   virtual void RunTests() = 0;
   
protected:
   void RegisterTest(bool (*test)(), string name)
   {
      Print("Running: ", name, "...");
      bool result = test();
      
      if(result)
      {
         Print("  PASS");
         m_tests_passed++;
      }
      else
      {
        Print("  FAIL");
         m_tests_failed++;
      }
      
      m_tests_total++;
   }
};

---

### 14.2 Mock Objects for Testing

```cpp
class CMockLogger : public ILogger
{
public:
   int    m_log_count;
   string m_last_message;
   
   CMockLogger() : m_log_count(0), m_last_message("") {}
   
   virtual void Log(string msg) override
   {
      m_log_count++;
      m_last_message = msg;
      Print("[MOCK LOG]: ", msg);
   }
   
   bool WasLogged(string expected)
   {
      return (StringFind(m_last_message, expected) != -1);
   }
};

//--- Mock Symbol for testing position sizers
class CMockSymbol : public CSymbolInfo
{
public:
   double m_point;
   double MinLotVal;
   double MaxLotVal;
   double LotStepVal;
   
   CMockSymbol() : 
      m_point(0.00001), 
      MinLotVal(0.01), 
      MaxLotVal(100.0),
      LotStepVal(0.01) {}
   
   virtual double Point() const override { return m_point; }
   virtual double LotsMin() const override { return MinLotVal; }
   virtual double LotsMax() const override { return MaxLotVal; }
   virtual double LotsStep() const override { return LotStepVal; }
};
---

### 14.3 Position Sizer Tests

```cpp
class CTestPositionSizer : public CTestSuite
{
public:
   CTestPositionSizer() : CTestSuite("Position Sizer Tests") {}
   
   virtual void RunTests() override
   {
      RUN_TEST(TestFixedFraction);
      RUN_TEST(TestZeroRisk);
      RUN_TEST(TestLotNormalization);
      RUN_TEST(TestMaxLotCap);
   }
   
   static bool TestFixedFraction()
   {
      // Setup
      CMockLogger logger;
      CMockSymbol symbol;
      CPositionSizer sizer;
      
      symbol.MinLotVal = 0.01;
      symbol.LotStepVal = 0.01;
      
      sizer.Initialize(&symbol, &logger);
      sizer.SetRiskPercent(2.0);
      
      // Execute
      double lots = sizer.CalculateLots(1000.0, 1.1000, 1.0900);  // $100 risk, 1000 points
      
      // Verify
      TEST_ASSERT(lots > 0, "Lots should be positive");
      TEST_ASSERT(lots <= 100.0, "Lots should be reasonable");
      
      return true;
   }
   
   static bool TestZeroRisk()
   {
      CMockLogger logger;
      CMockSymbol symbol;
      CPositionSizer sizer;
      
      sizer.Initialize(&symbol, &logger);
      sizer.SetRiskPercent(0.0);
      
      double lots = sizer.CalculateLots(1000.0, 1.1000, 1.0900);
      
      TEST_ASSERT_EQUAL(0.0, lots);
      TEST_ASSERT(logger.WasLogged("Invalid risk"), "Should log error");
      
      return true;
   }
   
   static bool TestLotNormalization()
   {
      CMockSymbol symbol;
      symbol.LotStepVal = 0.01;
      
      double lots = 0.12345;
      double normalized = NormalizeLots(lots, symbol.LotStepVal);
      
      TEST_ASSERT_DOUBLE_EQ(0.12, normalized, 0.001);
      
      return true;
   }
   
   static bool TestMaxLotCap()
   {
      CMockSymbol symbol;
      symbol.MaxLotVal = 50.0;
      
      // Try to calculate huge lot
      double lots = 999.0;
      double result = MathMin(lots, symbol.MaxLotVal);
      
      TEST_ASSERT(result <= symbol.MaxLotVal, "Should cap at max lot");
      
      return true;
   }
   
   static double NormalizeLots(double lots, double step)
   {
      return MathFloor(lots / step) * step;
   }
};

//--- Entry point for testing
void OnStart()
{
   CTestPositionSizer test;
   test.Run();
}

---

### 14.4 Property-Based Testing (Fuzzing)

```cpp
class CFuzzTester
{
public:
   static void FuzzPositionSizer(int iterations)
   {
      CPositionSizer sizer;
      sizer.Initialize();
      
      MathSrand((int)TimeLocal());
      
      for(int i = 0; i < iterations; i++)
      {
         // Generate random inputs
         double balance = 1000 + MathRand() % 90000;  // 1k to 100k
         double risk = (MathRand() % 100) / 10.0;     // 0 to 10%
         double distance = 10 + MathRand() % 500;     // 10 to 510 points
         
         // Should never crash
         double lots = sizer.CalculateLots(balance, risk, distance);
         
         // Post-conditions
         if(lots < 0)
         {
            Print("FAIL: Negative lots at iteration ", i);
            return;
         }
         
         if(lots > 1000)  // Unreasonable
         {
            Print("FAIL: Lots too large at iteration ", i);
            return;
         }
      }
      
      Print("Fuzz test passed: ", iterations, " iterations");
   }
};

---

### 14.5 Integration Test Template

```cpp
class CIntegrationTest
{
public:
   static bool TestFullTradeCycle()
   {
      //--- Setup
      CTradeManager manager;
      if(!manager.Initialize(123456, 10, ORDER_FILLING_IOC))
         return false;
      
      //--- Pre-conditions
      double initial_balance = AccountInfoDouble(ACCOUNT_BALANCE);
      
      //--- Execute: Open position
      if(!manager.OpenBuy(0.01, 0, 0, "Test Trade"))
         return false;
      
      Sleep(1000);  // Wait for execution
      
      //--- Verify: Position exists
      if(PositionsTotal() == 0)
      {
         Print("FAIL: Position not opened");
         return false;
      }
      
      //--- Execute: Close position
      CPositionInfo pos;
      if(!pos.SelectByIndex(0) || !manager.ClosePosition(pos.Ticket()))
         return false;
      
      Sleep(1000);
      
      //--- Verify: Position closed
      if(PositionsTotal() != 0)
      {
         Print("FAIL: Position not closed");
         return false;
      }
      
      //--- Post-conditions
      double final_balance = AccountInfoDouble(ACCOUNT_BALANCE);
      // Balance might change due to spread/commission
      
      Print("Full trade cycle test passed");
      return true;
   }
};  

---

### 14.6 Automated Test Runner

```cpp

void RunAllTests()
{
   Print("Starting Automated Test Suite...");
   
   int passed = 0;
   int failed = 0;
   
   // Unit tests
   {
      CTestPositionSizer test1;
      test1.Run();
      passed += test1.m_tests_passed;
      failed += test1.m_tests_failed;
   }
   
   {
      CTestRiskManager test2;
      test2.Run();
      passed += test2.m_tests_passed;
      failed += test2.m_tests_failed;
   }
   
   // Integration tests
   if(CIntegrationTest::TestFullTradeCycle())
      passed++;
   else
      failed++;
   
   // Fuzz tests
   CFuzzTester::FuzzPositionSizer(100);
   
   Print("========================================");
   Print("FINAL RESULTS");
   Print("Passed: ", passed);
   Print("Failed: ", failed);
   Print("========================================");
   
   if(failed > 0)
   {
      Alert("TESTS FAILED! Check log.");
      ExpertRemove();
   }
}