🎬 Technology Mask Man — Full Setup Guide

Video Title: I Built an AI That Hacks Websites For Me Legally | HexStrike + Kali Linux + Cloud LLM

Stack: Kali Linux → HexStrike AI → Claude Desktop → Cloud LLM → PortSwigger Lab
📋 Prerequisites

Before starting, confirm your environment:

Baash : 

# Verify Kali version
    lsb_release -a

# Verify Python
    python3 --version    # Must be 3.8+

# Verify Node.js (for Claude Desktop build)
    node --version
    npm --version

# Verify git
    git --version

Screen text: “Kali Linux Rolling | Python 3.11 | Node 20 | All green.”


🧩 Part 1: Install HexStrike AI

HexStrike is the bridge that exposes 150+ security tools to your LLM.
Option A: APT Install (Fastest)

bash

    sudo apt update
    sudo apt install -y hexstrike-ai


This installs both hexstrike_server and hexstrike_mcp as system binaries.

Option B: Manual Install (Recommended for Video — More Control)

bash

    cd ~/Desktop
    git clone https://github.com/0x4m4/hexstrike-ai.git
    cd hexstrike-ai

# Create and activate virtual environment
    python3 -m venv hexstrike-env
    source hexstrike-env/bin/activate

# Install Python dependencies
    pip3 install -r requirements.txt

Screen note: Show the hexstrike-env folder appearing.

🛠️ Part 2: Install Security Tools

HexStrike orchestrates the tools, but they must be installed separately.

bash

# Network & reconnaissance
    sudo apt install -y nmap masscan rustscan amass subfinder nuclei fierce dnsenum autorecon theharvester responder netexec enum4linux-ng

# Web application security
    sudo apt install -y gobuster feroxbuster dirsearch ffuf dirb katana nikto sqlmap wpscan arjun paramspider wafw00f

# Password & authentication
    sudo apt install -y hydra john hashcat medusa patator crackmapexec evil-winrm hash-identifier

# Binary analysis & reverse engineering
    sudo apt install -y gdb radare2 binwalk ghidra checksec foremost steghide exiftool

# Cloud security
    sudo apt install -y trivy

Screen note: Fast-forward this section. Show the list on screen with a “53 tools installed” counter.


🚀 Part 3: Start the HexStrike Server

Open a dedicated terminal and leave it running.

Bash : 

    cd ~/Desktop/hexstrike-ai
    source hexstrike-env/bin/activate
    python3 hexstrike_server.py --port 8888

You'll see the HexStrike banner and:

text

    [INFO] Server starting on 127.0.0.1:8888
    [INFO] 150+ integrated modules | Adaptive AI decision engine active

Screen text: “HexStrike server live on port 8888.”

✅ Health Check (New Terminal)

bash

    curl -s http://localhost:8888/health | jq

Expected: {"status":"healthy","tools_available":73} (number varies).

Common Issue: connection refused
Fix: Make sure the server terminal is still running and you used the correct port.

🖥️ Part 4: Build & Install Claude Desktop

Claude Desktop is your MCP client — the GUI that connects the LLM to HexStrike.
Step 1: Clone the Debian Packaging Repo

bash

    cd ~/Desktop/hexstrike-ai
    git clone https://github.com/aaddrick/claude-desktop-debian.git
    cd claude-desktop-debian

Step 2: Build the .deb Package

bash

    chmod +x build.sh
    ./build.sh --build deb --clean yes

This downloads the official Claude Desktop installer, replaces the Windows native module with a Linux-compatible one, and packages it.

Common Issue: Missing dependencies
Fix:

bash

    sudo apt install -y nodejs npm jq wget curl

Step 3: Install the Built Package

After the build completes (5–15 minutes), you'll see two .deb files:

bash

    ls -la *.deb

Install the large one (the real package):

bash

    sudo dpkg -i claude-desktop-unofficial_*.deb
    sudo apt --fix-broken install -y

Common Issue: chrome-sandbox permissions
Fix: The installer usually handles this. If you see an error:

bash

    sudo chown root:root /usr/lib/claude-desktop-unofficial/chrome-sandbox
    sudo chmod 4755 /usr/lib/claude-desktop-unofficial/chrome-sandbox

Step 4: Launch Once to Create Config

bash

    claude-desktop-unofficial &

Close the window after it opens. Then verify the config directory:

bash

    ls -la ~/.config/Claude/

Common Issue: claude-desktop: command not found
Fix: The binary is named claude-desktop-unofficial, not claude-desktop.

🔌 Part 5: Configure HexStrike MCP Server
Step 1: Edit the Config File

bash

    nano ~/.config/Claude/claude_desktop_config.json

Step 2: Add the MCP Server Block

Merge this into the existing JSON (keep your preferences block):


    json
    {
      "mcpServers": {
        "hexstrike-ai": {
          "command": "/home/kali/Desktop/hexstrike-ai/hexstrike-env/bin/python",
          "args": [
            "/home/kali/Desktop/hexstrike-ai/hexstrike_mcp.py",
            "--server",
            "http://localhost:8888"
          ],
          "env": {
            "HEXSTRIKE_SERVER": "http://localhost:8888",
            "HEXSTRIKE_TIMEOUT": "300"
          }
        }
      }
    }


Critical notes:

    command must be the Python inside your virtual environment.

    --server port must match your running HexStrike server (8888 by default).

    Use absolute paths, not ~.

Common Issue: MCP server disconnects immediately
Fix: Test the MCP script manually:
bash

    /home/kali/Desktop/hexstrike-ai/hexstrike-env/bin/python /home/kali/Desktop/hexstrike-ai/hexstrike_mcp.py --server http://localhost:8888 --debug

If it errors, you'll see the exact reason in the terminal.
✅ Part 6: Launch & Verify
Step 1: Restart Claude Desktop
bash

    pkill -f claude-desktop-unofficial
    sleep 2
    claude-desktop-unofficial &

Step 2: Verify MCP Connection

In a new conversation, type:

    What tools do you have available?

Claude should list tools like:

    nmap_scan()

    gobuster_scan()

    nuclei_scan()

    sqlmap_scan()

    httpx_scan()

Common Issue: No tools appear
Fix checklist:

    Is hexstrike_server.py still running?

    Are paths in config 100% correct? (Run cat ~/.config/Claude/claude_desktop_config.json)

    Did you fully quit Claude Desktop (not just close the window)?

    Check logs:

bash

    cat ~/.config/Claude/logs/mcp-server-hexstrike-ai.log




