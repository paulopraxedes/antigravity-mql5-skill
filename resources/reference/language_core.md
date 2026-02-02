## SKILL-001: Language Core & Preprocessor

**Feature ID:** LANG-001  
**Domain:** Core Language  
**Scope:** Syntax, types, preprocessor directives, memory management

---

### 1.1 Preprocessor Directives

**Mandatory Properties:**
```cpp
#property strict                    // CRITICAL: Enable strict compilation
#property copyright "Your Name"     // Required for identification
#property version   "1.00"          // Semantic versioning
#property description "EA Description" // Multi-line supported

// Option A: Traditional (Portable)
#ifndef __MYHEADER_MQH__
#define __MYHEADER_MQH__
// ... code ...
#endif // __MYHEADER_MQH__

// Option B: Modern (Faster compilation)
#pragma once

---

### 1.2 Data Types & Safety

**Integer Types:**

| Type   | Size    | Range       | Use Case                |
| ------ | ------- | ----------- | ----------------------- |
| char   | 1 byte  | -128 to 127 | Small counters, flags   |
| uchar  | 1 byte  | 0 to 255    | Color components, bytes |
| short  | 2 bytes | -32k to 32k | Chart handles           |
| ushort | 2 bytes | 0 to 65k    | Unicode characters      |
| int    | 4 bytes | -2B to 2B   | General purpose         |
| uint   | 4 bytes | 0 to 4B     | Bitwise operations      |
| long   | 8 bytes | Huge        | Volume, timestamps      |

**Critical Type Safety Rules:**

```cpp
// WRONG: Integer division truncates
double avg = high + low / 2;  // low/2 happens first!

// CORRECT: Force floating point
double avg = (high + low) / 2.0;

// WRONG: Float comparison
if(price == stop_loss)  // Never do this!

// CORRECT: Epsilon comparison
if(MathAbs(price - stop_loss) < _Point)  // Use symbol point

---

### 1.3 Memory Management

**Dynamic Arrays:**

```cpp
double buffer[];
ArrayResize(buffer, 100);  // Allocates
// ... use buffer ...
ArrayFree(buffer);         // CRITICAL: Free when done

**Object Lifecycle:**

```cpp
class CMyClass {
public:
   CMyClass() { Print("Constructor"); }
   ~CMyClass() { Print("Destructor"); }
};

// CORRECT: Automatic cleanup
void OnTick() {
   CMyClass obj;  // Stack allocation, auto destruct
}

// DANGEROUS: Manual memory
CMyClass* ptr = NULL;

void OnInit() {
   ptr = new CMyClass();  // Heap allocation
}

void OnDeinit(const int reason) {
   if(ptr != NULL) {
      delete ptr;      // CRITICAL: Must delete
      ptr = NULL;      // Prevent dangling pointer
   }
}

---

### 1.4 String Management

**Efficiency Rules:**

```cpp
// WRONG: Slow in loops (creates temporaries)
string result = "";
for(int i = 0; i < 1000; i++) {
   result += "data";  // O(n²) complexity!
}

// CORRECT: Pre-allocate and use StringAdd
string result;
StringInit(result, 10000);  // Pre-allocate
for(int i = 0; i < 1000; i++) {
   StringAdd(result, "data");  // Efficient append
}

// Formatting with precision
string price_str = DoubleToString(price, _Digits);
string time_str = TimeToString(TimeCurrent(), TIME_DATE|TIME_SECONDS);

---

### 1.5 Common Pitfalls

**Pitfall 1: Undeclared Variables in Strict Mode:**

```cpp
// In #property strict, all variables must be declared
int x = 0;  // Explicit type required
var y = 0;  // ERROR: 'var' not allowed in MQL5

**Pitfall 2: Array Series vs Array Index:**

```cpp
double close[];
ArraySetAsSeries(close, true);   // true = series (0=current bar)
CopyClose(_Symbol, PERIOD_CURRENT, 0, 10, close);
// close[0] = current price, close[1] = previous price

double close2[];
ArraySetAsSeries(close2, false);  // false = linear (0=oldest)
CopyClose(_Symbol, PERIOD_CURRENT, 0, 10, close2);
// close2[0] = 10 bars ago, close2[9] = current price

---

### 1.6 Function Contracts Template

```cpp
//+------------------------------------------------------------------+
//| FUNCTION CONTRACT                                                  |
//+------------------------------------------------------------------+
// Name:          FunctionName
// Domain:        [Trading|Analysis|Utility]
// Scope:         [Public|Private]
// Purpose:       One sentence description
//                                                                    
// INPUTS:                                                            
//   param1   [in]  type   - Description (range constraints)
//   param2   [in]  type   - Description
//                                                                    
// OUTPUTS:                                                           
//   return   type   - Description (error conditions)
//                                                                    
// PRECONDITIONS:                                                     
//   - Condition 1
//   - Condition 2
//                                                                    
// POSTCONDITIONS:                                                    
//   - State change 1
//                                                                    
// SIDE EFFECTS:                                                      
//   - File writes, global modifications, etc.
//                                                                    
// ERROR HANDLING:                                                    
//   - Returns false if...
//   - Logs error when...
//                                                                    
// TEST CASES:                                                        
//   TC1: Input X → Expected Y
//   TC2: Invalid input → Returns false
//+------------------------------------------------------------------+

---

### 1.7 CHANGELOG Template

```cpp
//+------------------------------------------------------------------+
//| CHANGELOG                                                          |
//+------------------------------------------------------------------+
// [YYYY-MM-DD] vX.XX - Author
//   - Added: Feature description
//   - Changed: Modification description  
//   - Fixed: Bug fix description
//   - Removed: Deprecated feature
//   - Security: Security fix description
//
// [2024-01-20] v1.01 - Developer
//   - Fixed: Memory leak in buffer allocation
//   - Added: Input validation for risk percent
//+------------------------------------------------------------------+