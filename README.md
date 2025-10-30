# CapSolver: The Ultimate Cloudflare Challenge Solver for Automation

![](https://assets.capsolver.com/prod/posts/how-to-solve-cloudflare-challenge-5s/yzgUfBP8mmrF-8466235683e82af5a6c994fbb69c05b6.jpg)

## 🚀 Overview

This repository provides a practical guide and Python example for automatically bypassing the **Cloudflare Challenge** (including Managed Challenge and JS Challenge/5-second screen) using the **CapSolver** API.

Cloudflare's anti-bot measures, while effective against malicious traffic, frequently block legitimate web scraping and automation tasks. CapSolver offers a robust, AI-powered solution by simulating a real browser environment to generate the necessary `cf_clearance` cookie.

## ✨ Key Advantages

CapSolver is optimized for high-volume, reliable challenge solving compared to traditional methods:

| Feature | CapSolver (Recommended) | Traditional Methods (e.g., Headless Browsers) |
| :--- | :--- | :--- |
| **Success Rate** | Very High (AI models constantly updated) | Low to Moderate (Prone to frequent detection) |
| **Integration** | Simple 2-step API call | Complex setup & configuration |
| **Maintenance** | Zero (Handled by CapSolver team) | High (Requires constant code updates/fingerprint analysis) |
| **Resource Use** | Minimal (Simple HTTP request) | Very High (Requires significant CPU/memory for emulation) |

## 🛠️ Implementation Guide (Python)

Bypassing the Cloudflare Challenge with CapSolver is a simple two-step API process.

### Prerequisites

1.  **CapSolver Account:** Get your API Key from the [CapSolver Dashboard](https://dashboard.capsolver.com/dashboard/overview/?utm_source=github&utm_medium=readme).
2.  **Proxy:** A static or sticky proxy is required for the task.
3.  **TLS-Friendly HTTP Client:** You must use a specialized library (e.g., `curl_cffi` or `requests-tls`) for the final request to the target site.

### Step 1: Create the Task (`AntiCloudflareTask`)

Send a `createTask` request to initiate the solving process.

**Request Payload Example:**

```json
{
  "clientKey": "YOUR_API_KEY",
  "task": {
    "type": "AntiCloudflareTask",
    "websiteURL": "https://www.example-protected-site.com",
    "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/141.0.0.0 Safari/537.36",
    "proxy": "ip:port:user:pass"
  }
}
```

### Step 2: Retrieve the Solution

Poll the `getTaskResult` endpoint using the returned `taskId`. Once the status is `ready`, the response will contain the `solution` object, including the critical `cf_clearance` cookie.

**Solution Response Example:**

```json
{
  "errorId": 0,
  "taskId": "df944101-64ac-468d-bc9f-41baecc3b8ca",
  "status": "ready",
  "solution": {
    "cookies": {
        "cf_clearance": "Bcg6jNLzTVaa3IsFhtDI.e4_LX8p7q7zFYHF7wiHPo...uya1bbdfwBEi3tNNQpc" // <-- Use this cookie
    },
    "token": "Bcg6jNLzTVaa3IsFhtDI.e4_LX8p7q7zFYHF7wiHPo...uya1bbdfwBEi3tNNQpc",
    "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36"
  }
}
```

### Full Python Example

This script demonstrates the end-to-end process.

```python
# pip install requests
import requests
import time

api_key = "YOUR_API_KEY"  # Replace with your CapSolver API key
target_url = "https://www.example-protected-site.com"
proxy_string = "ip:port:user:pass" # Replace with your proxy details

def capsolver_solve_cloudflare():
    # 1. Create Task
    create_task_payload = {
        "clientKey": api_key,
        "task": {
            "type": "AntiCloudflareTask",
            "websiteURL": target_url,
            "proxy": proxy_string
        }
    }
    
    print("Sending task to CapSolver...")
    res = requests.post("https://api.capsolver.com/createTask", json=create_task_payload)
    resp = res.json()
    task_id = resp.get("taskId")
    
    if not task_id:
        print("Task creation failed:", res.text)
        return None
    
    print(f"Got taskId: {task_id}. Polling for result...")

    # 2. Get Result
    while True:
        time.sleep(3)  # Wait 3 seconds before polling
        get_result_payload = {"clientKey": api_key, "taskId": task_id}
        res = requests.post("https://api.capsolver.com/getTaskResult", json=get_result_payload)
        resp = res.json()
        status = resp.get("status")
        
        if status == "ready":
            solution = resp.get("solution", {})
            print("Challenge solved successfully!")
            return solution
        
        if status == "failed" or resp.get("errorId"):
            print("Solve failed! Response:", res.text)
            return None

# --- Main Execution ---
solution = capsolver_solve_cloudflare()

if solution:
    cf_clearance_cookie = solution['cookies']['cf_clearance']
    user_agent = solution['userAgent']
    
    print("\n--- Final Request Details ---")
    print(f"User-Agent to use: {user_agent}")
    print(f"cf_clearance cookie: {cf_clearance_cookie[:20]}...")
    
    # Use the retrieved cookie and User-Agent for the final request
    final_request_headers = {
        'User-Agent': user_agent,
        'Cookie': f'cf_clearance={cf_clearance_cookie}'
    }
    
    # Example final request (Requires a TLS-friendly library and proxy setup)
    # final_response = requests.get(target_url, headers=final_request_headers, proxies={'http': f'http://{proxy_string}'})
    # print(final_response.text)
else:
    print("Failed to get solution.")
```

## 🎁 Bonus Code

Redeem code **CAP25** on the [CapSolver Dashboard](https://dashboard.capsolver.com/dashboard/overview/?utm_source=github&utm_medium=readme) to get an extra 5% bonus on every recharge (unlimited).

## 🔗 Resources

*   [CapSolver Documentation](https://www.capsolver.com/?utm_source=github&utm_medium=readme)
*   [Dashboard Login](https://dashboard.capsolver.com/dashboard/overview/?utm_source=github&utm_medium=readme)
*   [Contact Support](https://www.capsolver.com/contact-us/?utm_source=github&utm_medium=readme)
