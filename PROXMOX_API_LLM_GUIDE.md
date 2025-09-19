# Proxmox API Guide for LLMs

**Base URL**: `https://{host}:8006/api2/json/`
**OpenAPI Spec**: `openapi.json` (2.1MB complete specification)

## Quick Auth

### Ticket Auth (2h expiry)
```python
auth = requests.post(f"{base_url}/access/ticket",
    data={"username": "root@pam", "password": "password"}, verify=False).json()["data"]
headers = {"Cookie": f"PVEAuthCookie={auth['ticket']}", "CSRFPreventionToken": auth["CSRFPreventionToken"]}
```

### Token Auth (permanent)
```python
headers = {"Authorization": "PVEAPIToken=root@pam!token-name=uuid-value"}
```

## Common Operations Reference

| Operation | Method | Endpoint | Required Params |
|-----------|--------|----------|-----------------|
| **Authentication** |
| Login | POST | `/access/ticket` | username, password |
| **Virtual Machines** |
| List VMs | GET | `/nodes/{node}/qemu` | node |
| Create VM | POST | `/nodes/{node}/qemu` | node, vmid |
| VM Status | GET | `/nodes/{node}/qemu/{vmid}/status/current` | node, vmid |
| Start VM | POST | `/nodes/{node}/qemu/{vmid}/status/start` | node, vmid |
| Stop VM | POST | `/nodes/{node}/qemu/{vmid}/status/stop` | node, vmid |
| Shutdown VM | POST | `/nodes/{node}/qemu/{vmid}/status/shutdown` | node, vmid |
| Reboot VM | POST | `/nodes/{node}/qemu/{vmid}/status/reboot` | node, vmid |
| Clone VM | POST | `/nodes/{node}/qemu/{vmid}/clone` | node, vmid, newid |
| Delete VM | DELETE | `/nodes/{node}/qemu/{vmid}` | node, vmid |
| VM Config | GET | `/nodes/{node}/qemu/{vmid}/config` | node, vmid |
| Update Config | PUT | `/nodes/{node}/qemu/{vmid}/config` | node, vmid |
| **Containers** |
| List Containers | GET | `/nodes/{node}/lxc` | node |
| Create Container | POST | `/nodes/{node}/lxc` | node, vmid, ostemplate |
| Container Status | GET | `/nodes/{node}/lxc/{vmid}/status/current` | node, vmid |
| Start Container | POST | `/nodes/{node}/lxc/{vmid}/status/start` | node, vmid |
| Stop Container | POST | `/nodes/{node}/lxc/{vmid}/status/stop` | node, vmid |
| **Storage** |
| List Storage | GET | `/storage` | - |
| Node Storage | GET | `/nodes/{node}/storage` | node |
| Storage Content | GET | `/nodes/{node}/storage/{storage}/content` | node, storage |
| **Snapshots** |
| List Snapshots | GET | `/nodes/{node}/qemu/{vmid}/snapshot` | node, vmid |
| Create Snapshot | POST | `/nodes/{node}/qemu/{vmid}/snapshot` | node, vmid, snapname |
| Rollback Snapshot | POST | `/nodes/{node}/qemu/{vmid}/snapshot/{snapname}/rollback` | node, vmid, snapname |
| **Backups** |
| List Backup Jobs | GET | `/cluster/backup` | - |
| Create Backup Job | POST | `/cluster/backup` | - |
| Manual Backup | POST | `/nodes/{node}/vzdump` | node, vmid |
| **Monitoring** |
| Node Status | GET | `/nodes/{node}/status` | node |
| Cluster Status | GET | `/cluster/status` | - |
| Tasks | GET | `/nodes/{node}/tasks` | node |

## Code Templates

### Python Minimal Client
```python
import requests
from urllib3 import disable_warnings; disable_warnings()

class ProxmoxAPI:
    def __init__(self, host, user, password):
        self.base = f"https://{host}:8006/api2/json"
        auth = requests.post(f"{self.base}/access/ticket",
            data={"username": user, "password": password}, verify=False).json()["data"]
        self.headers = {"Cookie": f"PVEAuthCookie={auth['ticket']}",
                       "CSRFPreventionToken": auth["CSRFPreventionToken"]}

    def get(self, path): return requests.get(f"{self.base}{path}", headers=self.headers, verify=False).json()
    def post(self, path, data={}): return requests.post(f"{self.base}{path}", headers=self.headers, data=data, verify=False).json()
    def put(self, path, data={}): return requests.put(f"{self.base}{path}", headers=self.headers, data=data, verify=False).json()
    def delete(self, path): return requests.delete(f"{self.base}{path}", headers=self.headers, verify=False).json()

# Usage: api = ProxmoxAPI("192.168.1.100", "root@pam", "password")
```

### JavaScript Minimal Client
```javascript
class ProxmoxAPI {
    constructor(host, user, pass) {
        this.base = `https://${host}:8006/api2/json`;
        this.agent = new (require('https')).Agent({rejectUnauthorized: false});
    }

    async auth(user, pass) {
        const auth = await this.post('/access/ticket', `username=${user}&password=${pass}`);
        this.headers = {Cookie: `PVEAuthCookie=${auth.data.ticket}`, CSRFPreventionToken: auth.data.CSRFPreventionToken};
    }

    async get(path) { return (await require('axios').get(`${this.base}${path}`, {headers: this.headers, httpsAgent: this.agent})).data; }
    async post(path, data) { return (await require('axios').post(`${this.base}${path}`, data, {headers: this.headers, httpsAgent: this.agent})).data; }
}
```

## Response Patterns

### Standard Success Response
```json
{"data": {...}, "success": 1}
```

### Task Response (for long operations)
```json
{"data": "UPID:node:task_id:type:user:status", "success": 1}
```

### Error Response
```json
{"errors": {"param": "error message"}, "success": 0}
```

## VM Creation Pattern
```python
# Minimal VM creation
vm_config = {
    "vmid": 100,
    "name": "test-vm",
    "memory": 2048,
    "cores": 2,
    "net0": "virtio,bridge=vmbr0",
    "scsi0": "local-lvm:32",
    "ostype": "l26"
}
api.post(f"/nodes/{node}/qemu", vm_config)
```

## Container Creation Pattern
```python
# Minimal container creation
ct_config = {
    "vmid": 200,
    "hostname": "test-ct",
    "memory": 512,
    "rootfs": "local-lvm:8",
    "ostemplate": "local:vztmpl/debian-11-standard_11.3-1_amd64.tar.zst",
    "net0": "name=eth0,bridge=vmbr0,ip=dhcp"
}
api.post(f"/nodes/{node}/lxc", ct_config)
```

## Batch Operations Pattern
```python
# Start multiple VMs
vms = [100, 101, 102]
for vmid in vms:
    api.post(f"/nodes/{node}/qemu/{vmid}/status/start")

# Check status of multiple VMs
statuses = {vmid: api.get(f"/nodes/{node}/qemu/{vmid}/status/current")["data"]["status"] for vmid in vms}
```

## Common Parameters

### VM Config Parameters
- `vmid`: VM ID (required)
- `name`: VM name
- `memory`: RAM in MB
- `cores`: CPU cores
- `sockets`: CPU sockets
- `net0`: Network interface (virtio,bridge=vmbr0)
- `scsi0/ide0/sata0`: Storage (storage:size)
- `ostype`: OS type (l26, win10, etc.)

### Container Config Parameters
- `vmid`: Container ID (required)
- `hostname`: Container hostname
- `memory`: RAM in MB
- `rootfs`: Root filesystem (storage:size)
- `ostemplate`: OS template path
- `net0`: Network (name=eth0,bridge=vmbr0,ip=dhcp)

## Error Handling Pattern
```python
try:
    result = api.post(f"/nodes/{node}/qemu/{vmid}/status/start")
    if result.get("success"):
        task_id = result["data"]  # Monitor task if needed
    else:
        print(f"Error: {result.get('errors', 'Unknown error')}")
except Exception as e:
    print(f"API Error: {e}")
```

## Notes for LLMs
- Always use `verify=False` for self-signed certificates
- Include CSRF token for POST/PUT/DELETE operations with ticket auth
- Task IDs can be monitored at `/nodes/{node}/tasks/{upid}/status`
- Storage format: `{storage-name}:{size}` (e.g., "local-lvm:32")
- Network format: `{type},bridge={bridge}[,additional-params]`
- Most operations return task IDs for async execution
- Use `/cluster/nextid` to get next available VM ID