# SmartAgent Remote Installation and Startup Guide

This guide explains how to install and start the AppDynamics SmartAgent on multiple remote hosts using the `smartagentctl` tool.

## Prerequisites

- SSH access to all remote hosts
- SSH private key configured for authentication
- Sudo privileges on the control node
- Remote hosts must have SSH enabled and accessible

## Directory Structure

Navigate to the SmartAgent installation directory:

```bash
cd /home/ubuntu/appdsm
```

The directory contains:
- `smartagentctl` - Command-line utility to manage SmartAgent
- `smartagent` - The SmartAgent binary
- `config.ini` - Main configuration file
- `remote.yaml` - Remote hosts configuration file
- `conf/` - Additional configuration files
- `lib/` - Required libraries

## Configuration Files

### 1. config.ini

The `config.ini` file contains the main SmartAgent configuration that will be deployed to all remote hosts.

**Location:** `/home/ubuntu/appdsm/config.ini`

#### Key Sections:

**Controller Configuration:**
```ini
ControllerURL    = fso-tme.saas.appdynamics.com
ControllerPort   = 443
FMServicePort    = 443
AccountAccessKey = uylojnqg6e81
AccountName      = fso-tme
EnableSSL        = true
```
- `ControllerURL`: The AppDynamics SaaS controller endpoint
- `ControllerPort`: HTTPS port for the controller (default: 443)
- `FMServicePort`: Flow Monitoring service port
- `AccountAccessKey`: Your AppDynamics account access key
- `AccountName`: Your AppDynamics account name
- `EnableSSL`: Enable SSL/TLS encryption (true/false)

**Common Configuration:**
```ini
[CommonConfig]
AgentName            = smartagent
PollingIntervalInSec = 300
Tags                 = 
ServiceName          = 
```
- `AgentName`: Name identifier for the agent
- `PollingIntervalInSec`: How often the agent polls for data (in seconds)
- `Tags`: Custom tags for categorizing agents (comma-separated)
- `ServiceName`: Optional service name for grouping

**Telemetry Settings:**
```ini
[Telemetry]
LogLevel  = DEBUG
LogFile   = /opt/appdynamics/appdsmartagent/log.log
Profiling = false
```
- `LogLevel`: Logging verbosity (DEBUG, INFO, WARN, ERROR)
- `LogFile`: Path where logs will be written on remote hosts
- `Profiling`: Enable performance profiling (true/false)

**TLS Client Settings:**
```ini
[TLSClientSetting]
Insecure        = false
AgentHTTPProxy  = 
AgentHTTPSProxy = 
AgentNoProxy    = 
```
- `Insecure`: Skip TLS certificate verification (not recommended for production)
- `AgentHTTPProxy`: HTTP proxy server URL (if required)
- `AgentHTTPSProxy`: HTTPS proxy server URL (if required)
- `AgentNoProxy`: Comma-separated list of hosts to bypass proxy

**Auto Discovery:**
```ini
[AutoDiscovery]
RunAutoDiscovery          = false
ExcludeLabels             = process.cpu.usage,process.memory.usage
ExcludeProcesses          = 
ExcludeUsers              = 
AutoDiscoveryTimeInterval = 4h
AutoInstall               = false
```
- `RunAutoDiscovery`: Automatically discover applications (true/false)
- `ExcludeLabels`: Metrics to exclude from discovery
- `ExcludeProcesses`: Process names to exclude from monitoring
- `ExcludeUsers`: User accounts to exclude from monitoring
- `AutoDiscoveryTimeInterval`: How often to run discovery
- `AutoInstall`: Automatically install discovered applications

**Task Configuration:**
```ini
[TaskConfig]
NativeEnable        = true
UserPortalUserName  = 
UserPortalPassword  = 
UserPortalAuth      = none
AutoUpdateLdPreload = true
```
- `NativeEnable`: Enable native instrumentation
- `AutoUpdateLdPreload`: Automatically update LD_PRELOAD settings

### 2. remote.yaml

The `remote.yaml` file defines the remote hosts where SmartAgent will be installed and the connection parameters.

**Location:** `/home/ubuntu/appdsm/remote.yaml`

```yaml
max_concurrency: 4
remote_dir: "/opt/appdynamics/appdsmartagent"
protocol:
  type: ssh
  auth:
    username: ubuntu
    private_key_path: /home/ubuntu/.ssh/id_rsa
    privileged: true
    ignore_host_key_validation: true
    known_hosts_path: /home/ubuntu/.ssh/known_hosts
hosts:
  - host: "172.31.1.243"
    port: 22
    user: root
    group: root
  - host: "172.31.1.48"
    port: 22
    user: root
    group: root
  - host: "172.31.1.142"
    port: 22
    user: root
    group: root
  - host: "172.31.1.5"
    port: 22
    user: root
    group: root
```

#### Configuration Parameters:

**Global Settings:**
- `max_concurrency`: Maximum number of hosts to process simultaneously (4)
- `remote_dir`: Installation directory on remote hosts

**Protocol Configuration:**
- `type`: Connection protocol (ssh)
- `auth.username`: SSH username for authentication (ubuntu)
- `auth.private_key_path`: Path to SSH private key
- `auth.privileged`: Run agent with elevated privileges (true/false)
  - Set to `true` to install as root/systemd service
  - Set to `false` to install as a user process
- `auth.ignore_host_key_validation`: Skip SSH host key verification
- `auth.known_hosts_path`: Path to SSH known_hosts file

**Host Definitions:**

Each host entry requires:
- `host`: IP address or hostname of the remote machine
- `port`: SSH port (default: 22)
- `user`: User account that will own the SmartAgent process
- `group`: Group that will own the SmartAgent process

#### Adding More Hosts

To add additional remote hosts, append to the `hosts` list:

```yaml
hosts:
  - host: "172.31.1.xxx"
    port: 22
    user: root
    group: root
```

## Installation and Startup Process

### Step 1: Navigate to the Installation Directory

```bash
cd /home/ubuntu/appdsm
```

### Step 2: Verify Configuration Files

Ensure both configuration files are properly configured:

```bash
# Review remote hosts configuration
cat remote.yaml

# Review agent configuration
cat config.ini
```

### Step 3: Start SmartAgent on Remote Hosts

Run the following command to start SmartAgent on all remote hosts defined in `remote.yaml`:

```bash
sudo ./smartagentctl start --remote --verbose
```

**Command Breakdown:**
- `sudo`: Required for privileged operations
- `./smartagentctl`: The control utility
- `start`: Command to start the SmartAgent
- `--remote`: Deploy to remote hosts (reads from `remote.yaml`)
- `--verbose`: Enable detailed debug logging

### What Happens During Startup

1. **Connection**: Establishes SSH connections to all defined hosts
2. **Transfer**: Copies SmartAgent binaries and configuration to remote hosts
3. **Installation**: Installs SmartAgent in `/opt/appdynamics/appdsmartagent/` on each host
4. **Startup**: Starts the SmartAgent process on each remote host
5. **Logging**: Outputs detailed progress to console and log file

### Monitoring the Process

The `--verbose` flag provides detailed output including:
- SSH connection status
- File transfer progress
- Installation steps
- Agent startup status
- Any errors or warnings

Logs are also written to:
- **Local**: `/home/ubuntu/appdsm/log.log`
- **Remote**: `/opt/appdynamics/appdsmartagent/log.log` (on each host)

## Additional Commands

### Check Status

```bash
sudo ./smartagentctl status --remote --verbose
```

### Stop SmartAgent

```bash
sudo ./smartagentctl stop --remote --verbose
```

### Install SmartAgent (without starting)

```bash
sudo ./smartagentctl install --remote --verbose
```

### Uninstall SmartAgent

```bash
sudo ./smartagentctl uninstall --remote --verbose
```

### Install as System Service

To install SmartAgent as a systemd service:

```bash
sudo ./smartagentctl start --remote --verbose --service
```

## Troubleshooting

### SSH Connection Issues

1. Verify SSH key permissions:
   ```bash
   chmod 600 /home/ubuntu/.ssh/id_rsa
   ```

2. Test SSH connectivity manually:
   ```bash
   ssh -i /home/ubuntu/.ssh/id_rsa ubuntu@172.31.1.243
   ```

3. Check if remote hosts are reachable:
   ```bash
   ping 172.31.1.243
   ```

### Permission Issues

- Ensure the control node user has sudo privileges
- Verify the SSH user has appropriate permissions on remote hosts
- Check `privileged: true` setting in `remote.yaml`

### Agent Not Starting

1. Check logs on the control node:
   ```bash
   tail -f /home/ubuntu/appdsm/log.log
   ```

2. SSH to remote host and check logs:
   ```bash
   ssh ubuntu@172.31.1.243
   tail -f /opt/appdynamics/appdsmartagent/log.log
   ```

3. Verify the agent process:
   ```bash
   ps aux | grep smartagent
   ```

### Configuration Errors

- Validate YAML syntax in `remote.yaml`
- Verify all required fields in `config.ini`
- Check that AccountAccessKey and AccountName are correct
- Ensure ControllerURL is reachable from remote hosts

## Security Best Practices

1. **SSH Keys**: Use strong SSH keys (RSA 4096-bit or ED25519)
2. **Access Keys**: Store AccountAccessKey securely, consider using environment variables
3. **Host Key Validation**: In production, set `ignore_host_key_validation: false`
4. **SSL/TLS**: Always use `EnableSSL = true` for controller communication
5. **Log Files**: Restrict access to log files containing sensitive information
6. **Privileged Mode**: Only use `privileged: true` when necessary

## Next Steps

After starting SmartAgent on remote hosts:

1. Verify agents appear in the AppDynamics Controller UI
2. Check that metrics are being collected
3. Configure application monitoring as needed
4. Set up alerts and dashboards
5. Monitor agent health and performance

## Support

For additional help:
- View all available commands: `./smartagentctl --help`
- View command-specific help: `./smartagentctl [command] --help`
- Check AppDynamics documentation
- Review log files for detailed error messages
