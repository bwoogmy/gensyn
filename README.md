# RL-Swarm Node Auto-Restart Setup

## 🚀 Automatic Node Restart Configuration

This guide will help you set up automatic restart functionality for your RL-Swarm node using systemd service.

### 📋 Prerequisites
- Ubuntu/Debian Linux server
- RL-Swarm node already installed
- Root or sudo access

---

## 🔧 Setup Instructions

### 1. **Replace the run script**
Replace the existing `run_rl_swarm.sh` file in your node directory with the updated version from the `runner` folder.

### 2. **Configure node parameters**
Edit the run script to set your desired parameters:

```bash
nano run_rl_swarm.sh
```

Modify the configuration at the top of the file:
```bash
# === NODE CONFIGURATION =================================================
SELECTED_SWARM="A"         # Options: "A" — Math, "B" — Math Hard
PARAM_B_CHOICE="0.5"       # Options: 0.5, 1.5, 7, 32, 72
HF_PUSH_MODELS="N"         # Options: "Y" — yes, "N" — no
HF_TOKEN_INPUT=""          # Your HuggingFace token (if HF_PUSH_MODELS="Y")
# =======================================================================
```

**Default settings:** Math, 0.5B (suitable for CPU nodes)

**Save and exit nano:**
- Press `CTRL + O` (save)
- Press `Enter`
- Press `CTRL + X` (exit)

### 3. **Set execution permissions**
```bash
chmod +x run_rl_swarm.sh
```

### 4. **Test manual startup**
Run the node manually to verify configuration and create necessary files:

```bash
./run_rl_swarm.sh
```

### 5. **Stop the process**
After the node starts successfully:
- Press `CTRL + C` to stop

### 6. **Create systemd service**
Create the auto-restart service:

```bash
sudo tee /etc/systemd/system/rl_swarm.service > /dev/null <<'EOF'
[Unit]
Description=RL-Swarm training (GSM8K) – venv
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
WorkingDirectory=/root/rl-swarm
ExecStart=/usr/bin/env bash -c 'source /root/rl-swarm/.venv/bin/activate && exec /root/rl-swarm/run_rl_swarm.sh'
Restart=always
SuccessExitStatus=0
RestartSec=10s
StartLimitIntervalSec=5min
StartLimitBurst=10
KillMode=process
TimeoutStopSec=30
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF
```

### 7. **Enable and start the service**
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now rl_swarm.service
```

---

## 📊 Service Management

### **Check service status**
```bash
sudo systemctl status rl_swarm.service
```

### **View real-time logs**
```bash
sudo journalctl -u rl_swarm.service -f
```

### **View recent logs**
```bash
sudo journalctl -u rl_swarm.service --since "1 hour ago"
```

### **Stop the service**
```bash
sudo systemctl stop rl_swarm.service
```

### **Start the service**
```bash
sudo systemctl start rl_swarm.service
```

### **Restart the service**
```bash
sudo systemctl restart rl_swarm.service
```

### **Disable auto-start**
```bash
sudo systemctl disable rl_swarm.service
```

---

## 🎯 Configuration Options

### **Node Types:**
- **Math (A):** Standard difficulty, suitable for most hardware
- **Math Hard (B):** Higher difficulty, requires more resources

### **Parameter Sizes:**
- **0.5B:** Lightweight, good for CPU or low-end GPU
- **1.5B:** Moderate requirements
- **7B:** Requires decent GPU
- **32B,