# Installation Guide - MQL5 Expert Development System

Complete installation instructions for setting up the MQL5 Expert Development System skill in Antigravity.

---

## 📋 Prerequisites

- **Antigravity** (Google's AI coding assistant with Gemini 2.0)
- **Git** (to clone the repository)
- **MetaTrader 5** (optional, for testing generated code)

---

## 🚀 Installation Methods

Choose the installation method that best fits your workflow:

### Method 1: Local Installation (Project-Specific) ⭐ **RECOMMENDED**

Install the skill in a specific project's workspace. The skill will only be available in that project.

**When to use**: You want the skill available only in one MQL5 project.

```powershell
# Windows (PowerShell)
# 1. Navigate to your MQL5 project directory
cd path\to\your\mql5\project

# 2. Create .agent/skills directory if it doesn't exist
New-Item -Path ".agent\skills" -ItemType Directory -Force

# 3. Clone the skill into your project
git clone https://github.com/yourusername/antigravity-mql5-skill.git .agent\skills\mql5_expert
```

```bash
# Linux / macOS
# 1. Navigate to your MQL5 project directory
cd /path/to/your/mql5/project

# 2. Create .agent/skills directory if it doesn't exist
mkdir -p .agent/skills

# 3. Clone the skill into your project
git clone https://github.com/yourusername/antigravity-mql5-skill.git .agent/skills/mql5_expert
```

**✅ Advantages:**
- Isolated to specific project
- Easy to version control with project
- No conflicts with other projects
- Can have different skill versions per project

---

### Method 2: Symbolic Link (Multi-Project) 🔗

Clone once, share across multiple projects using symbolic links.

**When to use**: You work on multiple MQL5 projects and want the same skill version everywhere.

**Step 1: Clone the repository to a central location**

```powershell
# Windows (PowerShell)
# Clone to a central location (e.g., Documents)
cd $env:USERPROFILE\Documents
git clone https://github.com/yourusername/antigravity-mql5-skill.git
```

```bash
# Linux / macOS
# Clone to a central location (e.g., home directory)
cd ~
git clone https://github.com/yourusername/antigravity-mql5-skill.git
```

**Step 2: Create symbolic links in each project**

```powershell
# Windows (PowerShell - Run as Administrator)
# For each project, create a symbolic link:
cd path\to\project1
New-Item -ItemType SymbolicLink `
         -Path ".agent\skills\mql5_expert" `
         -Target "$env:USERPROFILE\Documents\antigravity-mql5-skill"

cd path\to\project2
New-Item -ItemType SymbolicLink `
         -Path ".agent\skills\mql5_expert" `
         -Target "$env:USERPROFILE\Documents\antigravity-mql5-skill"
```

```bash
# Linux / macOS
# For each project, create a symbolic link:
cd /path/to/project1
ln -s ~/antigravity-mql5-skill .agent/skills/mql5_expert

cd /path/to/project2
ln -s ~/antigravity-mql5-skill .agent/skills/mql5_expert
```

**✅ Advantages:**
- Single source of truth
- Easy updates (update once, affects all projects)
- Saves disk space
- Consistent across all projects

**⚠️ Requirements:**
- Windows: Requires Administrator privileges
- All projects must have access to the central location

---

### Method 3: Global Installation 🌐

Install once for all Antigravity projects system-wide.

**When to use**: You want the skill available in every Antigravity workspace automatically.

```powershell
# Windows (PowerShell)
# Clone to Antigravity's global skills directory
git clone https://github.com/yourusername/antigravity-mql5-skill.git `
          "$env:USERPROFILE\.antigravity\skills\mql5_expert"
```

```bash
# Linux / macOS
# Clone to Antigravity's global skills directory
git clone https://github.com/yourusername/antigravity-mql5-skill.git \
          ~/.antigravity/skills/mql5_expert
```

**✅ Advantages:**
- Available everywhere
- Install once, use anywhere
- No per-project setup needed

**⚠️ Disadvantages:**
- Same version across all projects
- Harder to have project-specific customizations
- May conflict with project-specific skills

---

### Method 4: Manual Download (No Git)

If you don't have Git installed or prefer manual installation.

**Step 1: Download**
1. Go to https://github.com/yourusername/antigravity-mql5-skill
2. Click "Code" → "Download ZIP"
3. Extract the ZIP file

**Step 2: Copy to destination**

Choose one of the destinations from Methods 1-3:

```powershell
# Windows - Local (Project-Specific)
Copy-Item -Path "path\to\extracted\folder" `
          -Destination "path\to\your\project\.agent\skills\mql5_expert" `
          -Recurse

# Windows - Global
Copy-Item -Path "path\to\extracted\folder" `
          -Destination "$env:USERPROFILE\.antigravity\skills\mql5_expert" `
          -Recurse
```

```bash
# Linux/macOS - Local (Project-Specific)
cp -r /path/to/extracted/folder /path/to/your/project/.agent/skills/mql5_expert

# Linux/macOS - Global
cp -r /path/to/extracted/folder ~/.antigravity/skills/mql5_expert
```

---

## ✅ Verify Installation

After installation, verify the skill is loaded:

### Option A: Check directory structure

```powershell
# Windows
Get-ChildItem -Path ".agent\skills" -Directory

# Expected output:
# Name
# ----
# mql5_expert
```

```bash
# Linux/macOS
ls -la .agent/skills/

# Expected output:
# drwxr-xr-x  mql5_expert
```

### Option B: Test in Antigravity

1. Open Antigravity in your project
2. Start a new conversation
3. Type: **"Create a simple MQL5 EA"**
4. The skill should activate automatically

**Expected behavior:**
- Antigravity recognizes MQL5 keywords
- Generates complete EA code with safety checks

---

## 🔄 Updating the Skill

### If installed via Git (Methods 1-3):

```bash
# Navigate to the skill directory
cd .agent/skills/mql5_expert    # For local installation
# OR
cd ~/.antigravity/skills/mql5_expert    # For global installation
# OR
cd ~/Documents/antigravity-mql5-skill    # For symlink source

# Update to latest version
git pull origin main
```

### If installed manually (Method 4):

1. Download the latest ZIP from GitHub
2. Extract and replace the existing folder
3. Restart Antigravity if needed

---

## 🛠️ Troubleshooting

### Skill not activating

**Problem**: Antigravity doesn't recognize MQL5 keywords

**Solutions**:
1. Verify installation path is correct
2. Check `SKILL.md` exists in the skill directory
3. Restart Antigravity
4. Try mentioning "MQL5" or "Expert Advisor" explicitly

### Permission errors (Windows symbolic links)

**Problem**: "You do not have sufficient privilege" error

**Solutions**:
1. Run PowerShell as Administrator
2. Or use Method 1 (local copy) instead
3. Or enable Developer Mode in Windows Settings

### SKILL.md not found

**Problem**: Error loading skill

**Solutions**:
1. Verify `SKILL.md` is in the skill root directory
2. Check file isn't corrupted
3. Re-clone or re-download the repository

---

## 📝 Quick Reference

| Method | Command | Scope | Updates |
|--------|---------|-------|---------|
| **Local** | Clone to `.agent/skills/mql5_expert` | Single project | Manual per project |
| **Symlink** | Link to central location | Multiple projects | Update once, affects all |
| **Global** | Clone to `~/.antigravity/skills/` | All projects | Update once, affects all |
| **Manual** | Copy extracted folder | Depends on destination | Manual replacement |

---

## 🎯 Recommended Setup

**For beginners**: Use **Method 1 (Local Installation)**
- Simplest setup
- No administrator rights needed
- Self-contained

**For power users**: Use **Method 2 (Symbolic Link)**
- Best of both worlds
- Easy updates
- Flexible per-project customization

**For casual use**: Use **Method 3 (Global)**
- Set it and forget it
- Always available

---

## 📞 Need Help?

- **Issues**: [Report on GitHub](https://github.com/yourusername/antigravity-mql5-skill/issues)
- **Discussions**: [Ask questions](https://github.com/yourusername/antigravity-mql5-skill/discussions)
- **Documentation**: See [README.md](README.md)

---

**Installation complete! Start creating amazing MQL5 EAs with AI assistance! 🚀**
