# LMArena Proxy - macOS Setup Guide

Complete step-by-step guide for setting up and running the LMArena Proxy on macOS.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Starting the Proxy Server](#starting-the-proxy-server)
- [Installing the Tampermonkey Userscript](#installing-the-tampermonkey-userscript)
- [Verifying the Connection](#verifying-the-connection)
- [Accessing the Monitoring Dashboard](#accessing-the-monitoring-dashboard)
- [Using the API](#using-the-api)
- [Troubleshooting](#troubleshooting)
- [Advanced Configuration](#advanced-configuration)

---

## Prerequisites

Before you begin, ensure you have the following installed on your macOS system:

### 1. Python 3.8 or Higher

Check your Python version:
```bash
python3 --version
```

If you don't have Python 3.8+, install it using Homebrew:
```bash
# Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Python
brew install python@3.11
```

### 2. Web Browser

You'll need one of the following browsers:
- **Google Chrome** (recommended)
- **Microsoft Edge**
- **Mozilla Firefox**

### 3. Tampermonkey Extension

Install the Tampermonkey browser extension:
- Chrome/Edge: [Chrome Web Store](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo)
- Firefox: [Firefox Add-ons](https://addons.mozilla.org/en-US/firefox/addon/tampermonkey/)

---

## Installation

### Step 1: Clone the Repository

Open Terminal and clone the project:
```bash
git clone https://github.com/zhongruichen/lmarena-proxy.git
cd lmarena-proxy
```

### Step 2: Install Python Dependencies

Install the required Python packages from `requirements.txt`:
```bash
pip3 install -r requirements.txt
```

The following packages will be installed:
- `fastapi` - Web framework for the proxy server
- `uvicorn` - ASGI server
- `websockets` - WebSocket support
- `aiohttp` - Async HTTP client
- `prometheus-client` - Metrics collection

Alternatively, you can create a virtual environment (recommended):
```bash
# Create virtual environment
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

---

## Starting the Proxy Server

### Method 1: Using the Start Script (Recommended)

The repository includes a convenient startup script (`start.sh`) that automatically:
- Starts the Python proxy server
- Opens LMArena website in your default browser
- Opens the monitoring dashboard

First, make the script executable:
```bash
chmod +x start.sh
```

Then run it:
```bash
./start.sh
```

### Method 2: Manual Start

If you prefer to start the server manually:
```bash
python3 proxy_server.py
```

### What Happens When the Server Starts

When the server starts successfully, you'll see output like:
```
🌐 Server access URLs:
  - Local: http://localhost:9080
  - Network: http://192.168.1.100:9080
📱 Use the Network URL to access from your phone on the same WiFi

📋 Available Endpoints:
  🖥️  Monitor Dashboard: http://localhost:9080/monitor
  📊 Metrics & Health:
     - Prometheus Metrics: http://localhost:9080/metrics
     - Health Check: http://localhost:9080/health
     - Detailed Health: http://localhost:9080/api/health/detailed
  🤖 AI API:
     - Chat Completions: POST http://localhost:9080/v1/chat/completions
     - List Models: GET http://localhost:9080/v1/models
```

The server runs on **port 9080** by default.

---

## Installing the Tampermonkey Userscript

The Tampermonkey userscript is the critical component that connects your browser to the proxy server and executes requests on LMArena.

### Method 1: Install from GitHub (Recommended - Auto-updates)

1. Click this direct installation link:
   ```
   https://raw.githubusercontent.com/zhongruichen/lmarena-proxy/main/lmarena_injector.user.js
   ```

2. Tampermonkey will automatically detect the script and show an installation page

3. Click the **"Install"** button

4. The script will be installed and will automatically update when new versions are available

### Method 2: Manual Installation

1. Open the file `lmarena_injector.user.js` in a text editor

2. Select all content (Cmd+A) and copy it (Cmd+C)

3. Click the Tampermonkey extension icon in your browser toolbar

4. Select **"Create a new script"** from the menu

5. Delete all default content in the editor

6. Paste the copied content (Cmd+V)

7. Save the script by pressing **Cmd+S** or clicking File → Save

### Verifying Script Installation

1. Click the Tampermonkey icon in your browser

2. You should see **"LMArena Proxy Injector"** in the list with a green indicator showing it's enabled

---

## Verifying the Connection

After starting the server and installing the userscript, follow these steps to verify everything is working:

### Step 1: Open LMArena Website

Navigate to [https://lmarena.ai/](https://lmarena.ai/) in your browser.

### Step 2: Open Browser Developer Console

Press **F12** or **Cmd+Option+I** to open the Developer Tools, then click on the **Console** tab.

### Step 3: Check for Success Message

If the connection is successful, you'll see a message like:
```
[Injector] ✅ Connection established with local server.
```

If you see connection errors, refer to the [Troubleshooting](#troubleshooting) section.

### Step 4: Check Server Logs

In the Terminal where you started the server, you should see:
```
INFO: Browser connected
INFO: WebSocket /ws opened
```

---

## Accessing the Monitoring Dashboard

The proxy server includes a real-time monitoring dashboard with comprehensive system metrics.

### Opening the Dashboard

1. Open your browser and navigate to:
   ```
   http://localhost:9080/monitor
   ```

2. If the `start.sh` script was used, this page opens automatically

### Dashboard Features

The monitoring dashboard provides:

- **Real-time Statistics**
  - Requests per second (QPS)
  - Average response time
  - Success/error rates
  - Active connections

- **Request Logs**
  - Live request tracking
  - Request ID and status
  - Model used
  - Response times

- **Performance Metrics**
  - P50/P95/P99 response times
  - Token usage statistics
  - Model usage breakdown

- **System Health**
  - Overall health score
  - Connection status
  - Active request count
  - Error rate monitoring

- **Configuration Management**
  - Dynamic IP settings
  - Request timeout configuration
  - Concurrent request limits
  - Network settings

### Other Monitoring Endpoints

- **Basic Health Check**: `http://localhost:9080/health`
- **Detailed Health**: `http://localhost:9080/api/health/detailed`
- **Prometheus Metrics**: `http://localhost:9080/metrics`

---

## Using the API

The proxy server provides an OpenAI-compatible API interface. You can use any OpenAI-compatible client or library.

### Available Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/chat/completions` | POST | OpenAI-compatible chat completions |
| `/v1/models` | GET | List all available models |
| `/v1/refresh-models` | POST | Refresh model list from LMArena |

### Example 1: Using Python OpenAI Library

Install the OpenAI Python library:
```bash
pip install openai
```

Create a file `test_api.py`:
```python
from openai import OpenAI

# Initialize client pointing to your local proxy
client = OpenAI(
    base_url="http://localhost:9080/v1",
    api_key="sk-any-string-you-like"  # API key can be any string
)

# Example: Chat completion with streaming
try:
    response = client.chat.completions.create(
        model="claude-3-5-sonnet-20241022",
        messages=[
            {"role": "user", "content": "Hello! What can you do?"}
        ],
        stream=True
    )

    print("Response from model:")
    for chunk in response:
        content = chunk.choices[0].delta.content
        if content:
            print(content, end="", flush=True)
    print("\n")

except Exception as e:
    print(f"An error occurred: {e}")
```

Run the script:
```bash
python3 test_api.py
```

### Example 2: Using cURL

#### Chat Completion (Streaming)
```bash
curl http://localhost:9080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-any-key" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "messages": [{"role": "user", "content": "Hello!"}],
    "stream": true
  }'
```

#### Chat Completion (Non-streaming)
```bash
curl http://localhost:9080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-any-key" \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "Hello!"}],
    "stream": false
  }'
```

#### List All Available Models
```bash
curl http://localhost:9080/v1/models
```

### Example 3: Image Generation

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:9080/v1",
    api_key="sk-any-string"
)

response = client.chat.completions.create(
    model="dall-e-3",  # or "flux-1.1-pro", "imagen-3.0-generate-002", etc.
    messages=[
        {"role": "user", "content": "A beautiful sunset over mountains"}
    ],
    stream=False
)

# The response contains markdown-formatted image URLs
print(response.choices[0].message.content)
```

### Example 4: Video Generation

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:9080/v1",
    api_key="sk-any-string"
)

response = client.chat.completions.create(
    model="pika-2.2",  # or "veo3", "kling-2.1-master", etc.
    messages=[
        {"role": "user", "content": "A cat playing with a ball"}
    ],
    stream=False
)

# The response contains video URLs
print(response.choices[0].message.content)
```

### Supported Models

The proxy supports 100+ models from LMArena, including:

**Chat Models:**
- Claude (claude-3-5-sonnet-20241022, claude-3-7-sonnet-20250219, etc.)
- GPT (gpt-4o, gpt-4-turbo, gpt-3.5-turbo, etc.)
- Gemini (gemini-2.0-flash-exp, gemini-exp-1206, etc.)
- DeepSeek (deepseek-chat-v3, deepseek-reasoner, etc.)
- And many more...

**Image Models:**
- DALL-E 3
- Flux (flux-1.1-pro, flux-1-kontext-pro, etc.)
- Imagen (imagen-3.0-generate-002, imagen-4.0-ultra-generate-preview-06-06)
- Recraft v3
- And more...

**Video Models:**
- Pika 2.2
- Veo3, Veo2
- Kling 2.1 Master
- Hailuo
- And more...

To get the complete, up-to-date list of models:
```bash
curl http://localhost:9080/v1/models
```

Or refresh from LMArena:
```bash
curl -X POST http://localhost:9080/v1/refresh-models
```

---

## Troubleshooting

### Issue 1: Browser Script Cannot Connect to Server

**Symptoms:**
- Console shows connection errors
- No `✅ Connection established` message

**Solutions:**

1. **Verify the server is running:**
   ```bash
   # Check if the process is running
   ps aux | grep proxy_server
   
   # Check if port 9080 is listening
   lsof -i :9080
   ```

2. **Check firewall settings:**
   ```bash
   # Temporarily disable firewall for testing
   sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate off
   
   # Re-enable after testing
   sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on
   ```
   
   Or allow the specific app:
   - Go to System Preferences → Security & Privacy → Firewall → Firewall Options
   - Add Python to the allowed applications list

3. **Verify the userscript is running:**
   - Click Tampermonkey icon → Dashboard
   - Ensure "LMArena Proxy Injector" is enabled
   - Check that it's set to run on `https://*.lmarena.ai/*`

4. **Check WebSocket URL configuration:**
   - Open Tampermonkey → Dashboard
   - Edit "LMArena Proxy Injector"
   - Verify the `CONFIG` section shows:
     ```javascript
     const CONFIG = {
         SERVER_URL: "ws://localhost:9080/ws",
     };
     ```

### Issue 2: API Requests Timeout or Fail

**Symptoms:**
- Requests return timeout errors
- No response from the API

**Solutions:**

1. **Check server logs:**
   - Look at Terminal output for errors
   - Check `logs/server.log` for detailed logs
   - Check `logs/errors.jsonl` for error messages

2. **Verify LMArena website access:**
   - Manually test if you can use models on [https://lmarena.ai/](https://lmarena.ai/)
   - Ensure you're logged in to LMArena
   - Complete any Cloudflare challenges if they appear

3. **Check browser tab is open:**
   - The LMArena tab must remain open in your browser
   - The userscript needs an active page to operate

4. **Refresh the LMArena page:**
   - Press Cmd+R to refresh
   - Check console for reconnection messages

5. **Increase timeout settings:**
   - Go to http://localhost:9080/monitor
   - Click "System Settings"
   - Increase "Request Timeout" value
   - Save changes

### Issue 3: Model List Is Empty

**Symptoms:**
- `/v1/models` returns empty list
- API returns "model not found" errors

**Solutions:**

1. **Ensure you're logged in to LMArena:**
   - Visit https://lmarena.ai/
   - Sign in with your account
   - Refresh the page

2. **Manually refresh models:**
   ```bash
   curl -X POST http://localhost:9080/v1/refresh-models
   ```
   
   Or use the monitoring dashboard:
   - Go to http://localhost:9080/monitor
   - Look for "Refresh Models" button

3. **Check browser console:**
   - Open Developer Tools (F12)
   - Look for errors related to model detection

### Issue 4: Port Already in Use

**Symptoms:**
- Error: "Address already in use" when starting server

**Solutions:**

1. **Find process using port 9080:**
   ```bash
   lsof -i :9080
   ```

2. **Kill the existing process:**
   ```bash
   kill -9 <PID>
   ```
   Replace `<PID>` with the process ID from the previous command

3. **Or use a different port:**
   Edit `proxy_server.py` and change the `PORT` value in the `Config` class, or use the configuration API after starting on a different port.

### Issue 5: "Module not found" Error

**Symptoms:**
- Python errors about missing modules

**Solutions:**

1. **Ensure all dependencies are installed:**
   ```bash
   pip3 install -r requirements.txt
   ```

2. **Use virtual environment:**
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   python proxy_server.py
   ```

### Issue 6: Cloudflare Challenges

**Symptoms:**
- Requests fail with Cloudflare errors
- Server logs show authentication issues

**Solutions:**

1. **Complete challenges manually:**
   - Go to https://lmarena.ai/ in your browser
   - Complete any CAPTCHA or security checks
   - Wait for the page to fully load

2. **Keep browser tab active:**
   - Don't close or minimize the LMArena tab
   - The userscript needs to be running

3. **Try refreshing:**
   - Refresh the LMArena page
   - Wait a few seconds for reconnection

### Issue 7: Server Won't Start - Permission Denied

**Symptoms:**
- Cannot execute `start.sh`
- "Permission denied" error

**Solutions:**

1. **Make script executable:**
   ```bash
   chmod +x start.sh
   ```

2. **Or run with bash directly:**
   ```bash
   bash start.sh
   ```

---

## Advanced Configuration

### Running on a Different Machine

If you want to run the proxy server on one machine and access it from another (e.g., your phone or another computer):

1. **Find your server's IP address:**
   ```bash
   ipconfig getifaddr en0  # For WiFi
   # or
   ipconfig getifaddr en1  # For Ethernet
   ```

2. **Update the userscript configuration:**
   - Click Tampermonkey icon → Dashboard
   - Edit "LMArena Proxy Injector"
   - Change the `CONFIG` section:
     ```javascript
     const CONFIG = {
         SERVER_URL: "ws://192.168.1.100:9080/ws",  // Use your server's IP
     };
     ```
   - Save the script

3. **Ensure firewall allows connections:**
   - Add an inbound rule for port 9080
   - Or disable firewall for testing (not recommended for production)

### Dynamic Configuration via Dashboard

You can modify many settings without restarting the server:

1. Go to http://localhost:9080/monitor
2. Click "System Settings"
3. Available settings:
   - **Network Settings**: IP detection mode (auto/manual)
   - **Request Settings**: Timeout, max concurrent requests
   - **Monitoring Settings**: Error thresholds, alert levels

4. Click "Save" to apply changes

Configuration is saved to `logs/config.json` and persists across restarts.

### Log Files

The server creates several log files in the `logs/` directory:

- `server.log` - General server activity
- `requests.jsonl` - Request/response details (JSONL format)
- `errors.jsonl` - Error logs (JSONL format)
- `config.json` - Server configuration

These files automatically rotate when they reach 50MB, keeping the 50 most recent files.

### Environment Variables

You can set environment variables to customize behavior:

```bash
# Example: Run on a different port
export PORT=8080
python3 proxy_server.py
```

### Using with Virtual Environment

For better dependency isolation:

```bash
# Create virtual environment
python3 -m venv venv

# Activate it
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run server
python proxy_server.py

# When done, deactivate
deactivate
```

### Integrating with System Services (Optional)

To run the proxy as a background service that starts automatically:

1. Create a LaunchAgent plist file at `~/Library/LaunchAgents/com.lmarena.proxy.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.lmarena.proxy</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/python3</string>
        <string>/path/to/lmarena-proxy/proxy_server.py</string>
    </array>
    <key>WorkingDirectory</key>
    <string>/path/to/lmarena-proxy</string>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/path/to/lmarena-proxy/logs/stdout.log</string>
    <key>StandardErrorPath</key>
    <string>/path/to/lmarena-proxy/logs/stderr.log</string>
</dict>
</plist>
```

2. Load the service:
```bash
launchctl load ~/Library/LaunchAgents/com.lmarena.proxy.plist
```

3. Control the service:
```bash
# Stop
launchctl unload ~/Library/LaunchAgents/com.lmarena.proxy.plist

# Start
launchctl load ~/Library/LaunchAgents/com.lmarena.proxy.plist
```

---

## Quick Reference

### Essential Commands
```bash
# Start server
./start.sh

# Start server manually
python3 proxy_server.py

# Check if server is running
lsof -i :9080

# View live server logs
tail -f logs/server.log

# List available models
curl http://localhost:9080/v1/models

# Test API with simple request
curl http://localhost:9080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "gpt-4o", "messages": [{"role": "user", "content": "Hi"}]}'
```

### Essential URLs
- **LMArena Website**: https://lmarena.ai/
- **Monitoring Dashboard**: http://localhost:9080/monitor
- **API Base URL**: http://localhost:9080/v1
- **Health Check**: http://localhost:9080/health
- **Prometheus Metrics**: http://localhost:9080/metrics

### Essential Files
- `proxy_server.py` - Main server application
- `lmarena_injector.user.js` - Tampermonkey userscript
- `requirements.txt` - Python dependencies
- `start.sh` - Startup script
- `logs/server.log` - Server logs
- `logs/config.json` - Configuration file

---

## Getting Help

If you encounter issues not covered in this guide:

1. **Check the main README**: See [README.md](./README.md) for general information
2. **GitHub Issues**: Report bugs or request features at the [GitHub repository](https://github.com/zhongruichen/lmarena-proxy/issues)
3. **Server Logs**: Check `logs/server.log` and `logs/errors.jsonl` for detailed error information
4. **Browser Console**: Press F12 to check for JavaScript errors in the browser

---

## Notes and Limitations

- **Frontend Dependency**: This proxy relies on LMArena's website structure. If LMArena updates their site significantly, the userscript may need updates.
- **Single Browser Instance**: Only one browser instance should run the userscript connected to each proxy server instance.
- **Active Browser Requirement**: The browser tab with LMArena must remain open and active for the proxy to work.
- **Rate Limits**: You're subject to LMArena's rate limits and terms of service.

---

**Last Updated**: 2025
**Version**: 1.1.0
**License**: MIT
