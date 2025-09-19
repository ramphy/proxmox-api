# Proxmox VE API: The Missing Manual 📖

[![OpenAPI](https://img.shields.io/badge/OpenAPI-3.0-green.svg)](https://spec.openapis.org/oas/v3.0.0) [![Proxmox VE](https://img.shields.io/badge/Proxmox%20VE-8.x-orange.svg)](https://www.proxmox.com/proxmox-ve) [![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE) [![Documentation](https://img.shields.io/badge/docs-official-brightgreen.svg)](https://pve.proxmox.com/wiki/Proxmox_VE_API) [![File Size](https://img.shields.io/badge/openapi.json-2.1MB-ff69b4.svg)](openapi.json)

> *Because sometimes the official docs are... let's say "minimalist" 🤷‍♂️*

Comprehensive OpenAPI specification for Proxmox Virtual Environment (VE) API. Born out of frustration with scattered documentation and crafted with love (and coffee ☕) to make your life easier.

## 📋 Table of Contents

- [The Star of the Show](#-the-star-of-the-show-openapijson-)
- [What's in the Box?](#-whats-in-the-box)
- [Features](#features)
- [Quick Start (No PhD Required)](#-quick-start-no-phd-required)
- [Installation](#-installation)
- [Authentication](#-authentication)
- [OpenAPI Specification](#openapi-specification)
- [API Endpoints](#api-endpoints)
- [Resources](#-resources)
- [Contributing](#contributing)
- [License](#license)

## ⭐ THE STAR OF THE SHOW: `openapi.json` ⭐

```
🎯 What makes this repo special?
📁 openapi.json (2.1MB of pure API goodness)
📊 Complete Proxmox VE API specification
🛠️ Ready for ANY API tool or code generator
```

> **Fun fact**: This 2.1MB file contains the ENTIRE Proxmox VE API specification. Every endpoint, every parameter, every response. It's basically the Rosetta Stone for Proxmox automation! 🗿

**Why this exists**: Because finding comprehensive Proxmox API docs feels like searching for Wi-Fi in the desert. This `openapi.json` file IS your oasis! 🏜️→🏝️

## 📦 What's in the Box?

This repository contains everything you need to work with the Proxmox VE API:

### 📄 Core Files

| File | Size | Purpose | Best For |
|------|------|---------|----------|
| **[`openapi.json`](openapi.json)** | 2.1MB | Complete API specification | API tools, code generation, comprehensive reference |
| **[`PROXMOX_API_LLM_GUIDE.md`](PROXMOX_API_LLM_GUIDE.md)** | 7KB | Ultra-compact guide for AI/LLMs | AI-assisted development, quick reference |
| **[`Proxmox VE API.postman_collection.json`](Proxmox%20VE%20API.postman_collection.json)** | 2.3MB | Ready-to-import Postman collection | API testing, interactive exploration |

### 🎯 Usage Scenarios

- **🤖 AI/LLM Development**: Use `PROXMOX_API_LLM_GUIDE.md` as context for ChatGPT, Claude, etc.
- **🔧 API Testing**: Import the Postman collection for immediate testing
- **📚 SDK Generation**: Feed `openapi.json` to OpenAPI Generator
- **📖 Documentation**: Browse the complete API reference
- **⚡ Quick Integration**: Copy-paste code patterns from the LLM guide

## ✨ Features

- **Complete OpenAPI 3.0 Specification** - The mythical `openapi.json` with ALL endpoints
- **AI/LLM Optimized Guide** - Ultra-compact reference designed for AI assistants
- **Postman Collection** - Ready-to-import collection for immediate API testing
- **Multiple Formats** - JSON specification + human-readable guide + interactive collection
- **Authentication Examples** - Both token and ticket methods (because choices matter)

## 🚀 Quick Start (No PhD Required)

### Using curl

```bash
# Get authentication ticket
curl -X POST https://your-proxmox-server:8006/api2/json/access/ticket \
  -d "username=root@pam&password=yourpassword"

# List all VMs using the ticket
curl -X GET https://your-proxmox-server:8006/api2/json/nodes/node1/qemu \
  -H "Cookie: PVEAuthCookie=PVE:root@pam:..."
```

### Using Python

```python
import requests
import json

# Disable SSL warnings for self-signed certificates
import urllib3
urllib3.disable_warnings()

# Authentication
def proxmox_auth(host, username, password):
    url = f"https://{host}:8006/api2/json/access/ticket"
    data = {'username': username, 'password': password}

    response = requests.post(url, data=data, verify=False)
    auth_data = response.json()['data']

    return {
        'ticket': auth_data['ticket'],
        'csrf': auth_data['CSRFPreventionToken']
    }

# Example: List VMs
def list_vms(host, node, auth):
    url = f"https://{host}:8006/api2/json/nodes/{node}/qemu"
    headers = {
        'Cookie': f"PVEAuthCookie={auth['ticket']}",
        'CSRFPreventionToken': auth['csrf']
    }

    response = requests.get(url, headers=headers, verify=False)
    return response.json()

# Usage
auth = proxmox_auth('192.168.1.100', 'root@pam', 'yourpassword')
vms = list_vms('192.168.1.100', 'pve', auth)
print(json.dumps(vms, indent=2))
```

### Using JavaScript/Node.js

```javascript
const axios = require('axios');
const https = require('https');

// Ignore self-signed certificate
const agent = new https.Agent({
  rejectUnauthorized: false
});

// Authentication
async function proxmoxAuth(host, username, password) {
  const url = `https://${host}:8006/api2/json/access/ticket`;
  const response = await axios.post(url,
    `username=${username}&password=${password}`,
    {
      httpsAgent: agent,
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' }
    }
  );

  return response.data.data;
}

// List VMs
async function listVMs(host, node, auth) {
  const url = `https://${host}:8006/api2/json/nodes/${node}/qemu`;
  const response = await axios.get(url, {
    httpsAgent: agent,
    headers: {
      'Cookie': `PVEAuthCookie=${auth.ticket}`,
      'CSRFPreventionToken': auth.CSRFPreventionToken
    }
  });

  return response.data;
}

// Usage
(async () => {
  const auth = await proxmoxAuth('192.168.1.100', 'root@pam', 'password');
  const vms = await listVMs('192.168.1.100', 'pve', auth);
  console.log(JSON.stringify(vms, null, 2));
})();
```

## 📦 Installation

### Using the OpenAPI Specification

1. Clone this repository:
```bash
git clone https://github.com/ramphy/proxmox-api.git
cd proxmox-api
```

2. Choose your preferred method:

#### 🚀 Option A: Ready-to-Use Postman Collection
   - **Postman**: Import → Upload Files → Select `Proxmox VE API.postman_collection.json`
   - **Instant access**: Pre-configured requests with examples and documentation

#### 🤖 Option B: AI/LLM Development
   - Use [`PROXMOX_API_LLM_GUIDE.md`](PROXMOX_API_LLM_GUIDE.md) as context for AI assistants
   - Perfect for ChatGPT, Claude, or any LLM-powered development

#### 🔧 Option C: OpenAPI Tools
   - **Postman**: Import → Upload Files → Select `openapi.json`
   - **Insomnia**: Import/Export → Import Data → From File → Select `openapi.json`
   - **Swagger UI**: Upload the `openapi.json` file directly

### Generate SDK from OpenAPI

You can generate client libraries in various languages using OpenAPI Generator:

```bash
# Install OpenAPI Generator
npm install -g @openapitools/openapi-generator-cli

# Generate Python client
openapi-generator-cli generate -i openapi.json -g python -o ./sdk/python

# Generate JavaScript client
openapi-generator-cli generate -i openapi.json -g javascript -o ./sdk/javascript

# Generate Go client
openapi-generator-cli generate -i openapi.json -g go -o ./sdk/go
```

## 🔐 Authentication

Proxmox VE API uses two authentication methods (because one would be too simple):

### Ticket Authentication (Session-based)

**Recommended for short-term scripts and interactive applications**

Tickets are signed random text values with a 2-hour lifetime. For write operations (POST/PUT/DELETE), you also need a CSRF prevention token.

```bash
# Get ticket and CSRF token
curl -X POST https://your-proxmox-server:8006/api2/json/access/ticket \
  -d "username=root@pam&password=yourpassword" \
  -k

# Response includes:
# - "ticket": "PVE:root@pam:..."
# - "CSRFPreventionToken": "..."

# Use in subsequent requests
curl -X GET https://your-proxmox-server:8006/api2/json/nodes \
  -H "Cookie: PVEAuthCookie=PVE:root@pam:..." \
  -H "CSRFPreventionToken: ..." \
  -k
```

### API Token Authentication (Stateless)

**Recommended for automation and long-running applications**

1. Create API token in Proxmox VE UI: **Datacenter** → **Permissions** → **API Tokens**
2. Use the token in Authorization header

```bash
curl -X GET https://your-proxmox-server:8006/api2/json/nodes \
  -H "Authorization: PVEAPIToken=root@pam!api-token=1234-5678-9abc-def0" \
  -k
```

## 📚 OpenAPI Specification

### 🎉 The Crown Jewel: [`openapi.json`](openapi.json)

This magnificent 2.1MB beast contains:

- **Every. Single. Endpoint.** - All Proxmox VE API routes (yes, even the obscure ones)
- **Complete Schemas** - Request/response models that actually make sense
- **Parameter Descriptions** - Because guessing is for lottery tickets, not APIs
- **Authentication Flows** - Both token and ticket methods explained
- **Real Examples** - Not just theory, but actual working samples

> 💡 **Pro Tip**: This file took countless hours of reverse engineering. Treat it like the precious artifact it is!

### Viewing the Specification

You can view the OpenAPI specification interactively using:

1. **Official Proxmox API Viewer**:
   - Visit [pve.proxmox.com/pve-docs/api-viewer/](https://pve.proxmox.com/pve-docs/api-viewer/)

2. **Online Swagger Editor**:
   - Visit [editor.swagger.io](https://editor.swagger.io)
   - File → Import file → Select `openapi.json`

3. **Local Swagger UI**:
```bash
docker run -p 8080:8080 -e SWAGGER_JSON=/openapi.json -v $(pwd):/usr/share/nginx/html swaggerapi/swagger-ui
```
Then visit http://localhost:8080

4. **ReDoc**:
```bash
npx redoc-cli serve openapi.json
```

## 🛤️ API Endpoints

### API Base URL
All API calls use HTTPS on port 8006:
```
https://your-proxmox-server:8006/api2/json/
```

### Main Categories

- **Cluster** (`/cluster/*`) - Cluster management, replication, HA
- **Nodes** (`/nodes/*`) - Node management, services, network, storage
- **Virtual Machines** (`/nodes/{node}/qemu/*`) - VM creation, configuration, control
- **Containers** (`/nodes/{node}/lxc/*`) - LXC container management
- **Storage** (`/storage/*`) - Storage configuration and content management
- **Access Control** (`/access/*`) - Users, groups, roles, permissions, authentication
- **Pools** (`/pools/*`) - Resource pools management
- **Backup** (`/nodes/{node}/vzdump/*`) - Backup and restore operations

### Common Operations

| Operation | Endpoint | Method |
|-----------|----------|--------|
| List all nodes | `/api2/json/nodes` | GET |
| List VMs on node | `/api2/json/nodes/{node}/qemu` | GET |
| Create VM | `/api2/json/nodes/{node}/qemu` | POST |
| Start VM | `/api2/json/nodes/{node}/qemu/{vmid}/status/start` | POST |
| Stop VM | `/api2/json/nodes/{node}/qemu/{vmid}/status/stop` | POST |
| Clone VM | `/api2/json/nodes/{node}/qemu/{vmid}/clone` | POST |
| Backup VM | `/api2/json/nodes/{node}/vzdump` | POST |
| List storages | `/api2/json/storage` | GET |
| Get cluster status | `/api2/json/cluster/status` | GET |

## 📖 Resources

### Official Documentation

- **[Proxmox VE API Documentation](https://pve.proxmox.com/wiki/Proxmox_VE_API)** - Official API wiki
- **[Proxmox VE API Viewer](https://pve.proxmox.com/pve-docs/api-viewer/)** - Interactive API browser
- **[Proxmox VE Administration Guide](https://pve.proxmox.com/pve-docs/pve-admin-guide.html)** - Complete admin documentation

### Command Line Tools

- **`pvesh`** - Native Proxmox shell for API access (available on Proxmox nodes)
```bash
# Examples using pvesh
pvesh get /nodes
pvesh get /nodes/pve/qemu
pvesh create /nodes/pve/qemu --vmid 100 --memory 2048 --cores 2
```

### Client Libraries

#### Official
- **[libpve-apiclient-perl](https://git.proxmox.com/?p=libpve-apiclient-perl.git)** - Official Perl client

#### Community Libraries
- **[proxmoxer](https://github.com/proxmoxer/proxmoxer)** - Python wrapper for Proxmox API
- **[go-proxmox](https://github.com/Telmate/proxmox-api-go)** - Go client for Proxmox API
- **[node-proxmox](https://github.com/alo-is/node-proxmox)** - Node.js client
- **[PowerShell-ProxmoxCLI](https://github.com/Corsinvest/cv4pve-api-powershell)** - PowerShell module

### Community Resources

- **[Proxmox Forum](https://forum.proxmox.com/)** - Official community forum
- **[Reddit r/Proxmox](https://www.reddit.com/r/Proxmox/)** - Reddit community
- **[Proxmox VE GitHub Topics](https://github.com/topics/proxmox-ve)** - Related projects on GitHub

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Updating the OpenAPI Specification

Found a bug in our precious `openapi.json`? Want to add that one endpoint we missed?

1. Edit the `openapi.json` file (carefully, it's fragile! 🥚)
2. Validate the JSON structure (broken JSON = sad developers)
3. Test against a real Proxmox VE installation (because theory ≠ reality)
4. Submit a PR with a description (bonus points for humor 😄)

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## ⭐ Star History

If you find this project useful, please consider giving it a star ⭐ to help others discover it!

---

**Note**: This is an unofficial documentation project. For official Proxmox VE support and documentation, please visit [proxmox.com](https://www.proxmox.com).

**Repository Structure**:
- 🌟 `openapi.json` - **THE MAIN ATTRACTION** - 2.1MB of pure Proxmox API specification
- 🤖 `PROXMOX_API_LLM_GUIDE.md` - **AI DEVELOPER'S BEST FRIEND** - Ultra-compact guide for LLMs (7KB)
- 🔧 `Proxmox VE API.postman_collection.json` - **INSTANT API TESTING** - Ready-to-import Postman collection (2.3MB)
- 📜 `LICENSE` - Apache 2.0 license (boring but necessary)
- 📖 `README.md` - This fabulous documentation you're reading right now