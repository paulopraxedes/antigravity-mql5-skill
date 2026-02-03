# MQL5 Security Guardrails & Vulnerability Prevention

This module defines the mandatory security standards for MQL5 development within the Antigravity system. These guardrails are designed to prevent malicious code execution, data leaks, and system compromise.

## 🛡️ Critical Vulnerabilities & Mitigations

### 1. Code Injection (DLL & Shell)
**Threat**: Malicious actors may attempt to execute arbitrary system commands via Windows DLLs (`shell32.dll`, `kernel32.dll`).
**Guardrail**:
- **Strict Prohibition**: Usage of `shell32.dll` (ShellExecute) and `kernel32.dll` (WinExec, CreateProcess) is **FORBIDDEN** unless explicitly authorized by the user for a specific system integration task.
- **Safe Imports**: Only import DLLs from trusted sources.
- **Sandboxing**: MQL5 runs in a sandbox. Do not attempt to break out of the `MQL5/Files` directory sandbox using system calls.

**❌ VULNERABLE:**
```cpp
#import "shell32.dll"
int ShellExecuteW(int hwnd, string oper, string file, string param, string dir, int show);
#import
// ...
ShellExecuteW(0, "open", "cmd.exe", "/c format c:", "", 1);
```

**✅ SECURE:**
Native MQL5 functions should be used for file operations and networking. If external execution is needed, use Python integration via `Python` library or named pipes, not raw DLL calls.

### 2. SQL Injection
**Threat**: Concatenating user inputs or variable strings directly into SQL queries allows attackers to manipulate database logic.
**Guardrail**:
- **Parameterized Queries**: Always use `DatabaseBind` to bind variables to queries.
- **Escaping**: If binding is not possible, strictly validate and escape all string inputs.

**❌ VULNERABLE:**
```cpp
string query = "INSERT INTO Trades VALUES (" + IntegerToString(ticket) + ", '" + comment + "')";
DatabaseExecute(db, query); // Comment could contain: '); DROP TABLE Trades; --
```

**✅ SECURE:**
```cpp
string query = "INSERT INTO Trades VALUES (?, ?)";
DatabasePrepare(db, query);
DatabaseBind(db, 0, ticket);
DatabaseBind(db, 1, comment); // MQL5 handles escaping automatically here
DatabaseRead(db);
```

### 3. Unsafe Web Requests
**Threat**: Sending data to unauthorized servers or downloading malicious payloads.
**Guardrail**:
- **URL Whitelisting**: `WebRequest` only works for URLs explicitly allowed in terminal settings.
- **Protocol Security**: Only use `HTTPS`.
- **Data Validation**: Validate all response data before parsing.

### 4. Input Validation (Parameter Tampering)
**Threat**: Users or optimization algorithms providing invalid inputs that crash the EA or cause logic errors (e.g., negative StopLoss, zero division).
**Guardrail**:
- **Sanity Checks**: All `input` variables MUST be validated in `OnInit()`.

**✅ SECURE:**
```cpp
input int InpPeriod = 14;

int OnInit() {
   if(InpPeriod <= 0) {
      Print("Error: InpPeriod must be > 0");
      return INIT_PARAMETERS_INCORRECT;
   }
   // ...
}
```

### 5. Buffer Overflows & Array Access
**Threat**: Accessing arrays out of bounds, crashing the terminal.
**Guardrail**:
- **Dynamic Checks**: Always check `ArraySize()` before access.
- **Safe Copying**: Use `CopyBuffer` with error checking.

---

## 🔒 Linter Security Rules

The Antigravity linter enforces the following security rules:

| ID | Rule Name | Description | Severity |
|----|-----------|-------------|----------|
| **SEC001** | `UnsafeDLLImport` | Flags imports of `kernel32`, `shell32`, `user32` | **CRITICAL** |
| **SEC002** | `SQLInjection` | Detects string concatenation in `DatabaseExecute` | **CRITICAL** |
| **SEC003** | `UnvalidatedWebRequest` | Checks for `WebRequest` usage warnings | **WARNING** |
| **SEC004** | `InputSanityCheck` | Requires validation of inputs in `OnInit` | **ERROR** |
| **SEC005** | `GlobalScopeExec` | No code execution allowed in global scope | **CRITICAL** |

## 🛠️ Best Practices

1. **Principle of Least Privilege**: Only request permissions (DLL, WebRequest) if absolutely necessary.
2. **Defense in Depth**: Assume all external data (price feeds, API responses, user inputs) is potentially malformed.
3. **Fail Safe**: If an error occurs, the EA should stop trading (`ExpertRemove`) rather than continuing with undefined state.
