# Ansible Windows Server Configuration

This directory contains Ansible configuration files to connect to and configure a Windows server with a modular, maintainable approach.

## Files Description

- `ansible.cfg`: Main Ansible configuration file with Windows-specific settings and fact caching
- `inventory.yml`: Inventory file containing the Windows server details (YAML format)
- `host_vars/w2025dc.yml`: Host-specific variable overrides
- `playbook.yml`: Main playbook file that imports all individual plays
- `group_vars/windows_servers/vars.yml`: Variables file for Windows server configuration
- `plays/`: Directory containing individual play files
  - `test-connection.yml`: Test play to verify connectivity and gather system info
  - `network-settings.yml`: Network configuration (DNS, hostname, firewall rules)
  - `configure-rdp.yml`: Remote Desktop Protocol configuration with rollback
  - `install-chocolatey.yml`: Chocolatey package manager installation
  - `install-software.yml`: Common software installation via Chocolatey
  - `update-software.yml`: Software package updates via Chocolatey
  - `update-system.yml`: Windows system updates (security updates only)
  - `cleanup.yml`: Basic cleanup tasks (disk cleanup, temp files, etc.)
  - `domain-controller.yml`: Active Directory Domain Services setup
- `Enable-AnsibleRemoting.ps1`: PowerShell script to enable Ansible remoting on Windows

## Prerequisites

1. **On the Ansible control node (Linux):**
   - Install Ansible: `sudo apt install ansible`
   - Install pywinrm: `sudo apt install python3-winrm`

2. **On the Windows server (10.211.55.101):**
   - Run the `Enable-AnsibleRemoting.ps1` script as Administrator
   - Ensure WinRM is enabled and configured

## Usage

### Run Complete Workflow
```bash
ansible-playbook playbook.yml
```

### Individual Plays

#### Test Connection
```bash
ansible-playbook plays/test-connection.yml
```

#### Network Settings
```bash
ansible-playbook plays/network-settings.yml
```

#### Configure RDP
```bash
ansible-playbook plays/configure-rdp.yml
```

#### Install Chocolatey
```bash
ansible-playbook plays/install-chocolatey.yml
```

#### Install Software
```bash
ansible-playbook plays/install-software.yml
```

#### Update Software
```bash
ansible-playbook plays/update-software.yml
```

#### Update System (Security Updates Only)
```bash
ansible-playbook plays/update-system.yml
```

#### Basic Cleanup
```bash
ansible-playbook plays/cleanup.yml
```

#### Domain Controller Setup
```bash
ansible-playbook plays/domain-controller.yml
```

### Ad-hoc Commands

#### Test Inventory
```bash
ansible windows_servers -m win_ping
```

#### Get System Information
```bash
ansible windows_servers -m win_shell -a "Get-ComputerInfo"
```

#### Check Windows Services
```bash
ansible windows_servers -m win_service_facts
```

#### Check Installed Software
```bash
ansible windows_servers -m win_shell -a "choco list"
```

#### Check RDP Status
```bash
ansible windows_servers -m win_shell -a "Get-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name 'fDenyTSConnections'"
```

## Configuration Details

- **Server IP**: 10.211.55.101 (configurable via inventory and host_vars)
- **Username**: Administrator
- **Password**: Uses default_password variable (Pa55w.rd)
- **Connection**: WinRM over HTTP (port 5985)
- **Authentication**: NTLM
- **Hostname**: Uses inventory_hostname (w2025dc)
- **Domain**: acme.lab (when configured as DC)
- **Passwords**: All use default_password variable (Pa55w.rd)
- **Fact Caching**: JSON file-based caching for improved performance

## Variables

All configuration variables are stored in `group_vars/windows_servers/vars.yml`:

- **Default Password**: Centralized password variable for all services
- **IP Address**: Uses ansible_host from inventory (10.211.55.101)
- **Network Configuration**: Gateway (10.211.55.1) and subnet (24) settings
- **Connection Settings**: WinRM configuration
- **Domain Controller Settings**: Domain name, passwords, NetBIOS name
- **Network Settings**: DNS servers (inventory IP as primary, 8.8.8.8 as secondary)
- **Windows Features**: Features to install
- **Cleanup Settings**: Paths to clean during maintenance
- **Firewall Rules**: ICMP and RDP firewall configurations
- **Software Packages**: List of software to install via Chocolatey

## Playbook Features

### Test Connection
- Verifies connectivity to Windows server
- Displays basic system information (hostname, OS version, user, directory)
- Optimized with `gather_facts: no` for performance

### Network Settings
- Configures DNS servers (target IP as primary, 8.8.8.8 as secondary)
- Sets hostname using inventory_hostname variable
- Enables ping through Windows Firewall (ICMP Echo Request/Reply)
- Uses variables for gateway and subnet configuration
- Assumes host is already configured with IP (no static IP configuration)

### RDP Configuration
- Enables Remote Desktop Protocol
- Configures Network Level Authentication (NLA)
- Sets up RDP firewall rules (TCP/UDP port 3389)
- Implements robust rollback mechanism with `block`/`rescue`/`always`
- Verifies RDP configuration and connectivity
- Provides connection details for RDP clients

### Chocolatey Package Management
- Installs Chocolatey package manager
- Installs common software packages (VS Code, Chrome, 7-Zip, etc.)
- Updates installed software packages
- Provides package management capabilities

### System Updates
- Installs security updates only (feature updates commented out)
- Automatic reboot handling with connection wait
- Optimized with `gather_facts: no` for performance

### Basic Cleanup
- Runs Disk Cleanup (cleanmgr) automatically
- Cleans temporary files and Windows Update cache
- Removes browser cache (if browsers are installed)
- Optimized with `gather_facts: no` for performance

### Domain Controller Setup
- Installs Active Directory Domain Services
- Installs DNS Server and Group Policy Management
- Promotes server to Domain Controller for acme.lab domain
- Uses variables for passwords (safe mode and domain admin)
- Optimized with `gather_facts: no` for performance

## Performance Optimizations

- **Fact Caching**: JSON file-based caching with 24-hour timeout
- **Smart Gathering**: Uses `gathering = smart` for conditional fact collection
- **Selective Fact Gathering**: Most plays use `gather_facts: no` unless needed
- **Connection Timeouts**: Optimized WinRM timeouts for better reliability

## Error Handling

- **RDP Rollback**: Automatic rollback of RDP configuration on failure
- **Firewall Rules**: Proper cleanup of firewall rules on rollback
- **Registry Settings**: Reversion of registry changes on failure
- **Connection Recovery**: Automatic connection recovery after reboots

## Security Notes

- The configuration disables SSL certificate validation for testing purposes
- In production, consider using HTTPS (port 5986) with proper certificates
- Store passwords securely using Ansible Vault in production environments
- RDP is configured with Network Level Authentication (NLA) for security
- Firewall rules are specific and minimal for required functionality

## Package Management

The configuration uses **Chocolatey** as the Windows package manager:

- **Installation**: `choco install package-name`
- **Updates**: `choco upgrade all` or `choco upgrade package-name`
- **Listing**: `choco list` or `choco list --local-only`
- **Outdated**: `choco outdated`
- **Uninstall**: `choco uninstall package-name`

## Troubleshooting

### Connection Issues
- Verify WinRM is enabled on Windows server
- Check firewall rules for port 5985
- Ensure Administrator credentials are correct

### RDP Issues
- Verify RDP service is running
- Check firewall rules for port 3389
- Ensure NLA is properly configured

### Package Installation Issues
- Check Chocolatey installation
- Verify internet connectivity
- Review Chocolatey logs for errors

### Performance Issues
- Clear fact cache: `rm -rf /tmp/ansible_facts`
- Check fact cache timeout settings
- Use `gather_facts: no` for plays that don't need facts 