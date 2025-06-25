# 🔍 NetSnoop - Universal Process Monitor & Anomaly Detection

<div align="center">

![Python](https://img.shields.io/badge/python-3.6+-blue.svg)
![Platform](https://img.shields.io/badge/platform-Linux-green.svg)
![License](https://img.shields.io/badge/license-MIT-yellow.svg)
![Status](https://img.shields.io/badge/status-Active-brightgreen.svg)

**Real-time process monitoring and anomaly detection for Linux systems**

*Language-agnostic process burst detection with intelligent CPU monitoring*

</div>

---

## 🚀 Features

### 🔧 **Process Monitoring**
- **Real-time process detection** - Monitors new process spawns continuously
- **Complete process genealogy** - Traces full parent-child relationships
- **Multi-user support** - Tracks processes across all system users
- **Detailed process information** - Captures executable paths, command lines, and user contexts

### 🚨 **Anomaly Detection**
- **Process burst detection** - Identifies suspicious rapid process spawning
- **Intelligent instigator tracing** - Automatically identifies the root cause process
- **Smart filtering** - Ignores safe system processes to reduce false positives
- **Grouped alerts** - Batches related anomalies to prevent spam

### 💻 **CPU Monitoring**
- **System-wide CPU tracking** - Monitors overall system CPU usage
- **Per-process CPU analysis** - Tracks individual process CPU consumption
- **Multi-level alerting** - Critical, suspicious, and high usage alerts
- **Load average integration** - Correlates CPU usage with system load

### 📊 **Logging & Persistence**
- **Persistent logging** - All events saved to `netsnoop_persistent.txt`
- **Timestamped records** - IST timezone support for accurate timestamps
- **Session tracking** - Clear session boundaries with start/end markers
- **Graceful shutdown** - Proper cleanup and final log entries on exit

---

## 🛠️ Installation

### Prerequisites
- **Python 3.6+**
- **Linux system** (Ubuntu, Debian, CentOS, etc.)
- **Root/sudo access** (for comprehensive process monitoring)

### Quick Start
```bash
# Clone the repository
git clone https://github.com/yourusername/netsnoop.git
cd netsnoop

# Make executable
chmod +x netsnoop.py

# Run with elevated privileges
sudo python3 netsnoop.py
```

---

## 🎯 Usage

### Basic Monitoring
```bash
# Standard monitoring mode
sudo python3 netsnoop.py
```

### Debug Mode
```bash
# Enable debug output (modify DEBUG_MODE in script)
# Set DEBUG_MODE = True for verbose console output
sudo python3 netsnoop.py
```

### Background Operation
```bash
# Run in background
sudo nohup python3 netsnoop.py > /dev/null 2>&1 &

# Stop background process
sudo pkill -f netsnoop.py
```

---

## 📈 Configuration

### Alert Thresholds
```python
# Process burst detection
burst_threshold = 8      # Number of processes to trigger alert
burst_window = 3         # Time window in seconds

# CPU monitoring
CPU_SAMPLE_WINDOW = 10   # Number of samples to keep
CPU_GROUP_WINDOW = 3     # Alert grouping window
CPU_GROUP_COOLDOWN = 10  # Cooldown between similar alerts
```

### CPU Alert Levels
- **🧨 CRITICAL**: >95% CPU usage for 5+ seconds
- **🚩 SUSPICIOUS**: >90% CPU usage (immediate alert)
- **🔺 HIGH**: >80% CPU usage for 3+ consecutive samples

---

## 📋 Output Examples

### Process Detection
```
[14:32:15] 🔧 New Process Detected:
    └── systemd (PID 1, User: root)
        └── SessionRelay (PID 1234, User: user)
            └── bash (PID 5678, User: user)
                └── python3 (PID 9101, User: user)
                    ├── Executable: /usr/bin/python3
                    └── CmdLine: python3 my_script.py
```

### Anomaly Alerts
```
⚠️  Multiple Anomalies Detected (Grouped):
  • [14:32:20] PID 5678 → 12 spawns — python3 build_script.py (PID 9101)
  • [14:32:22] PID 5678 → 8 spawns — gcc -O2 main.c (PID 9205)
```

### CPU Alerts
```
🧨 CRITICAL CPU: PID 9101 (python3) 98.5% for 5+s → python3 intensive_task.py
🚩 SUSPICIOUS CPU SPIKE: PID 9205 (gcc) 94.2% → gcc -O2 large_project.c
🔺 HIGH CPU ALERT: PID 9301 (make) 87.3% for 3s → make -j8 all
```

---

## 🏗️ Architecture

### System Workflow

```
┌─────────────────┐
│   🚀 START      │
│   NetSnoop      │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│ Initialize      │
│ • Setup logging │
│ • Signal handlers│
│ • Session start │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│ MAIN LOOP       │
│ Scan /proc/     │
│ for new PIDs    │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐      NO     ┌─────────────────┐
│ New PIDs        │─────────────▶│ Check CPU       │
│ detected?       │              │ Anomalies       │
└─────────┬───────┘              └─────────┬───────┘
          │ YES                            │
          ▼                                │
┌─────────────────┐                       │
│ Build Process   │                       │
│ Chain Tree      │                       │
│ • Parent-child  │                       │
│ • User info     │                       │
│ • Command line  │                       │
└─────────┬───────┘                       │
          │                                │
          ▼                                │
┌─────────────────┐                       │
│ Log Process     │                       │
│ Details to      │                       │
│ Persistent File │                       │
└─────────┬───────┘                       │
          │                                │
          ▼                                │
┌─────────────────┐      NO                │
│ Process Burst   │──────────┐             │
│ Detected?       │          │             │
│ (>8 in 3 sec)   │          │             │
└─────────┬───────┘          │             │
          │ YES              │             │
          ▼                  │             │
┌─────────────────┐          │             │
│ Find Common     │          │             │
│ Parent PID      │          │             │
│ (Most frequent) │          │             │
└─────────┬───────┘          │             │
          │                  │             │
          ▼                  │             │
┌─────────────────┐   YES    │             │
│ Safe System     │──────────┤             │
│ Process?        │          │             │
│ (systemd, etc.) │          │             │
└─────────┬───────┘          │             │
          │ NO               │             │
          ▼                  │             │
┌─────────────────┐          │             │
│ Trace Real      │          │             │
│ Instigator      │          │             │
│ • Walk up tree  │          │             │
│ • Find scripts  │          │             │
│ • Identify user │          │             │
│   programs      │          │             │
└─────────┬───────┘          │             │
          │                  │             │
          ▼                  │             │
┌─────────────────┐          │             │
│ Buffer Anomaly  │          │             │
│ Alert (Group    │          │             │
│ for 5 seconds)  │          │             │
└─────────┬───────┘          │             │
          │                  │             │
          └──────────────────┘             │
                             │             │
                             ▼             │
                   ┌─────────────────┐     │
                   │ Check CPU       │◀────┘
                   │ Anomalies       │
                   │ • System CPU    │
                   │ • Process CPU   │
                   │ • Load average  │
                   └─────────┬───────┘
                             │
                             ▼
                   ┌─────────────────┐
                   │ CPU Threshold   │
                   │ Analysis        │
                   │ >95% = Critical │
                   │ >90% = Suspicious│
                   │ >80% = High     │
                   └─────────┬───────┘
                             │
                             ▼
                   ┌─────────────────┐
                   │ Group Similar   │
                   │ CPU Alerts      │
                   │ (3 sec window)  │
                   └─────────┬───────┘
                             │
                             ▼
                   ┌─────────────────┐
                   │ Display Alerts  │
                   │ • Process bursts│
                   │ • CPU warnings  │
                   │ • Log to file   │
                   └─────────┬───────┘
                             │
                             ▼
                   ┌─────────────────┐
                   │ Cleanup & Sleep │
                   │ • Remove old    │
                   │   process data  │
                   │ • Wait 1 second │
                   └─────────┬───────┘
                             │
                             └─────────┐
                                       │
          ┌────────────────────────────┘
          │
          ▼
┌─────────────────┐
│ Back to         │
│ MAIN LOOP       │
└─────────────────┘

    Signal (Ctrl+C)
          │
          ▼
┌─────────────────┐
│ Graceful        │
│ Shutdown        │
│ • Flush alerts  │
│ • Log session   │
│   end           │
│ • Exit cleanly  │
└─────────────────┘
```

### Core Components

```mermaid
graph TD
    A[Process Monitor] --> B[PID Detection]
    A --> C[Process Chain Builder]
    A --> D[Anomaly Detection]
    
    B --> E[Real-time Scanning]
    C --> F[Parent-Child Mapping]
    D --> G[Burst Detection]
    D --> H[Instigator Tracing]
    
    I[CPU Monitor] --> J[System CPU Stats]
    I --> K[Per-Process Stats]
    I --> L[Alert Classification]
    
    M[Logger] --> N[Persistent Storage]
    M --> O[Session Management]
    M --> P[Timestamp Handling]
```

### Key Algorithms

**Instigator Tracing**
1. Traverse process tree upward
2. Identify meaningful user programs
3. Filter out system processes
4. Find script executors and compilers
5. Return most likely root cause

**CPU Anomaly Detection**
1. Sample CPU usage every second
2. Track consecutive high usage periods
3. Group similar alerts to prevent spam
4. Correlate with system load average

---

## 🔧 Advanced Configuration

### Safe Process Filtering
```python
SAFE_PARENT_NAMES = {
    "systemd", "init", "rsyslogd", "cron", "agetty", 
    "dbus-daemon", "systemd-journal", "bash", "login"
}
```

### Custom Alert Colors
```python
CYAN = "\033[96m"      # Info messages
YELLOW = "\033[93m"    # Warnings
RED = "\033[91m"       # Critical alerts
GREEN = "\033[92m"     # Success messages
MAGENTA = "\033[95m"   # High CPU alerts
BLINK = "\033[5m"      # Critical emphasis
BOLD = "\033[1m"       # Alert emphasis
```

---

## 📁 File Structure

```
netsnoop/
├── netsnoop.py                 # Main monitoring script
├── netsnoop_persistent.txt     # Log file (auto-created)
├── README.md                   # This file
└── LICENSE                     # License file
```

---

## 🚨 Use Cases

### Security Monitoring
- **Malware detection** - Identify suspicious process spawning patterns
- **Intrusion detection** - Monitor for unusual system activity
- **Privilege escalation** - Track process execution across users

### Development & Debugging
- **Build system monitoring** - Track compiler and build tool activity
- **Performance analysis** - Identify CPU-intensive processes
- **Resource leak detection** - Monitor for runaway processes

### System Administration
- **Capacity planning** - Understand system usage patterns
- **Troubleshooting** - Identify processes causing system issues
- **Audit logging** - Maintain detailed process execution records

---

## 🤝 Contributing

We welcome contributions! Please feel free to submit issues, feature requests, or pull requests.

### Development Setup
```bash
git clone https://github.com/yourusername/netsnoop.git
cd netsnoop

# Create feature branch
git checkout -b feature/your-feature-name

# Make changes and test
sudo python3 netsnoop.py

# Submit pull request
```

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- Built for Linux systems using `/proc` filesystem
- Inspired by the need for intelligent process monitoring
- Designed with system administrators and security professionals in mind

---

<div align="center">

**⭐ Star this repository if you find it useful!**

[Report Bug](https://github.com/yourusername/netsnoop/issues) · [Request Feature](https://github.com/yourusername/netsnoop/issues) · [Documentation](https://github.com/yourusername/netsnoop/wiki)

</div>
