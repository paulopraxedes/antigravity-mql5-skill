## SKILL-005: Custom Indicator Development

**Feature ID:** IND-001  
**Domain:** Technical Analysis  
**Scope:** Custom indicators, buffers, plotting, subwindows

---

### 5.1 Indicator Structure Template

```cpp
#property indicator_separate_window    // or indicator_chart_window
#property indicator_buffers 3
#property indicator_plots   3

//--- Plot 1: Main line
#property indicator_type1   DRAW_LINE
#property indicator_color1  clrRed
#property indicator_style1  STYLE_SOLID
#property indicator_width1  2
#property indicator_label1  "MA Fast"

//--- Plot 2: Signal line
#property indicator_type2   DRAW_LINE
#property indicator_color2  clrBlue
#property indicator_width2  1
#property indicator_label2  "MA Slow"

//--- Plot 3: Histogram
#property indicator_type3   DRAW_HISTOGRAM
#property indicator_color3  clrGreen
#property indicator_label3  "Difference"

//--- Input parameters
input int      InpFastPeriod = 10;
input int      InpSlowPeriod = 20;
input ENUM_MA_METHOD InpMethod = MODE_SMA;

//--- Indicator buffers
double BufferMA1[];
double BufferMA2[];
double BufferHist[];

//--- Indicator handles
int HandleMA1;
int HandleMA2;

//+------------------------------------------------------------------+
//| Custom indicator initialization function                         |
//+------------------------------------------------------------------+
int OnInit()
{
//--- Set index buffers
   SetIndexBuffer(0, BufferMA1, INDICATOR_DATA);
   SetIndexBuffer(1, BufferMA2, INDICATOR_DATA);
   SetIndexBuffer(2, BufferHist, INDICATOR_DATA);
   
//--- Set plotting properties dynamically
   PlotIndexSetInteger(0, PLOT_ARROW, 159);
   PlotIndexSetInteger(0, PLOT_ARROW_SHIFT, -5);
   
//--- Create indicator handles
   HandleMA1 = iMA(NULL, 0, InpFastPeriod, 0, InpMethod, PRICE_CLOSE);
   HandleMA2 = iMA(NULL, 0, InpSlowPeriod, 0, InpMethod, PRICE_CLOSE);
   
   if(HandleMA1 == INVALID_HANDLE || HandleMA2 == INVALID_HANDLE)
     {
      Print("Error creating MA handles");
      return INIT_FAILED;
     }
   
//--- Set indicator short name
   IndicatorSetString(INDICATOR_SHORTNAME, "MA Crossover(" + 
                     IntegerToString(InpFastPeriod) + "," + 
                     IntegerToString(InpSlowPeriod) + ")");
   
   return INIT_SUCCEEDED;
}

//+------------------------------------------------------------------+
//| Custom indicator deinitialization function                       |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
{
//--- CRITICAL: Release indicator handles
   IndicatorRelease(HandleMA1);
   IndicatorRelease(HandleMA2);
}

//+------------------------------------------------------------------+
//| Custom indicator iteration function                              |
//+------------------------------------------------------------------+
int OnCalculate(const int rates_total,
                const int prev_calculated,
                const datetime &time[],
                const double &open[],
                const double &high[],
                const double &low[],
                const double &close[],
                const long &tick_volume[],
                const long &volume[],
                const int &spread[])
{
//--- Start index for calculation
   int start;
   if(prev_calculated == 0)
      start = 0;  // First run, calculate all
   else
      start = prev_calculated - 1;  // Continue from last calculated bar
   
//--- Get MA data
   if(CopyBuffer(HandleMA1, 0, 0, rates_total, BufferMA1) &lt;= 0)
      return 0;
   if(CopyBuffer(HandleMA2, 0, 0, rates_total, BufferMA2) &lt;= 0)
      return 0;
   
//--- Calculate histogram
   for(int i = start; i &lt; rates_total; i++)
     {
      BufferHist[i] = BufferMA1[i] - BufferMA2[i];
     }
   
//--- Return value for next call
   return rates_total;
}

---

### 5.2 Drawing Styles Reference

```
| Style                  | Description          | Use Case                  |
| ---------------------- | -------------------- | ------------------------- |
| `DRAW_LINE`            | Simple line          | Trends, MAs               |
| `DRAW_SECTION`         | Line segments        | Support/resistance zones  |
| `DRAW_HISTOGRAM`       | Bars from zero       | MACD, Volume, Momentum    |
| `DRAW_ARROW`           | Symbols (arrows)     | Buy/sell signals          |
| `DRAW_ZIGZAG`          | Connecting extremums | ZigZag patterns           |
| `DRAW_BARS`            | OHLC bars            | Mini chart                |
| `DRAW_CANDLES`         | Candlesticks         | Mini chart                |
| `DRAW_COLOR_LINE`      | Multi-color line     | Dynamic trend coloring    |
| `DRAW_COLOR_HISTOGRAM` | Multi-color bars     | Strength indication       |
| `DRAW_NONE`            | Invisible buffer     | Intermediate calculations |

---

### 5.3 Multi-Color Dynamic Indicator

```cpp
#property indicator_color1 clrRed, clrGreen, clrBlue  // Up to 64 colors

double BufferLine[];
double BufferColors[];  // Color index buffer (0, 1, 2...)

int OnInit()
{
   SetIndexBuffer(0, BufferLine, INDICATOR_DATA);
   SetIndexBuffer(1, BufferColors, INDICATOR_COLOR_INDEX);
   
   // Set color for each index
   PlotIndexSetInteger(0, PLOT_COLOR_INDEXES, 3);  // 3 colors
   PlotIndexSetInteger(0, PLOT_COLOR_INDEX, 0, clrRed);    // Index 0
   PlotIndexSetInteger(0, PLOT_COLOR_INDEX, 1, clrGreen);  // Index 1
   PlotIndexSetInteger(0, PLOT_COLOR_INDEX, 2, clrBlue);   // Index 2
   
   return INIT_SUCCEEDED;
}

int OnCalculate(...)
{
   for(int i = start; i < rates_total; i++)
     {
      BufferLine[i] = iClose(NULL, 0, i);
      
      // Dynamic coloring based on slope
      if(i > 0)
        {
         if(BufferLine[i] > BufferLine[i-1] * 1.001)
            BufferColors[i] = 1;  // Green - Up
         else if(BufferLine[i] < BufferLine[i-1] * 0.999)
            BufferColors[i] = 0;  // Red - Down
         else
            BufferColors[i] = 2;  // Blue - Flat
        }
     }
   return rates_total;
}

---

### 5.4 Data Window and Tooltip

```cpp
int OnInit()
{
//--- Set data window labels
   PlotIndexSetString(0, PLOT_LABEL, "MA(10)");
   PlotIndexSetString(1, PLOT_LABEL, "MA(20)");
   PlotIndexSetString(2, PLOT_LABEL, "Diff");
   
//--- Set indicator levels (horizontal lines)
   IndicatorSetDouble(INDICATOR_LEVELVALUE, 0, 0.0);
   IndicatorSetString(INDICATOR_LEVELTEXT, 0, "Zero Line");
   IndicatorSetInteger(INDICATOR_LEVELCOLOR, 0, clrGray);
   IndicatorSetInteger(INDICATOR_LEVELSTYLE, 0, STYLE_DOT);
   
//--- Set digits precision
   IndicatorSetInteger(INDICATOR_DIGITS, _Digits);
   
   return INIT_SUCCEEDED;
}

---

### 5.5 Subwindow Minimum/Maximum

```cpp
int OnCalculate(...)
{
//--- Dynamic min/max based on visible data
   if(prev_calculated == 0)
     {
      double min_val = BufferHist[ArrayMinimum(BufferHist, 0, rates_total)];
      double max_val = BufferHist[ArrayMaximum(BufferHist, 0, rates_total)];
      
      IndicatorSetDouble(INDICATOR_MINIMUM, min_val * 1.1);
      IndicatorSetDouble(INDICATOR_MAXIMUM, max_val * 1.1);
     }
   
   return rates_total;
}

---

### 5.6 Indicator Buffer Management

```
CRITICAL RULES:
 - Buffer Indexing: Always use ArraySetAsSeries(buffer, true) for indicator buffers
 - Empty Values: Set PlotIndexSetDouble(index, PLOT_EMPTY_VALUE, EMPTY_VALUE) for gaps
 - Buffer Sizing: Never resize indicator buffers manually - handled by terminal
 - CopyBuffer Return: Always check if CopyBuffer returned expected count

 ```cpp
// Correct buffer handling
int OnCalculate(...)
{
   int start = (prev_calculated == 0) ? 0 : prev_calculated - 1;
   
   for(int i = start; i < rates_total; i++)
     {
      // Buffer is series (0 = current), but OnCalculate gives arrays as series too
      // So both align perfectly
      BufferMain[i] = CalculateValue(close[i], i);
      
      // Mark as empty for gaps
      if(BufferMain[i] == 0)
         BufferMain[i] = EMPTY_VALUE;
     }
   
   return rates_total;
}

---

### 5.7 Custom Indicator as Data Source for EA

```cpp
// In EA: Read custom indicator using iCustom
int handle = iCustom(_Symbol, PERIOD_CURRENT, "MyIndicator", 10, 20);

if(handle == INVALID_HANDLE)
{
   Print("Failed to load custom indicator");
   return INIT_FAILED;
}

// In OnTick(), read buffer values
double value[];
if(CopyBuffer(handle, 0, 0, 1, value) > 0)
{
   if(value[0] > 0) // Signal detected
   {
      // Execute trade
   }
}