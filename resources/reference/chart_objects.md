
---

## 📁 File 16: `skills/11_chart_objects.md`

```markdown
## SKILL-011: Chart Objects & GUI

**Feature ID:** GUI-001  
**Domain:** Visualization  
**Scope:** Objects, buttons, panels, event handling

---

### 11.1 Trading Panel Builder

```cpp
class CTradingPanel
{
private:
   long              m_chart_id;
   string            m_prefix;
   int               m_button_y;
   
public:
   CTradingPanel(string prefix) : m_prefix(prefix), m_button_y(30) 
   {
      m_chart_id = ChartID();
   }
   
   bool CreateButton(string name, int x, int width, string text, color bg_color)
   {
      string obj_name = m_prefix + name;
      
      if(!ObjectCreate(m_chart_id, obj_name, OBJ_EDIT, 0, 0, 0))
         return false;
      
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_XDISTANCE, x);
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_YDISTANCE, m_button_y);
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_XSIZE, width);
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_YSIZE, 25);
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_BGCOLOR, bg_color);
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_COLOR, clrWhite);
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_FONTSIZE, 10);
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_ALIGN, ALIGN_CENTER);
      ObjectSetString(m_chart_id, obj_name, OBJPROP_FONT, "Arial Bold");
      ObjectSetString(m_chart_id, obj_name, OBJPROP_TEXT, text);
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_SELECTABLE, false);
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_READONLY, true);
      
      return true;
   }
   
   bool CreateLabel(string name, int x, int y, string text, color clr = clrWhite)
   {
      string obj_name = m_prefix + name;
      
      if(!ObjectCreate(m_chart_id, obj_name, OBJ_LABEL, 0, 0, 0))
         return false;
      
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_XDISTANCE, x);
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_YDISTANCE, y);
      ObjectSetString(m_chart_id, obj_name, OBJPROP_TEXT, text);
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_COLOR, clr);
      ObjectSetInteger(m_chart_id, obj_name, OBJPROP_FONTSIZE, 9);
      
      return true;
   }
   
   void UpdateLabel(string name, string text, color clr = clrNONE)
   {
      string obj_name = m_prefix + name;
      
      if(ObjectFind(m_chart_id, obj_name) < 0) return;
      
      ObjectSetString(m_chart_id, obj_name, OBJPROP_TEXT, text);
      if(clr != clrNONE)
         ObjectSetInteger(m_chart_id, obj_name, OBJPROP_COLOR, clr);
      
      ChartRedraw(m_chart_id);
   }
   
   void Destroy()
   {
      ObjectsDeleteAll(m_chart_id, m_prefix);
      ChartRedraw(m_chart_id);
   }
   
   string GetClickedObject(string sparam)
   {
      if(StringFind(sparam, m_prefix) == 0)
         return StringSubstr(sparam, StringLen(m_prefix));
      return "";
   }
};

---

### 11.2 Trend Line Auto-Drawing

```cpp
class CAutoTrendLines
{
private:
   string            m_prefix;
   int               m_max_lines;
   
public:
   CAutoTrendLines(string prefix, int max_lines = 10) 
      : m_prefix(prefix), m_max_lines(max_lines) {}
   
   void DrawTrendLine(string name, datetime t1, double p1, datetime t2, double p2, 
                     color clr, bool ray = true)
   {
      string obj_name = m_prefix + name;
      
      if(!ObjectCreate(0, obj_name, OBJ_TREND, 0, t1, p1, t2, p2))
         return;
      
      ObjectSetInteger(0, obj_name, OBJPROP_COLOR, clr);
      ObjectSetInteger(0, obj_name, OBJPROP_WIDTH, 2);
      ObjectSetInteger(0, obj_name, OBJPROP_RAY_RIGHT, ray);
      ObjectSetInteger(0, obj_name, OBJPROP_SELECTABLE, false);
   }
   
   void DrawHorizontalLine(string name, double price, color clr, int style = STYLE_SOLID)
   {
      string obj_name = m_prefix + name;
      
      if(!ObjectCreate(0, obj_name, OBJ_HLINE, 0, 0, price))
         return;
      
      ObjectSetInteger(0, obj_name, OBJPROP_COLOR, clr);
      ObjectSetInteger(0, obj_name, OBJPROP_STYLE, style);
      ObjectSetInteger(0, obj_name, OBJPROP_WIDTH, 1);
   }
   
   void DrawTradeLevels(ulong ticket, double open_price, double sl, double tp, 
                       double current_price)
   {
      string base_name = m_prefix + "Trade_" + IntegerToString(ticket);
      
      // Entry level
      DrawHorizontalLine(base_name + "_Entry", open_price, clrBlue, STYLE_SOLID);
      
      // SL level
      if(sl > 0)
         DrawHorizontalLine(base_name + "_SL", sl, clrRed, STYLE_DASH);
      
      // TP level
      if(tp > 0)
         DrawHorizontalLine(base_name + "_TP", clrGreen, STYLE_DASH);
      
      // Current P&L label
      double pnl = (current_price - open_price) / _Point;
      color pnl_color = (pnl >= 0) ? clrGreen : clrRed;
      
      string label_name = base_name + "_P&L";
      ObjectCreate(0, label_name, OBJ_TEXT, 0, TimeCurrent(), current_price);
      ObjectSetString(0, label_name, OBJPROP_TEXT, 
                     StringFormat("P&L: %.1f pts", pnl));
      ObjectSetInteger(0, label_name, OBJPROP_COLOR, pnl_color);
   }
   
   void CleanupTradeLines(ulong ticket)
   {
      string base_name = m_prefix + "Trade_" + IntegerToString(ticket);
      ObjectsDeleteAll(0, base_name);
   }
};