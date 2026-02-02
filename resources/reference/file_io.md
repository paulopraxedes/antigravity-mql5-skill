
---

## 📁 File 14: `skills/09_file_io.md`

```markdown
## SKILL-009: File I/O & Data Persistence

**Feature ID:** FILE-001  
**Domain:** Data Storage  
**Scope:** CSV, Binary, JSON, Serialization

---

### 9.1 CSV Trade Logger

```cpp
class CCSVTradeLogger
{
private:
   int               m_file_handle;
   string            m_filename;
   string            m_delimiter;
   bool              m_initialized;
   
public:
   bool Initialize(string filename, bool common_folder = true, 
                  string delimiter = ",")
   {
      m_delimiter = delimiter;
      m_filename = filename;
      
      int flags = FILE_WRITE|FILE_CSV|FILE_SHARE_WRITE;
      if(common_folder) flags |= FILE_COMMON;
      
      //--- Check if exists to write header
      bool exists = FileIsExist(filename, (common_folder ? FILE_COMMON : 0));
      
      m_file_handle = FileOpen(filename, flags, delimiter);
      if(m_file_handle == INVALID_HANDLE) return false;
      
      //--- Write header if new file
      if(!exists)
      {
         FileWrite(m_file_handle, "Ticket", "Time", "Symbol", "Type", 
                   "Volume", "Price", "SL", "TP", "Profit", "Comment");
         FileFlush(m_file_handle);
      }
      
      m_initialized = true;
      return true;
   }
   
   void LogTrade(const MqlTradeResult &result, const MqlTradeRequest &request)
   {
      if(!m_initialized) return;
      
      FileWrite(m_file_handle,
                result.order,
                TimeToString(TimeCurrent(), TIME_DATE|TIME_MINUTES),
                request.symbol,
                EnumToString(request.type),
                result.volume,
                result.price,
                request.sl,
                request.tp,
                0,  // Profit unknown at opening
                request.comment
               );
      
      FileFlush(m_file_handle);  // Ensure written to disk
   }
   
   void LogClose(ulong ticket, double profit)
   {
      if(!m_initialized) return;
      
      FileWrite(m_file_handle,
                ticket,
                TimeToString(TimeCurrent()),
                "", "", 0, 0, 0, 0, profit, "CLOSE"
               );
      FileFlush(m_file_handle);
   }
   
   void Close()
   {
      if(m_file_handle != INVALID_HANDLE)
      {
         FileClose(m_file_handle);
         m_file_handle = INVALID_HANDLE;
         m_initialized = false;
      }
   }
};

---

### 9.2 Binary State Persistence

```cpp
struct RobotState
{
   double            current_equity;
   double            peak_equity;
   int               trade_count;
   bool              emergency_stop;
   datetime          last_trade_time;
   char              reserved[64];  // Padding for future
};

class CStateManager
{
private:
   string            m_filename;
   RobotState        m_state;
   
public:
   CStateManager(string filename) : m_filename(filename)
   {
      ZeroMemory(m_state);
   }
   
   void SaveState()
   {
      int handle = FileOpen(m_filename, FILE_WRITE|FILE_BIN|FILE_COMMON);
      if(handle != INVALID_HANDLE)
      {
         FileWriteStruct(handle, m_state);
         FileClose(handle);
      }
   }
   
   bool LoadState()
   {
      if(!FileIsExist(m_filename, FILE_COMMON)) return false;
      
      int handle = FileOpen(m_filename, FILE_READ|FILE_BIN|FILE_COMMON);
      if(handle == INVALID_HANDLE) return false;
      
      bool success = FileReadStruct(handle, m_state);
      FileClose(handle);
      
      return success;
   }
   
   void UpdateEquity(double equity)
   {
      m_state.current_equity = equity;
      if(equity > m_state.peak_equity)
         m_state.peak_equity = equity;
   }
   
   RobotState* GetState() { return &m_state; }
};

---

### 9.3 JSON Configuration Loader

```cpp
class CConfigManager
{
private:
   string            m_config_file;
   
public:
   bool LoadConfig(string &keys[], string &values[])
   {
      int handle = FileOpen(m_config_file, FILE_READ|FILE_TXT|FILE_COMMON);
      if(handle == INVALID_HANDLE) return false;
      
      string json = FileReadString(handle);
      FileClose(handle);
      
      // Simple key-value parsing (not full JSON)
      // Assumes format: "key": "value"
      int pos = 0;
      while((pos = StringFind(json, "\"", pos)) != -1)
      {
         int key_start = pos + 1;
         int key_end = StringFind(json, "\"", key_start);
         if(key_end == -1) break;
         
         string key = StringSubstr(json, key_start, key_end - key_start);
         
         int val_start = StringFind(json, "\"", key_end + 1);
         if(val_start == -1) break;
         val_start++;
         
         int val_end = StringFind(json, "\"", val_start);
         if(val_end == -1) break;
         
         string value = StringSubstr(json, val_start, val_end - val_start);
         
         int idx = ArraySize(keys);
         ArrayResize(keys, idx + 1);
         ArrayResize(values, idx + 1);
         keys[idx] = key;
         values[idx] = value;
         
         pos = val_end + 1;
      }
      
      return ArraySize(keys) > 0;
   }
   
   string GetValue(string &keys[], string &values[], string key_to_find)
   {
      for(int i = 0; i < ArraySize(keys); i++)
      {
         if(keys[i] == key_to_find)
            return values[i];
      }
      return "";
   }
};

---

9.4 Folder Management

```cpp
class CFolderManager
{
public:
   static bool CreateFolderStructure(string base_name)
   {
      string folders[] = {"Logs", "Data", "Backups", "Reports"};
      
      for(int i = 0; i < ArraySize(folders); i++)
      {
         string path = base_name + "\\\\" + folders[i];
         if(!FolderCreate(path, FILE_COMMON))
         {
            if(GetLastError() != 5002)  // Already exists
               return false;
         }
      }
      return true;
   }
   
   static bool CleanOldFiles(string folder, string extension, int older_than_days)
   {
      datetime cutoff = TimeCurrent() - older_than_days * 86400;
      
      string file_name;
      long search = FileFindFirst(folder + "\\\\*" + extension, file_name, FILE_COMMON);
      
      if(search != INVALID_HANDLE)
      {
         do
         {
            datetime file_time = (datetime)FileGetInteger(folder + "\\\\" + file_name, 
                                                         FILE_MODIFY_DATE, FILE_COMMON);
            if(file_time < cutoff)
            {
               FileDelete(folder + "\\\\" + file_name, FILE_COMMON);
            }
         }
         while(FileFindNext(search, file_name));
         
         FileFindClose(search);
      }
      
      return true;
   }
};