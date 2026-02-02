
---

## 📁 File 15: `skills/10_network.md`

```markdown
## SKILL-010: Network Communication

**Feature ID:** NET-001  
**Domain:** Integration  
**Scope:** WebRequest, REST APIs, WebSocket, Security

---

### 10.1 REST API Client with Retry

```cpp
class CRestClient
{
private:
   string            m_base_url;
   string            m_api_key;
   int               m_timeout;
   int               m_max_retries;
   
public:
   CRestClient(string base_url, string api_key) 
      : m_base_url(base_url), m_api_key(api_key), m_timeout(5000), m_max_retries(3) {}
   
   bool Get(string endpoint, string &response)
   {
      return Request("GET", endpoint, "", response);
   }
   
   bool Post(string endpoint, string json_data, string &response)
   {
      return Request("POST", endpoint, json_data, response);
   }
   
private:
   bool Request(string method, string endpoint, string payload, string &response)
   {
      string url = m_base_url + endpoint;
      char data[], result[];
      string headers;
      int res;
      
      //--- Prepare headers
      headers = "Content-Type: application/json\r\n";
      if(StringLen(m_api_key) > 0)
         headers += "Authorization: Bearer " + m_api_key + "\r\n";
      
      //--- Convert payload
      StringToCharArray(payload, data);
      
      for(int i = 0; i < m_max_retries; i++)
      {
         res = WebRequest(method, url, headers, m_timeout, data, result, headers);
         
         if(res == 200)  // HTTP OK
         {
            response = CharArrayToString(result);
            return true;
         }
         else if(res == -1)  // System error
         {
            int err = GetLastError();
            if(err == 5203)  // No internet
            {
               Print("No internet connection");
               return false;
            }
         }
         else if(res == 401)  // Unauthorized
         {
            Print("API authentication failed");
            return false;
         }
         else if(res == 429)  // Rate limited
         {
            Sleep(1000 * (i + 1));  // Wait longer
            continue;
         }
         
         Sleep(500);
      }
      
      return false;
   }
};

---

### 10.2 Signal Webhook Receiver

```cpp
class CSignalWebhook
{
private:
   string            m_last_signal;
   datetime          m_last_time;
   
public:
   bool CheckForSignal(string url, string &signal_type, double &strength)
   {
      string response;
      if(!SendRequest(url, response))
         return false;
      
      // Prevent duplicate processing
      if(response == m_last_signal && TimeCurrent() - m_last_time < 60)
         return false;
      
      m_last_signal = response;
      m_last_time = TimeCurrent();
      
      // Parse simple format: "BUY:0.85" or "SELL:0.92"
      string parts[];
      if(StringSplit(response, ':', parts) != 2)
         return false;
      
      signal_type = parts[0];
      strength = StringToDouble(parts[1]);
      
      return true;
   }
   
private:
   bool SendRequest(string url, string &response)
   {
      char data[], result[];
      string headers;
      
      int res = WebRequest("GET", url, headers, 5000, data, result, headers);
      
      if(res == 200)
      {
         response = CharArrayToString(result);
         return true;
      }
      
      return false;
   }
};

---

### 10.3 Secure Credential Storage

```cpp
class CSecureStorage
{
public:
   //--- In real implementation, use proper encryption
   static bool SaveCredential(string key, string value)
   {
      // Hash the key for filename
      string filename = "cred_" + IntegerToString(StringHash(key)) + ".dat";
      
      int handle = FileOpen(filename, FILE_WRITE|FILE_BIN|FILE_COMMON);
      if(handle == INVALID_HANDLE) return false;
      
      // Simple XOR obfuscation (NOT secure, just basic obfuscation)
      char data[];
      StringToCharArray(value, data);
      uchar xor_key = 0x5A;
      
      for(int i = 0; i < ArraySize(data); i++)
         data[i] = (char)((uchar)data[i] ^ xor_key);
      
      FileWriteArray(handle, data);
      FileClose(handle);
      
      return true;
   }
   
   static string LoadCredential(string key)
   {
      string filename = "cred_" + IntegerToString(StringHash(key)) + ".dat";
      
      if(!FileIsExist(filename, FILE_COMMON))
         return "";
      
      int handle = FileOpen(filename, FILE_READ|FILE_BIN|FILE_COMMON);
      if(handle == INVALID_HANDLE) return "";
      
      char data[];
      FileReadArray(handle, data);
      FileClose(handle);
      
      // Deobfuscate
      uchar xor_key = 0x5A;
      for(int i = 0; i < ArraySize(data); i++)
         data[i] = (char)((uchar)data[i] ^ xor_key);
      
      return CharArrayToString(data);
   }
   
private:
   static int StringHash(string str)
   {
      int hash = 0;
      for(int i = 0; i < StringLen(str); i++)
         hash = hash * 31 + (int)StringGetCharacter(str, i);
      return hash;
   }
};