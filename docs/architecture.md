# weak-rdp-bruteforce-dc - Architecture Documentation

## Lab Overview

This document details the architecture and design of the security lab environment.

## Components

### Virtual Machines
- **Attacker VM**: Kali Linux for attack simulation
- **Target VM**: Windows Server Domain Controller
- **Monitoring VM**: Security Onion/Elastic for security monitoring
- **Optional VMs**: Additional systems for specific scenarios

### Network Architecture
. . .
```
Attacker Network: 192.168.1.0/24
Target Network: 192.168.2.0/24  
Monitoring Network: 192.168.3.0/24
```

### Security Stack
- **Log Aggregation**: Security Onion, Elastic Stack
- **Endpoint Monitoring**: Sysmon, Windows Event Forwarding
- **Network Security**: Suricata, Zeek
- **Detection Engine**: EQL, Sigma rules

## Data Flow

1. **Telemetry Collection**: Endpoint and network events are collected
2. **Log Aggregation**: Events are forwarded to the SIEM
3. **Rule Evaluation**: Detection rules evaluate incoming events
4. **Alert Generation**: Matches trigger security alerts
5. **Investigation**: Alerts are investigated and responded to

## Configuration Details

*(Specific configuration details from the lab setup)*

## Validation & Testing

The architecture was validated through:
- Connectivity testing between components
- Telemetry verification
- Detection rule testing
- Attack simulation validation
