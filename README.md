# MQL5 Expert Development System for Antigravity

<div align="center">

![MQL5](https://img.shields.io/badge/MQL5-Expert%20Advisor-blue?style=for-the-badge&logo=metaquotes&logoColor=white)
![Antigravity](https://img.shields.io/badge/Antigravity-Skill-green?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)
![Version](https://img.shields.io/badge/Version-2.0-orange?style=for-the-badge)

**Transform Antigravity into an MQL5-Architect Pro - Expert system for creating institutional-grade MetaTrader 5 Expert Advisors and custom indicators with AI assistance.**

[Features](#-features) • [Installation](#-installation) • [Usage](#-usage) • [Examples](#-examples) • [Documentation](#-documentation) • [Contributing](#-contributing)

</div>

---

## 📖 Overview

This skill transforms [Google's Antigravity](https://deepmind.google/technologies/gemini/antigravity/) into an expert MetaTrader 5 programming assistant specialized in creating production-ready Expert Advisors (EAs) and custom indicators with institutional-grade safety standards.

### What This Skill Does

- ✅ **Generates complete Expert Advisors** with all safety checks built-in
- ✅ **Creates custom indicators** with proper buffer management
- ✅ **Debugs MQL5 code** and fixes compilation/runtime errors
- ✅ **Implements advanced risk management** (Kelly Criterion, ATR-based sizing, drawdown monitoring)
- ✅ **Reviews code quality** against 40+ linting rules
- ✅ **Optimizes performance** while maintaining safety
- ✅ **Explains MQL5 concepts** with practical examples

### Why Use This Skill?

- 🎯 **Production-Ready Code**: No placeholders or TODOs - every line is complete and functional
- 🛡️ **Safety-First**: All critical validations included (stop levels, lot normalization, margin checks)
- 📚 **Comprehensive Knowledge**: 7,040 pages of MQL5 reference + 14 specialized modules
- 🏗️ **Institutional Standards**: Follows best practices from professional trading firms
- ⚡ **Time-Saving**: Generate in minutes what would take hours to code manually
- 🔍 **Educational**: Learn MQL5 best practices through high-quality examples

---

## 🎯 Features

### Core Capabilities

| Feature | Description |
|---------|-------------|
| **Expert Advisor Generation** | Complete EA templates with entry/exit logic, risk management, and error handling |
| **Custom Indicators** | Multi-buffer indicators with efficient calculations and proper memory management |
| **Code Debugging** | Identifies and fixes compilation errors, runtime issues, and trading errors |
| **Risk Management** | Kelly Criterion, volatility-based sizing, drawdown monitoring, circuit breakers |
| **Code Review** | Validates against 40+ safety and quality rules |
| **Performance Optimization** | Efficient loops, caching strategies, memory optimization |
| **Concept Explanation** | Clear explanations with practical code examples |

### Safety Standards

Every generated code includes:

- ✅ `#property strict` mode enforcement
- ✅ Stop level validation (`SYMBOL_TRADE_STOPS_LEVEL`)
- ✅ Lot normalization (`SYMBOL_VOLUME_STEP`)
- ✅ Margin verification before trades
- ✅ Comprehensive error handling
- ✅ New bar detection filter (prevents tick-flooding)
- ✅ Resource cleanup (indicator handles, file handles)
- ✅ Magic number configuration
- ✅ Filling mode setup

### Advanced Features

- **Risk Management Patterns**:
  - Risk-based position sizing
  - Kelly Criterion implementation
  - ATR-based volatility adjustment
  - Drawdown monitoring with auto-shutoff
  - Correlation risk management

- **Code Quality**:
  - Function contracts (Purpose, Inputs, Outputs)
  - Meaningful variable names
  - Comprehensive comments
  - Modern (CTrade) and legacy (OrderSend) approaches
  - RAII pattern for resource management

- **Performance**:
  - Loop invariant optimization
  - Indicator value caching
  - Efficient string handling
  - Minimal memory footprint

---

## 📦 Installation

### Prerequisites

- **Antigravity** (Google's AI coding assistant with Gemini 2.0)
- **MetaTrader 5** (for testing generated code)
- **Git** (for cloning the repository)

### Step 1: Clone the Repository

```bash
git clone https://github.com/yourusername/mql5-expert-antigravity-skill.git
cd mql5-expert-antigravity-skill
```

### Step 2: Install in Antigravity

Choose one of the following methods:

#### Method A: Copy to Workspace (Recommended for Single Project)

Navigate to your MQL5 project workspace and copy the skill:

```powershell
# Windows (PowerShell)
cd path\to\your\mql5\workspace
Copy-Item -Path "path\to\mql5-expert-antigravity-skill" `
          -Destination ".\.agent\skills\mql5_expert" `
          -Recurse
```

```bash
# Linux/macOS
cd /path/to/your/mql5/workspace
cp -r /path/to/mql5-expert-antigravity-skill .agent/skills/mql5_expert
```

#### Method B: Symbolic Link (Recommended for Multiple Projects)

Create a symbolic link to use the skill across multiple workspaces:

```powershell
# Windows (PowerShell - Run as Administrator)
cd path\to\your\mql5\workspace
New-Item -ItemType SymbolicLink `
         -Path ".\.agent\skills\mql5_expert" `
         -Target "path\to\mql5-expert-antigravity-skill"
```

```bash
# Linux/macOS
cd /path/to/your/mql5/workspace
ln -s /path/to/mql5-expert-antigravity-skill .agent/skills/mql5_expert
```

#### Method C: Global Installation (Use Anywhere)

Place the skill in Antigravity's global skills directory:

```powershell
# Windows
Copy-Item -Path "path\to\mql5-expert-antigravity-skill" `
          -Destination "$env:USERPROFILE\.antigravity\skills\mql5_expert" `
          -Recurse
```

```bash
# Linux/macOS
cp -r /path/to/mql5-expert-antigravity-skill ~/.antigravity/skills/mql5_expert
```

### Step 3: Verify Installation

The skill will be automatically activated when you mention MQL5-related keywords in Antigravity. To verify:

1. Open Antigravity
2. Start a new conversation
3. Type: `List available skills` or simply start using it with phrases like:
   - "Create an EA for moving average crossover"
   - "Generate a custom RSI indicator"
   - "Fix this MQL5 compilation error"

---

## 🚀 Usage

### Activation Keywords

The skill activates automatically when you mention:

- **MetaTrader 5**, MT5, MQL5
- **Expert Advisor**, EA, trading robot
- **Custom indicator**, technical indicator
- **Automated trading**, algorithmic trading
- File extensions: `.mq5`, `.mqh`
- Phrases: "create EA", "generate indicator", "debug MQL5", "fix error"

### Basic Usage Examples

#### Example 1: Create a Simple EA

**Prompt:**
```
Create an Expert Advisor that buys when price crosses above the 20-period 
moving average and sells when it crosses below
```

**Result:**
Complete EA with:
- MA indicator initialization
- New bar detection
- Crossover logic
- Risk-based position sizing (2% default)
- All safety checks
- Error handling
- Resource cleanup

#### Example 2: Generate Custom Indicator

**Prompt:**
```
Create a custom indicator that calculates and displays the average 
of the last 14 closing prices with a blue line
```

**Result:**
Complete indicator with:
- Proper buffer setup
- Efficient calculation loop
- Drawing properties
- Memory management

#### Example 3: Fix an Error

**Prompt:**
```
My EA is giving "invalid stops" error. The code sets SL 10 points from price.
How do I fix this?
```

**Result:**
- Explanation of the error
- Complete `ValidateStops()` function
- Corrected code example
- Prevention tips

#### Example 4: Implement Risk Management

**Prompt:**
```
Add ATR-based position sizing to my EA with 2% risk per trade
```

**Result:**
- Complete ATR initialization
- Volatility-based lot calculation
- Integration with existing EA
- Cleanup code

---

## 💡 Examples

### Complete EA: RSI Strategy

**Prompt:**
```
Create an EA that trades RSI overbought/oversold:
- RSI period: 14
- Buy when RSI < 30
- Sell when RSI > 70
- Risk: 2% per trade
- One position at a time
```

**Generated Code:**
- ✅ 200+ lines of production-ready code
- ✅ RSI indicator setup
- ✅ Signal detection logic
- ✅ Position management
- ✅ Risk calculation
- ✅ All safety validations
- ✅ Complete error handling

### Advanced: Multi-Indicator EA

**Prompt:**
```
Create an EA combining RSI and Moving Averages:
- Buy: RSI < 30 AND price > MA(50)
- Sell: RSI > 70 AND price < MA(50)
- Stop loss: 2 * ATR(14)
- Risk: 2% per trade
- Trailing stop when profit > 1%
```

**Generated Code:**
- ✅ Multiple indicator management
- ✅ Complex entry conditions
- ✅ Dynamic stop loss based on ATR
- ✅ Trailing stop implementation
- ✅ Advanced risk management
- ✅ Position tracking

### Custom Indicator: Trend Strength

**Prompt:**
```
Create an indicator showing trend strength:
- Calculate difference between MA(10) and MA(20)
- Normalize by ATR(14)
- Draw in separate window
- Green line when positive, red when negative
```

**Generated Code:**
- ✅ Multi-buffer setup
- ✅ Normalized calculations
- ✅ Color switching logic
- ✅ Separate window display

---

## 📚 Documentation

### Project Structure

```
mql5-expert-antigravity-skill/
├── SKILL.md                    # Main skill file (loaded by Antigravity)
├── README.md                   # This file
├── LICENSE                     # MIT License
├── INSTALL.md                  # Installation instructions
├── VALIDATION.md               # Quality validation checklist
└── resources/
    ├── mql5.pdf                # Full MQL5 Reference (7,040 pages)
    ├── config/
    │   ├── code_templates.json # Reusable code templates
    │   └── linter_rules.yaml   # 40+ safety and quality rules
    └── reference/
        ├── language_core.md       # Core MQL5 syntax & types
        ├── trading_engine.md      # Order execution patterns
        ├── risk_management.md     # Position sizing & risk control
        ├── indicators.md          # Custom indicator development
        ├── event_driven.md        # Event handlers (OnTick, OnTimer, etc.)
        ├── oop_architecture.md    # Class design patterns
        ├── standard_library.md    # CTrade, CSymbolInfo, Collections
        ├── file_io.md             # File operations & persistence
        ├── time_series.md         # Time series analysis
        ├── testing.md             # Strategy Tester & optimization
        ├── network.md             # WebRequest & REST APIs
        ├── gpu_opencl.md          # GPU acceleration
        ├── machine_learning.md    # ML integration
        └── chart_objects.md       # Drawing objects & GUI
```

### Knowledge Modules

The skill contains 14 specialized knowledge modules:

| Module | Coverage | File Size |
|--------|----------|-----------|
| **Language Core** | Syntax, types, memory, preprocessor | 6.3 KB |
| **Trading Engine** | CTrade, OrderSend, position management | 12.9 KB |
| **Risk Management** | Kelly, ATR sizing, drawdown monitoring | 11.4 KB |
| **Indicators** | Custom indicators, buffers, multi-timeframe | 8.7 KB |
| **Event-Driven** | OnTick, OnTimer, OnTrade, OnChartEvent | 7.4 KB |
| **OOP Architecture** | Classes, RAII, dependency injection | 6.5 KB |
| **Standard Library** | CTrade, CSymbolInfo, collections | 10.0 KB |
| **File I/O** | CSV, binary files, data persistence | 6.9 KB |
| **Time Series** | Seasonality, pattern detection | 6.7 KB |
| **Testing** | Strategy Tester, optimization | 9.2 KB |
| **Network** | WebRequest, REST APIs | 5.2 KB |
| **GPU/OpenCL** | Hardware acceleration | 5.3 KB |
| **Machine Learning** | Feature engineering, model integration | 6.0 KB |
| **Chart Objects** | Drawing, GUI elements | 5.4 KB |

### Code Standards

All generated code follows these standards:

- **Naming**: Classes=`CPascalCase`, Functions=`PascalCase`, Variables=`camelCase`, Globals=`g_prefix`
- **Indentation**: 3 spaces (MQL5 standard)
- **Braces**: 1TBS (One True Brace Style)
- **Documentation**: Function contracts for all public methods
- **Error Handling**: Every trade operation checks for errors
- **Resource Management**: All handles released in `OnDeinit()`

---

## 🔒 Safety & Quality

### Mandatory Safety Checks

Every EA generated includes:

1. **Stop Level Validation**
   ```cpp
   // Validates SL/TP against broker's minimum distance
   bool ValidateStops(ENUM_ORDER_TYPE type, double price, double sl, double tp)
   ```

2. **Lot Normalization**
   ```cpp
   // Normalizes to broker's lot step (0.01, 0.1, etc.)
   double NormalizeLots(double lots)
   ```

3. **Margin Verification**
   ```cpp
   // Checks available margin before trading
   bool CheckMargin(ENUM_ORDER_TYPE type, double lots, double price)
   ```

4. **New Bar Detection**
   ```cpp
   // Prevents tick-flooding and over-trading
   bool IsNewBar()
   ```

5. **Resource Cleanup**
   ```cpp
   void OnDeinit(const int reason) {
      IndicatorRelease(handle);  // ALWAYS release handles
   }
   ```

### Linting Rules

40+ rules across categories:

- **Critical Safety** (8 rules): Magic number, stop validation, lot normalization, error handling
- **Memory Management** (4 rules): Handle release, file close, object deletion, timer cleanup
- **Code Quality** (5 rules): Strict mode, copyright, function length, global limits
- **Performance** (5 rules): Loop optimization, string handling, buffer caching
- **Architecture** (4 rules): Header guards, function contracts, dependency injection
- **Security** (4 rules): Memory leaks, buffer overflow, SQL injection, credentials
- **Style** (3 rules): Indentation, line length, whitespace

---

## 🎓 Learning Resources

### Included Documentation

- **SKILL.md** (35 KB): Complete skill reference with all patterns and templates
- **resources/mql5.pdf** (32 MB): Official MQL5 Reference (7,040 pages, 2025 Edition)
- **14 Module Files** (~135 KB): Detailed guides on specific topics
- **code_templates.json**: 6 pre-validated code templates
- **linter_rules.yaml**: Complete rule definitions with examples

### External Resources

- [MQL5 Official Documentation](https://www.mql5.com/en/docs)
- [MetaTrader 5 Platform](https://www.metatrader5.com/)
- [MQL5 Community](https://www.mql5.com/en/forum)
- [Antigravity Documentation](https://deepmind.google/technologies/gemini/antigravity/)

---

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

### Ways to Contribute

1. **Report Bugs**: Open an issue describing the problem
2. **Suggest Features**: Propose new capabilities or improvements
3. **Improve Documentation**: Fix typos, add examples, clarify instructions
4. **Add Templates**: Contribute new code templates or patterns
5. **Enhance Rules**: Suggest new linting rules or improve existing ones
6. **Share Examples**: Submit real-world EA/indicator examples

### Contribution Process

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Test thoroughly (generate EAs, verify they compile and run)
5. Commit with clear messages (`git commit -m 'Add amazing feature'`)
6. Push to your fork (`git push origin feature/amazing-feature`)
7. Open a Pull Request

### Code Guidelines

When contributing code:

- Follow the existing code style (see `SKILL.md` for standards)
- Include function contracts for new templates
- Add safety checks where applicable
- Test generated code compiles without errors
- Update documentation if needed

---

## 📊 Statistics

| Metric | Value |
|--------|-------|
| **Main Skill File** | 35 KB |
| **Knowledge Modules** | 14 specialized files |
| **Total Knowledge Base** | ~135 KB (text) + 32 MB (PDF) |
| **Code Templates** | 6 pre-validated templates |
| **Linting Rules** | 40+ safety & quality rules |
| **Safety Patterns** | 5+ critical patterns |
| **Risk Patterns** | 3+ advanced strategies |
| **MQL5 Reference** | 7,040 pages (2025 Edition) |
| **Lines of Code** | Generated EAs typically 200-500 lines |

---

## 🔍 FAQ

### Q: Do I need to know MQL5 to use this skill?

**A:** No! The skill generates complete, production-ready code. However, basic understanding helps you customize and optimize the results.

### Q: Will the generated code compile without errors?

**A:** Yes. All generated code is designed to compile cleanly. If you encounter errors, you can ask the skill to fix them.

### Q: Can I use this for live trading?

**A:** The code follows institutional safety standards, but **always test thoroughly in demo first**. Trading involves risk - never trade with money you can't afford to lose.

### Q: Does it work with MQL4 (MT4)?

**A:** No. This skill is specifically for MQL5 (MetaTrader 5). MQL4 has different syntax and architecture.

### Q: Can I modify the generated code?

**A:** Absolutely! The code is yours to modify. The skill can also help you make changes.

### Q: What if I need help with existing code?

**A:** The skill can review, debug, and optimize your existing MQL5 code. Just paste it and ask for help.

### Q: Is there a limit to EA complexity?

**A:** No hard limits. The skill can generate simple 100-line EAs or complex 1000+ line systems with multiple indicators and advanced logic.

### Q: Can it generate indicators for separate windows?

**A:** Yes! The skill handles both chart window and separate window indicators with proper buffer management.

---

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

### What This Means

- ✅ **Commercial Use**: Use in commercial projects
- ✅ **Modification**: Modify and adapt to your needs
- ✅ **Distribution**: Share with others
- ✅ **Private Use**: Use privately without publishing
- ⚠️ **Attribution**: Include original copyright notice
- ⚠️ **No Warranty**: Provided "as is" without warranty

### Third-Party Content

- **MQL5 Reference** (resources/mql5.pdf): © MetaQuotes Ltd. - Redistributed for educational purposes
- **Antigravity Platform**: © Google DeepMind - This skill is an independent addon

---

## 🙏 Acknowledgments

- **MetaQuotes Software Corp.** for MetaTrader 5 and MQL5
- **Google DeepMind** for Antigravity platform
- **MQL5 Community** for shared knowledge and best practices
- **Contributors** who help improve this skill

---

## 📞 Support

### Getting Help

- **Issues**: [GitHub Issues](https://github.com/yourusername/mql5-expert-antigravity-skill/issues)
- **Discussions**: [GitHub Discussions](https://github.com/yourusername/mql5-expert-antigravity-skill/discussions)
- **MQL5 Forum**: [MQL5 Community](https://www.mql5.com/en/forum)

### Reporting Bugs

When reporting bugs, please include:

1. Your Antigravity version
2. The prompt you used
3. Generated code (if applicable)
4. Error messages
5. Expected vs actual behavior

### Feature Requests

Have an idea? Open an issue with the `enhancement` label and describe:

1. The feature you'd like
2. Why it would be useful
3. Example use cases
4. Any implementation ideas

---

## 🗺️ Roadmap

### Current Version: 2.0

- ✅ Complete EA generation
- ✅ Custom indicator creation
- ✅ Code review and debugging
- ✅ Advanced risk management
- ✅ 40+ linting rules
- ✅ 14 knowledge modules

### Planned Features

- [ ] MQL5 Strategy Tester integration prompts
- [ ] Backtesting result analysis
- [ ] Multi-symbol EA templates
- [ ] Portfolio management strategies
- [ ] Machine learning EA templates
- [ ] WebRequest/API integration examples
- [ ] Chart pattern recognition indicators
- [ ] Advanced order flow analysis
- [ ] Interoperability with Python (via DLL)
- [ ] Video tutorials and walkthroughs

---

## 🌟 Showcase

### Community Projects

Have you built something amazing with this skill? Let us know!

Submit your project to be featured here:
1. Create a showcase issue
2. Include description, screenshots, and results
3. Share your experience

---

## 📈 Version History

### Version 2.0 (2026-02-01)
- ✅ Complete reorganization for Antigravity compatibility
- ✅ Consolidated SKILL.md with YAML frontmatter
- ✅ 14 specialized knowledge modules
- ✅ 40+ linting rules
- ✅ Advanced risk management patterns
- ✅ Comprehensive documentation

### Version 1.0 (Initial Development)
- ✅ Basic EA templates
- ✅ Core safety patterns
- ✅ Initial knowledge base

---

<div align="center">

**Made with ❤️ for the MQL5 and Antigravity communities**

⭐ **Star this repo** if you find it useful!

[Report Bug](https://github.com/yourusername/mql5-expert-antigravity-skill/issues) · [Request Feature](https://github.com/yourusername/mql5-expert-antigravity-skill/issues) · [Share Your Success](https://github.com/yourusername/mql5-expert-antigravity-skill/discussions)

---

### Quick Links

[🏠 Home](#mql5-expert-development-system-for-antigravity) · [📦 Install](#-installation) · [🚀 Usage](#-usage) · [📚 Docs](#-documentation) · [🤝 Contribute](#-contributing) · [📄 License](#-license)

</div>
