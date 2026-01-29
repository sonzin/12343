# Deployment Pattern: Python/JS Financial Dashboard on Ubuntu VPS

This document outlines the standard pattern for deploying lightweight financial tracking tools (like the Vietnam Stock Tracker) to an Ubuntu Virtual Private Server (VPS). It emphasizes data resilience, security, and automated deployment.

## 🏗️ Production Architecture

The deployment follows a professional multi-tier architecture on a single node:
1.  **Entry Point**: Nginx acting as a reverse proxy and static file server.
2.  **Application Layer**: Python API server managed by `systemd`.
3.  **Data Layer**: Client-side storage (localStorage) for user-specific settings and portfolio data.

## 🔧 VPS Provisioning Checklist

### 1. System Dependencies
The following packages are essential for a Python-based finance backend:
- `python3`, `python3-pip`, `python3-venv`: Core runtime.
- `nginx`: Web server and proxy.
- `certbot`, `python3-certbot-nginx`: SSL/TLS certificate management.
- `git`, `curl`: Utility tools for code management and health checks.

### 4. Resource Intensive Libraries
Note that the `vnstock` library has heavy dependencies:
- **Major Deps**: `pandas`, `numpy`, `matplotlib`, `seaborn`, `pydantic`.
- **VPS Impact**: Installation can take 5-10 minutes on a 1vCPU machine. 
- **Latency & Warnings**: During service startup, `vnstock` may log warnings (e.g., `Could not patch KBS Finance`). On low-resource VPS nodes (1GB RAM), the initial import of `pandas/numpy` can take 5-10 seconds, causing Nginx `proxy_read_timeout` errors on the first user request.
- **Architectural Shift (Fast Edition)**: To eliminate this latency, dashboards can be refactored into a "Fast Edition" that uses the standard library `urllib` to fetch JSON from broker APIs (SSI, Cafef) directly, bypassing heavy library dependencies entirely.
- **Requirement**: Ensure the VPS has at least 1GB of RAM. If using `vnstock`, 2GB is recommended for stability.

### 2. Secure User Management
Never run application services as `root`.
```bash
# Create a restricted system user
useradd -r -s /bin/false stocktracker
# Create application directory
mkdir -p /opt/stock-tracker
chown -R stocktracker:stocktracker /opt/stock-tracker
```

### 3. Backend Persistence (Systemd)
Configure a service unit to ensure high availability.
- **Restart Policy**: `always` with a `RestartSec=10` delay.
- **Logging**: Redirect `StandardOutput` and `StandardError` to dedicated log files in `/opt/stock-tracker/logs/`.

## 🌐 Nginx Configuration Pattern

For a hybrid JS/Python app, use a "Same-Origin" proxy pattern to bypass CORS:
- **Root (`/`)**: Serves the static HTML/CSS/JS files.
- **Proxy (`/api/`)**: Forwards requests to the local Python server (e.g., `127.0.0.1:8888`).

```nginx
server {
    listen 80;
    server_name your-domain.com;

    root /opt/stock-tracker/static;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8888;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 💡 Sub-domain vs Sub-path
While the above uses `/api/` (sub-path), deploying to a sub-domain (e.g., `api.your-domain.com`) is often cleaner for complex cookie management, though the sub-path approach avoids additional SSL certificate requirements if using simpler setups.

### 🛡️ Coexisting with Control Panels (e.g., FastPanel)
If deploying to a VPS with a pre-installed control panel:
- **Default Conflicts**: Many panels install a `default` site that captures all traffic on port 80. Remove `/etc/nginx/sites-enabled/default` to allow your custom configuration to take over.
- **Config Location**: Panels like FastPanel may use `/etc/nginx/conf.d/` or `/etc/nginx/fastpanel2-sites/`. Ensure your manually added config does not conflict with panel-managed sites.
- **Port Matching**: Ensure the panel's internal services aren't already using your chosen backend port. 
    - **Critical Example**: FastPanel often uses `8888` for `fastpanel2-ngin`. If this occurs, remap the application port (e.g., to **9797**) to prevent service startup failure.
- **The "Strange Port" Strategy**: If ports 80, 443, or 8080 are heavily managed or redirected (e.g., by FastPanel's parking page), use a high-numbered non-standard port (e.g., `listen 9696;`) for the tool's entrance. This allows the tool to run as a standalone service accessible via `http://IP:9696` without interfering with existing websites managed by the panel. This strategy was successfully validated using **9696 (Frontend)** and **9797 (Backend)** on a FastPanel-enabled Ubuntu VPS.

### 🔍 Port Conflict Diagnostics
If a service fails to start or Nginx returns a 302/404/502 error, check if the port is already occupied:
- **Missing netstat**: Many minimal Ubuntu images lack `netstat`. Use `ss` instead.
- **Command**: `ss -tlnp | grep 8888`
- **Output Analysis**: Look for process names like `fastpanel2-ngin` or `nginx: worker process`. If the port is bound to `0.0.0.0` or the server's public IP, it will capture external traffic even if Nginx is configured otherwise.

### ⚙️ Programmatic Port Remapping
When ports must be changed in production, use `sed` to update configurations and source code without manual editing:
- **Source Code Update**: `sed -i 's/OLD_PORT/NEW_PORT/g' app.py`
- **Config Example**: `sed -i "s|listen 8080|listen 9696|g" /etc/nginx/conf.d/tool.conf`

## 🚀 Automated Deployment (CI/CD)

For small-scale projects, a "Push-to-Deploy" shell script using `scp` and `ssh` is often more efficient than a full CI/CD pipeline.

### The `sshpass` Pattern
`sshpass` allows non-interactive password authentication for scripts (often used when SSH keys aren't yet provisioned).
- **Security Warning**: Only use in controlled environments and avoid hardcoding passwords in scripts; use environment variables or secret managers.
- **Troubleshooting Special Characters**: If the password contains special characters like `@`, `$`, or `!`, ensure it is properly **single-quoted** in shell commands to prevent shell expansion.
    - **Usage**: `sshpass -p 'YourPassword@' ssh -o StrictHostKeyChecking=no -p 8686 root@IP`
    - **Note**: Omission of closing special characters (e.g., `@`) is a frequent cause of `Permission denied (5)` errors.
- **Custom SSH Ports**: Many VPS providers (like the current one at `103.152.164.242`) use non-standard ports (e.g., **8686**) for security. Always verify the port before initiating `rsync` or `ssh`.
    - **Rsync command**: `rsync -avz -e "ssh -p 8686" [LOCAL_PATH] root@IP:[REMOTE_PATH]`
- **Interrupted Syncs (SIGINT)**: If an `rsync` or `ssh` command hangs or is interrupted (Exit Code 20), it may be due to terminal session timeouts or manual cancellation. Ensure the connection is stable or use `nohup`/`screen` for large transfers.
- **Robust Full Sync Pattern**: Instead of syncing individual folders, use a comprehensive `rsync` with exclusions for build artifacts, virtual environments, and sensitive metadata.
    - **Command**: `rsync -avz --progress -e "ssh -p 8686" --exclude '.git' --exclude '__pycache__' --exclude '.venv' --exclude 'venv' --exclude 'node_modules' --exclude '.DS_Store' --exclude 'reports/*.html' ./ root@<VPS_IP>:/root/stock-tracker/`
- **SSH Key Pattern (Recommended)**: To avoid "Password Fatigue" (entering passwords for every `rsync` call), use SSH keys to establish a trust relationship.
    - **Usage**: `ssh-copy-id -p 8686 root@<VPS_IP>`
    - **Benefit**: Subsequent commands like `rsync` or `ssh` will connect instantly/passwordless. This is essential for automated cron-update scripts or rapid iterative debugging.

### Atomic Updates
1.  **Build/Package**: Prepare static assets locally.
2.  **Transfer**: Use `scp` to upload the API server and the static directory.
3.  **Config Refinement**: Use `sed` to update API URLs in JS files from `localhost` to relative production paths (`/api`).
    - **Command**: `sed -i "s|http://localhost:8888||g" /opt/stock-tracker/static/realtime.js`
    - **Effect**: This converts `fetch('http://localhost:8888/api/...')` to `fetch('/api/...')`, allowing the browser to use the relative path served by Nginx.
4.  **Firewall Management**: Open the entrance port (Nginx listener):
    - `ufw allow 9696/tcp`
    - `iptables -I INPUT -p tcp --dport 9696 -j ACCEPT`
5.  **Restart & Verify**: Use `ssh` to reload the `systemd` service and Nginx, then check for the new listener:
    - `systemctl restart stock-tracker && nginx -t && systemctl reload nginx`
    - `ss -tlnp | grep 9797` (Verify backend)
    - `ss -tlnp | grep 9696` (Verify entrance)

### 💀 Handling OSError: [Errno 98] Address already in use
If the backend fails to start despite remapping the port, a "zombie" process or a hidden system service may still be holding the port.
- **Diagnosis**: `ss -tlnp | grep [PORT]` to find the PID.
- **Remedy 1**: `systemctl stop [SERVICE-NAME]` if managed by systemd.
- **Remedy 2**: `pkill -f [SCRIPT-NAME].py` to force-kill Python processes.
- **Remedy 3**: Increment the port number (e.g., 9898 -> 9797) across both the Python script and the Nginx `proxy_pass` configuration.

## 📈 Data Resilience during After-Hours

The backend must be designed to handle the 404/502 errors returned by brokers (like TCBS) when the market is closed.
- **Pattern**: Implement a fallback to a secondary source (like `VCI` history) to retrieve the "latest known" close price, ensuring the dashboard remains informative 24/7.
- **Caching**: Implement a simple TTL cache (e.g., 30s for prices, 1h for company info) to stay within broker rate limits.

### 3. DNS Resolution & Reliability
Even with firewall and protocol (IPv4) fixes, API requests may hang or fail if the VPS's default DNS is slow or unreliable (leading to `urllib.error.URLError: <urlopen error [Errno -2] Name or service not known>`).
- **Diagnosis**: Check `/etc/resolv.conf`. If it uses unknown internal IPs or is empty, DNS is likely the bottleneck.
- **Remedy**: Update to reliable public DNS servers (Google/Cloudflare). In some environments (like FastPanel), the system may revert `/etc/resolv.conf`; ensure it's made immutable or configured via the control panel if possible. 
```bash
# Force Google and Cloudflare DNS
echo 'nameserver 8.8.8.8' > /etc/resolv.conf
echo 'nameserver 1.1.1.1' >> /etc/resolv.conf
```
- **Verification**: Run `nslookup s.cafef.vn` or `ping -4 s.cafef.vn` (Force IPv4) to ensure sub-second resolution. Slow DNS is a primary cause of 500/Timeout errors in financial proxies.

## 🛠️ Production API Server Optimization
For a stable VPS deployment, the Python backend should:
1. **Bind to All Interfaces**: Use `HOST='0.0.0.0'` to allow Nginx to connect from the external network bridge if needed (though `127.0.0.1` is preferred for security).
2. **Logging**: Use the standard `logging` library to record requests and errors to files for easy debugging.
3. **Flexible SSL (Context Bypass)**: Since VPS nodes often have outdated CA certs or restricted outbound proxying, creating a custom `ssl.CERT_NONE` context is often necessary to prevent `urllib.error.URLError` during data fetching.
4. **Environment Config**: Use `os.environ` to manage sensitive configuration like ports and cache durations.

## 🕰️ Background Command Performance
When managing a VPS via automated scripts or remote shells (like Gemini's shell tools):
- **Hang Management**: Long-running `curl` commands or API polls that "hang" (running for 2m+) should be terminated and reviewed for timeout settings (`timeout=10` in Python).
- **Diagnostics**: Use `ss -tlnp | grep python` to check for multiple running instances. If an old process is "stuck," it will prevent the new systemd service from binding correctly.
- **Vercel Integration**: To enable cloud reports (`REPORT_BASE_URL`), the environment requires Vercel CLI.
  - **VPS Installation**: If deploying reports directly from the VPS, install Node.js/NPM and then the CLI:
    ```bash
    curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
    sudo apt-get install -y nodejs
    npm install -g vercel
    ```
  - **Account**: `quangsoncenstaf-2994`
  - **Login**: `vercel login`
- **Remote Interaction**: For tools that provide background command IDs, use tools like `command_status` and `send_command_input` to monitor and gracefully exit hung processes without crashing the entire deployment flow.

## 🌐 Outbound Connectivity Diagnostics
A persistent challenge in financial tool deployment is the VPS's ability to reach external broker APIs (SSI, TCBS, Cafef).

### 1. Identifying unreachable endpoints
If the dashboard shows "online" but all data fields are empty/error, the VPS likely has outbound restrictions:
- **Test**: `ping -c 2 google.com`
- **Error: Address unreachable**: Often occurs on low-cost VPS where IPv6 is prioritized but misconfigured or blocked by the destination broker.
    - **Specific Signal**: `From [IP] icmp_seq=1 Destination unreachable: Address unreachable`.
- **Remedy**: Force IPv4 for critical data fetches if IPv6 is unstable. 
- **Test (CURL)**: `curl -4 -s https://s.cafef.vn/` to verify if IPv4 connectivity is working while the default (which might be IPv6) is failing.
- **Python Remedy**: When using `urllib`, if the system defaults to a broken IPv6 route, you may need to force IPv4 at the socket level or use a library that supports explicit IP family selection.

### 2. Header-Based Blockage
Some Vietnamese financial sources block "headless" user agents. If `ping` works but `curl` return nothing:
- **Check**: Use `-v` to see the HTTP response code (e.g., `403 Forbidden`).
- **Standard Header**: Ensure every request includes a production-grade `User-Agent`.
- **Latency Check**: Use `time curl ...` to measure if the network latency is exceeding the application's internal `timeout` settings (usually 10-15s).

## 🔄 Developer Workflow: Local ↔ VPS Synchronization

Managing a project that runs on a VPS but is developed locally requires a smooth synchronization workflow.

### 1. Initial/Bulk Sync using `rsync`
For the first time you pull code from the VPS or when you need to sync many files (like the initial setup of a local development environment), `rsync` is the most efficient tool.

```bash
# Pull from VPS to Local
rsync -avzP root@<VPS_IP>:~/path/to/project/ /local/path/to/project/

# Push from Local to VPS
rsync -avzP /local/path/to/project/ root@<VPS_IP>:~/path/to/project/
```
*   **Flags**: `-avzP` ensures archive mode (a), verbose (v), compression (z), and progress display (P).
*   **Trailing Slash**: A trailing `/` on the source path syncs the *contents* of the directory rather than the directory itself.

### 2. Continuous Sync with VS Code (SFTP Extension)
The **"SFTP" extension (by Natizyskunk)** allows for an "Edit-Save-Deploy" loop where files are automatically uploaded to the VPS on save.

**Configuration (`.vscode/sftp.json`):**
Create this file in your local project root.

```json
{
    "name": "VPS-Stock-Tracker",
    "host": "<VPS_IP>",
    "protocol": "sftp",
    "port": 22,
    "username": "root",
    "password": "YourPassword@",
    "remotePath": "/root/stock-tracker",
    "uploadOnSave": true,
    "useTempFile": false,
    "openSsh": false,
    "ignore": [
        ".vscode",
        ".git",
        ".DS_Store",
        "__pycache__",
        "venv"
    ]
}
```

*   **Security Note**: Do not commit this file to public repositories if it contains plain-text passwords.
*   **Password Characters**: Characters like `@` inside the JSON string are generally handled well, but ensure the `remotePath` is absolute and existing.

### 3. Verification after Sync
After syncing, verify that local changes are reflected on the server by checking the file modification time:
`ls -l /root/stock-tracker/app.py`

### 🛠️ Troubleshooting Connection Issues

#### Error: `ECONNREFUSED 103.x.x.x:22`
This error means the computer cannot connect to the VPS on the default SSH port (22).
- **Cause**: The VPS likely has been configured to use a non-standard SSH port for security (e.g., `8686`).
- **Solution**: Check your SSH connection settings (e.g., in Termius or your SSH config). Update the `"port"` field in `.vscode/sftp.json` to the correct value.
  ```json
  "port": 8686,
  ```

#### Missing SSH Config
If the SFTP extension complains about a missing `/Users/sonho/.ssh/config`, it is usually a warning and not a fatal error, provided the `host`, `username`, and `password` are correctly specified in `sftp.json`.

## 🖥️ Project Execution Patterns (CLI)

For command-line tools and backtesters, execution typically involves activating a virtual environment and running specific entry point scripts.

### 1. Environment Activation
The standard environment for the Stock Tracker project on VPS is `stock-venv` located in the root.

```bash
# Standard activation sequence
source /root/stock-venv/bin/activate
cd /root/stock-tracker
```

### 2. Execution Entry Points

#### Interactive Mode
Use this for manual data exploration and interactive menus.
```bash
python cli_interactive.py
```

#### Task-Specific Mode
Use the core `cli.py` for automated or one-off tasks.
```bash
# List portfolio
python cli.py portfolio list

# Run market scan
python cli.py scan
```

### 3. Automated Wrapper Scripts
To facilitate cron jobs or easy execution without manual activation, use shell wrapper scripts.

*   **`run_scanner.sh`**: Executes the market scanner.
*   **`run_portfolio.sh`**: Performs portfolio analysis and logs results.

### 4. Real-Time Monitor Crontab (Silent Watcher)
For high-frequency alerting without resource bloat, use separate cron entries for morning and afternoon market sessions.

```cron
# Stock Price Alert Monitor (Every 1 minute during trading hours)
# Monday - Friday | Morning Session: 09:00 - 11:30
# 9:00 - 10:59
*/1 9-10 * * 1-5 python3 /root/stock-tracker/scripts/price_alert.py
# 11:00 - 11:30 (Checks minutes 0-30)
*/1 11 * * 1-5 [ $(date +\%M) -le 30 ] && python3 /root/stock-tracker/scripts/price_alert.py

# Monday - Friday | Afternoon Session: 13:00 - 15:00
# 13:00 - 14:59
*/1 13-14 * * 1-5 python3 /root/stock-tracker/scripts/price_alert.py
# 15:00 Exactly (Closing sync)
0 15 * * 1-5 python3 /root/stock-tracker/scripts/price_alert.py
```

*   **Silent Mode**: The script should be designed to produce zero output (stdout/stderr) unless an alert is triggered. This prevents the VPS `/var/spool/mail/root` or log files from filling up with "No alert" messages.
*   **Time Sensitivity**: Ensure the VPS system time (`date`) matches the stock market timezone (GMT+7) or adjust cron offsets accordingly.

### 4. Critical Dependencies
The project relies on `vnstock_data` Bronze edition, which requires a specific CLI installer:
```bash
wget https://vnstocks.com/files/vnstock-cli-installer.run
chmod +x vnstock-cli-installer.run
./vnstock-cli-installer.run
```
(Manual API key input is required during the first installation).

### 🛠️ Runtime Debugging & Data Safety

#### Error: `TypeError: argument of type 'NoneType' is not iterable` or `NoneType object is not callable`
In financial tools, this frequently occurs when a broker API returns an empty response or a `None` field during market "Blackout" periods (e.g., 8:45 AM or 3:01 PM).

- **Cause**: Attempting to iterate (e.g., `for item in data`) over a variable that failed to fetch and returned `None`.
- **Solution - The "Safety Wrap"**: Always initialize lists and check for truthiness.
  ```python
  # Safe pattern
  data = api.fetch() or []  # Ensures data is at least an empty list
  for item in data:
      # process
  ```
- **Formatting Trap**: Formatting `None` with `:+.2f` in f-strings:
  ```python
  # Crashes if val is None
  msg = f"Change: {val:+.2f}%" 
  
  # Safe alternative
  msg = f"Change: {val or 0:+.2f}%"
  ```
- **Diagnostics**: If the error occurs in a complex orchestrator (like `PortfolioAnalyzer`), check the return values of `_fetch_market_data()` or `load_strategy_context()`. If these return `None` instead of a dictionary/list, the template generation will fail.
### 📉 API Rate Limit Management (vnstock Bronze tier)

When running automated scanners (like `daily_scanner.py`) on a VPS using the **vnstock Bronze tier**, you are limited to **180 requests per minute**. Exceeding this limit results in a `Rate limit exceeded` error and process termination.

- **The Problem**: A high-speed loop scanning 50+ symbols with multiple technical indicator requests per symbol will easily hit the 180 req/min ceiling.
- **The Recovery Message**: 
  ```text
  Rate limit exceeded.
  Wait 32 seconds to continue (Wait to retry)
  Process terminated.
  ```
- **Optimization Strategy**: Implement batching with mandatory "cool-down" periods.
  ```python
  # Optimized Rate Limiting for Bronze Tier
  for i, sym in enumerate(symbols):
      # ... scanning logic ...
      
      # Batch Size: 20 symbols
      # Sleep: 35 seconds
      if i > 0 and i % 20 == 0:
          print(f"    [!] Rate limit pause (35s) to avoid API limit...")
          time.sleep(35)
  ```
- **Guidelines**:
    - **Batch Size**: 20 symbols is a safe threshold for a script performing 5-8 API calls per symbol.
    - **Sleep Duration**: 35-45 seconds ensures the "rolling window" of the API broker resets.

### 🚀 Remote Diagnostics & Verification

When debugging issues that only manifest on the VPS (e.g., environment differences), use a targeted "Checker Script" pattern.

1.  **Direct SSH Command**: Run specific Python logic directly on the VPS to verify paths and library availability without launching the full CLI.
    ```bash
    # Check if a file exists on the VPS
    ssh -p 8686 root@<VPS_IP> "[ -f /root/stock-tracker/data/strategy/strategy_2026.md ] && echo 'Found' || echo 'Not Found'"
    
    # Run a diagnostic check (assuming a helper script is uploaded)
    ssh -p 8686 root@<VPS_IP> "python3 /root/stock-tracker/tools/checker_none.py"
    ```
2.  **Environment Sync**: If a fix is applied locally, ensure the SFTP synchronization is complete before re-running verification.
3.  **Process Monitoring**: If a background process (like a bot or scanner) hangs, use `ss -tlnp` or `ps aux | grep python` via SSH to identify and restart it.

### 💀 The "Missing Package" Trap
When deploying standalone scripts (e.g., in the `scripts/` folder) that import from internal project folders (e.g., `from crawler import UnifiedCrawler`), uploading only the script is NOT enough.
- **Symptom**: `ModuleNotFoundError: No module named 'crawler'` when running the script on VPS.
- **Cause**: The `crawler/` package directory exists locally but was omitted during the `rsync` of individual files.
- **Solution**: Always upload the required package folders along with the scripts. The most common cause of failure is omitting the `crawler/` or `src/` directories.
    ```bash
    # Recursive upload ensuring all dependencies are present
    rsync -avz -e "ssh -p 8686" scripts/ crawler/ src/ data/ root@IP:/root/stock-tracker/
    ```
- **PYTHONPATH Check**: Ensure the script correctly calculates the `PROJECT_ROOT` and inserts it into `sys.path`.

### 🏆 Verified Success Case: Real-Time Monitoring
The deployment of the `price_alert.py` monitor on an Ubuntu VPS demonstrated the following results:
- **Port Strategy**: Successfully used **Port 8686** for `rsync` and `ssh` after default Port 22 was refused.
- **Data Accuracy**: Prioritizing **KBS Intraday Trades** allowed the system to detect the live ACB price of **23,900 VND** during the 14:45 session, correctly triggering a `-4.4%` warning alert when other sources (like Cafef) were returning stale data (24,700 VND).
- **Spam Suppression**: The monitor verified that `data/alert_state.json` successfully prevented duplicate alerts for the same symbol within the same trading session.
- **Automation**: The crontab configuration (detailed in section 4) successfully scheduled the monitor to run every minute only during active market hours, conserving VPS resources during lunch breaks and overnight.
- **Full Project Sync**: Verified the "Robust Full Sync" strategy by synchronizing the entire codebase (including `crawler/`, `src/`, `data/`, and `scripts/`). This resolved the `ModuleNotFoundError` for internal packages and prepared the VPS for remote execution of heavy AI analysis scripts.
- **Vercel CLI Setup**: Successfully installed Vercel CLI (v50.8.1) and authenticated via the `--device` flow. This enables the server to deploy HTML reports directly to the cloud, mirroring the local development workflow.
- **Environmental Parity**: By synchronizing the entire codebase and installing identical CLI tools (Vercel, Node, Python dependencies), the VPS now functions as a high-availability mirror of the local environment, capable of running heavy analytical tasks and automated alerting without environment-specific friction.
