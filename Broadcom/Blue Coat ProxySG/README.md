# Blue Coat ProxySG to LogRhythm SIEM Integration Guide

<div align="center">

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![ProxySG](https://img.shields.io/badge/ProxySG-Latest-blue.svg)](https://www.broadcom.com)
[![LogRhythm](https://img.shields.io/badge/LogRhythm-SIEM-red.svg)](https://logrhythm.com)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](CONTRIBUTING.md)

**Complete configuration guide for integrating Symantec Blue Coat ProxySG with LogRhythm SIEM**

*Zero Log Loss | Complete Visibility | Production Ready*

[Features](#features) •
[Prerequisites](#prerequisites) •
[Quick Start](#quick-start) •
[Configuration](#detailed-configuration) •
[Contributing](#contributing)

</div>

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Architecture](#architecture)
- [Quick Start](#quick-start)
- [Detailed Configuration](#detailed-configuration)
  - [Phase 1: ProxySG Access Log Configuration](#phase-1-proxysg-access-log-configuration)
  - [Phase 2: Enable All Event Logs](#phase-2-enable-all-event-logs)
  - [Phase 3: SSL/TLS Interception Logging](#phase-3-ssltls-interception-logging)
  - [Phase 4: Advanced Threat Protection Logging](#phase-4-advanced-threat-protection-logging)
  - [Phase 5: Policy Trace Logging](#phase-5-policy-trace-logging)
  - [Phase 6: Authentication Realm Logging](#phase-6-authentication-realm-logging)
  - [Phase 7: Bandwidth Gain Logging](#phase-7-bandwidth-gain-logging)
  - [Phase 8: Configure Log Reliability Features](#phase-8-configure-log-reliability-features)
  - [Phase 9: LogRhythm Configuration](#phase-9-logrhythm-configuration)
  - [Phase 10: Verification & Testing](#phase-10-verification--testing)
  - [Phase 11: Create Monitoring & Alerting](#phase-11-create-monitoring--alerting)
  - [Phase 12: Documentation & Maintenance](#phase-12-documentation--maintenance)
- [Verification Checklist](#verification-checklist)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [FAQ](#faq)
- [Contributing](#contributing)
- [License](#license)
- [Support](#support)

---

## 🎯 Overview

This guide provides a comprehensive, step-by-step configuration for integrating **Symantec Blue Coat ProxySG** with **LogRhythm SIEM** to achieve complete visibility into all proxy activity with zero log loss.

### What This Guide Covers

- ✅ Complete ProxySG log configuration (50+ fields per log)
- ✅ All event types (access, security, admin, health, attack detection)
- ✅ Reliable TCP syslog with buffering and backup
- ✅ LogRhythm log source setup and MPE configuration
- ✅ Monitoring, alerting, and validation procedures
- ✅ Troubleshooting and maintenance guidance

### What You'll Achieve

- 🔒 **Zero Log Loss** - TCP syslog with local backup and queuing
- 👁️ **Complete Visibility** - Every ProxySG action logged and visible
- ⚡ **Real-Time Monitoring** - Live dashboards and alerting
- 📊 **Full Compliance** - Meet audit and regulatory requirements
- 🛠️ **Production Ready** - Battle-tested configuration

---

## ✨ Features

### Comprehensive Logging

- **Web Traffic**: HTTP/HTTPS requests with full URL details
- **Security Events**: Blocks, threats, malware, IPS detections
- **Authentication**: Login attempts, failures, authorization events
- **SSL/TLS**: Certificate validation, cipher info, handshake failures
- **Admin Events**: Configuration changes, policy updates
- **System Health**: Performance, resource usage, service status
- **Application Control**: Application detection and operations
- **DLP Events**: Data loss prevention violations (if applicable)

### Reliability Features

- **TCP Syslog**: Guaranteed delivery vs UDP
- **Message Queuing**: 50,000+ message buffer
- **Local Backup**: Automatic local storage on ProxySG
- **Disk Buffering**: Additional protection during outages
- **Connection Monitoring**: Automatic reconnection
- **Health Checks**: Continuous syslog connection validation

### LogRhythm Integration

- **Native Parser Support**: Pre-built or custom MPE policies
- **Field Mapping**: All 50+ fields mapped to LogRhythm metadata
- **Real-Time Alerting**: Gap detection, threat alerts, admin changes
- **Dashboards**: Pre-built monitoring and analysis views
- **Correlation**: Integration with existing LogRhythm rules

---

## 📦 Prerequisites

### Required Access

| System | Requirement | Notes |
|--------|-------------|-------|
| **ProxySG** | Administrator access to Management Console (port 8082) | Full configuration privileges required |
| **LogRhythm** | Administrator access to Console and Deployment Manager | Log source creation and MPE policy management |
| **Network** | Firewall access for rule creation | TCP port 514 (ProxySG → LogRhythm) |

### Software Versions

- **ProxySG**: Latest SGOS (tested on 6.7+)
- **LogRhythm**: 7.x or later (tested on 7.4+)

### Network Requirements

- Direct TCP connectivity from ProxySG to LogRhythm (port 514)
- Latency <100ms recommended (same datacenter)
- Bandwidth: ~1-10 Kbps per 1,000 requests/hour

### Storage Requirements

| Component | Minimum | Recommended | Notes |
|-----------|---------|-------------|-------|
| **ProxySG Local Storage** | 10 GB free | 50 GB free | For local log backup |
| **LogRhythm Storage** | Depends on volume | Calculate using formula¹ | Plan for 30-90 day retention |

¹ Formula: `(Daily Requests × Avg Log Size × Retention Days) / 1024`

Example: `(1,000,000 requests × 1KB × 90 days) = ~85 GB`

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        Blue Coat ProxySG                     │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Access Logs (main_access_log_to_logrhythm)            │  │
│  │  • Web Traffic (HTTP/HTTPS)                            │  │
│  │  • User Authentication                                 │  │
│  │  • URL Categories                                      │  │
│  │  • 50+ Fields per Log                                  │  │
│  └──────────────────────┬─────────────────────────────────┘  │
│                         │                                    │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Event Logs                                            │  │
│  │  • Security Events (auth failures, policy violations)  │  │
│  │  • Admin Events (config changes)                       │  │
│  │  • Health Events (system status)                       │  │
│  │  • Attack Detection (IPS, malware)                     │  │
│  └──────────────────────┬─────────────────────────────────┘  │
│                         │                                    │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Syslog Destination: LogRhythm_SIEM                    │  │
│  │  • Protocol: TCP (reliable)                            │  │
│  │  • Port: 514                                           │  │
│  │  • Queue Size: 50,000 messages                         │  │
│  │  • Local Backup: Enabled                               │  │
│  └──────────────────────┬─────────────────────────────────┘  │
└─────────────────────────┼────────────────────────────────────┘
                          │
                          │ TCP/514
                          │ (Firewall: ALLOW)
                          ▼
┌──────────────────────────────────────────────────────────────┐
│                     LogRhythm System Monitor                 │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Syslog Server (listening on port 514/TCP)             │  │
│  └──────────────────────┬─────────────────────────────────┘  │
│                         │                                    │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Log Source: BlueCoat_ProxySG_Production               │  │
│  │  • Type: Blue Coat ProxySG                             │  │
│  │  • MPE Policy: Custom/Native Parser                    │  │
│  │  • Field Mapping: 50+ fields                           │  │
│  └──────────────────────┬─────────────────────────────────┘  │
│                         │                                    │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Processing & Storage                                  │  │
│  │  • Parsing (MPE)                                       │  │
│  │  • Indexing                                            │  │
│  │  • Correlation Rules                                   │  │
│  │  • Alarms & Notifications                              │  │
│  └──────────────────────┬─────────────────────────────────┘  │
│                         │                                    │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  LogRhythm Console                                     │  │
│  │  • Dashboards                                          │  │
│  │  • Log Search                                          │  │
│  │  • Investigations                                      │  │
│  │  • Reports                                             │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start

### For the Impatient

> ⚠️ **Warning**: This is a simplified quick start. For production environments, follow the [Detailed Configuration](#detailed-configuration) section.

1. **ProxySG**: Create custom log format with 50+ fields (see Phase 1)
2. **ProxySG**: Configure syslog destination to LogRhythm IP:514/TCP
3. **ProxySG**: Link access log to syslog destination
4. **ProxySG**: Enable all event logs (security, admin, health, attack)
5. **LogRhythm**: Add Blue Coat ProxySG log source
6. **LogRhythm**: Configure MPE policy for parsing
7. **Verify**: Check logs flowing in LogRhythm Log Message Viewer

**Estimated Time**: 30-45 minutes (basic setup)

### Recommended Approach

1. Complete the [Pre-Flight Checklist](./PRE_FLIGHT_CHECKLIST.md) ✅
2. Follow the [Detailed Configuration](#detailed-configuration) (2-4 hours)
3. Run [Verification Tests](#phase-10-verification--testing) ✅
4. Configure [Monitoring & Alerting](#phase-11-create-monitoring--alerting) ✅

---

## 📖 Detailed Configuration

### Phase 1: ProxySG Access Log Configuration

#### Step 1: Access ProxySG Management Console

1. Open web browser: `https://<ProxySG-IP>:8082`
2. Login with administrator credentials
3. Accept certificate warning if using self-signed certificate

#### Step 2: Create Comprehensive Access Log Format

**Navigation**: `Configuration → Access Logging → Formats`

1. Click **New** button
2. **Name**: `LogRhythm_Complete_Format`
3. **Type**: Custom Format
4. Click **Add** to configure fields

**Critical Fields (50+ fields for complete visibility)**:

```plaintext
date                                    # Transaction date
time                                    # Transaction time
time-taken                              # Total processing time
c-ip                                    # Client IP address
cs-username                             # Authenticated username
cs-auth-group                           # Authentication group
cs-header(X-Forwarded-For)             # Original client IP (if proxied)
sc-status                               # HTTP status code
s-action                                # Action taken (ALLOWED, DENIED, etc.)
cs-method                               # HTTP method (GET, POST, etc.)
cs-uri-scheme                           # Protocol (http, https, ftp)
cs-host                                 # Destination hostname
cs-uri-port                             # Destination port
cs-uri-path                             # URL path
cs-uri-query                            # Query string
cs(User-Agent)                          # Client user agent
cs(Referer)                             # Referer URL
sc-bytes                                # Bytes sent to client
cs-bytes                                # Bytes received from client
rs(Content-Type)                        # Content MIME type
cs-categories                           # URL filtering categories
x-virus-id                              # Virus/malware ID (if detected)
x-virus-details                         # Virus details
x-icap-error-code                       # ICAP scanning errors
x-icap-error-details                    # ICAP error details
x-bluecoat-application-name             # Detected application
x-bluecoat-application-operation        # Application operation
x-bluecoat-transaction-uuid             # Unique transaction ID
cs-threat-risk                          # Threat risk score
s-supplier-name                         # Upstream server
s-supplier-ip                           # Upstream server IP
rs-version                              # HTTP version
sc-filter-result                        # Content filter result
sc-filter-category                      # Filter category
x-exception-id                          # Policy exception ID
x-exception-category-id                 # Exception category
cs-uri-extension                        # File extension
x-cs-client-ip-country                  # Client country code
sr-bytes-to-client                      # Response bytes to client
sr-bytes-from-server                    # Request bytes from server
time-taken-millisecs                    # Processing time (milliseconds)
x-rs-certificate-validate-status        # SSL certificate validation
x-rs-certificate-observed-errors        # Certificate errors
x-cs-certificate-subject                # Client certificate subject
x-rs-cipher-suite                       # SSL/TLS cipher used
x-cs-ocsp-error                         # OCSP client errors
x-rs-ocsp-error                         # OCSP server errors
cs-auth-mechanism                       # Authentication method used
x-cs-client-ip-reputation               # Client IP reputation score
x-bluecoat-reference-id                 # Reference ID for support
x-bluecoat-placeholder-name             # Placeholder page name
x-sc-connection-issuer-keyring          # SSL issuer keyring
dnslookup-time                          # DNS resolution time
connect-time                            # Connection establishment time
request-time-millisecs                  # Request processing time
response-time-millisecs                 # Response processing time
```

5. Click **OK** to save format

> 💡 **Tip**: This format captures ALL ProxySG activity including web traffic, SSL transactions, authentication, malware, DLP, certificates, threats, applications, and bandwidth.

---

#### Step 3: Create Main Access Log

**Navigation**: `Configuration → Access Logging → Logs`

1. Click **New**
2. Configure:
   - **Name**: `main_access_log_to_logrhythm`
   - **Type**: Access Log
   - **Format**: `LogRhythm_Complete_Format`
   - **Protocol**: TCP

3. **Log File Section**:
   - **Enable**: Yes
   - **Filename**: `main_access`
   - **Max Size**: 100 MB
   - **Max Files**: 10
   - **Upload**: Continuous
   - **Upload Interval**: 60 seconds

4. Click **OK**

---

#### Step 4: Configure Syslog Destination for LogRhythm

**Navigation**: `Configuration → Access Logging → Syslog`

1. Click **New**
2. Configure:

```plaintext
Name: LogRhythm_SIEM
Facility: Local0
Host: <LOGRHYTHM_IP_ADDRESS>    # Replace with actual IP
Port: 514
Protocol: TCP                    # Critical: Use TCP not UDP
Format: Traditional Syslog
Priority: Notice
```

3. **Advanced Settings** (Show Advanced):
   - **Persistent Connection**: Yes
   - **Reconnect Interval**: 60 seconds
   - **Connection Timeout**: 30 seconds
   - **Message Queue Size**: 10000

4. Click **OK**

> ⚠️ **Critical**: Use **TCP** protocol for guaranteed delivery. UDP may lose logs during network congestion.

---

#### Step 5: Link Access Log to Syslog Destination

**Navigation**: `Configuration → Access Logging → Logs → main_access_log_to_logrhythm`

1. Double-click the log created in Step 3
2. In **Remote Servers** section:
   - Click **Add**
   - Select: `LogRhythm_SIEM`
   - Click **OK**

3. Verify configuration:
   - ✅ Format: LogRhythm_Complete_Format
   - ✅ Log to Syslog: LogRhythm_SIEM
   - ✅ Continuous Upload: Enabled

4. Click **OK**

---

### Phase 2: Enable All Event Logs

#### Step 6: Configure Security Event Log

**Navigation**: `Configuration → Access Logging → Event Logs → Security`

1. **Enable**: Yes
2. **Severity Level**: Information (captures all security events)
3. **Log to Syslog**: LogRhythm_SIEM
4. **Format**: CEF (Common Event Format)

5. **Enable these events** (select ALL):
   - ✅ Authentication failures
   - ✅ Authorization failures
   - ✅ Policy violations
   - ✅ Certificate errors
   - ✅ Credential cache events
   - ✅ IWA/Kerberos events
   - ✅ LDAP authentication events
   - ✅ SSL errors
   - ✅ Threat detection events

6. Click **Apply**

---

#### Step 7: Configure Admin Event Log

**Navigation**: `Configuration → Access Logging → Event Logs → Admin`

1. **Enable**: Yes
2. **Severity Level**: Information
3. **Log to Syslog**: LogRhythm_SIEM
4. **Format**: CEF

5. **Enable logging for**:
   - ✅ Configuration changes
   - ✅ User login/logout
   - ✅ Policy installations
   - ✅ System restarts
   - ✅ Backup/restore operations
   - ✅ License changes
   - ✅ Certificate installations

6. Click **Apply**

---

#### Step 8: Configure Health Event Log

**Navigation**: `Configuration → Access Logging → Event Logs → Health`

1. **Enable**: Yes
2. **Severity Level**: Warning
3. **Log to Syslog**: LogRhythm_SIEM
4. **Format**: CEF

5. **Enable logging for**:
   - ✅ CPU/Memory thresholds
   - ✅ Disk space warnings
   - ✅ Service failures
   - ✅ Connection errors
   - ✅ Performance degradation

6. Click **Apply**

---

#### Step 9: Configure Attack Detection Event Log

**Navigation**: `Configuration → Access Logging → Event Logs → Attack Detection`

1. **Enable**: Yes
2. **Severity Level**: Information
3. **Log to Syslog**: LogRhythm_SIEM
4. **Format**: CEF

5. **Enable all attack detection events**:
   - ✅ IPS events
   - ✅ Malware detections
   - ✅ Exploit attempts
   - ✅ DDoS patterns
   - ✅ Suspicious traffic

6. Click **Apply**

---

### Phase 3: SSL/TLS Interception Logging

#### Step 10: Enable SSL Logging

**Navigation**: `Configuration → SSL → Logging`

1. **Enable**: SSL Interception Logging
2. Configure:
   - ✅ Log SSL Errors
   - ✅ Log Certificate Validation
   - ✅ Log Handshake Failures
   - ✅ Log Cipher Negotiations
   - ✅ Include in Access Log

3. **Certificate Validation Logging**:
   - ✅ Log all validation errors
   - ✅ Log expired certificates
   - ✅ Log untrusted certificates
   - ✅ Log hostname mismatches
   - ✅ Log revocation check failures

4. Click **Apply**

---

### Phase 4: Advanced Threat Protection Logging

#### Step 11: Enable ATP Logging (if licensed)

**Navigation**: `Configuration → Security → Advanced Threat Protection`

1. **Enable**: Malware Scanning Logging
2. Configure:
   - ✅ Log all malware detections
   - ✅ Log suspicious file downloads
   - ✅ Log sandboxing results
   - ✅ Log threat intelligence matches
   - ✅ Include ATP data in access logs

3. **File Type Detection Logging**:
   - ✅ Log all file types analyzed
   - ✅ Log file reputation lookups

4. Click **Apply**

---

### Phase 5: Policy Trace Logging

#### Step 12: Enable Policy Trace (Optional - High Volume)

**Navigation**: `Configuration → Policy → Policy Tracing`

1. **Enable**: Policy Trace Logging
2. Configure:
   - **Trace Level**: Summary (or Detailed if needed)
   - **Log to**: Syslog (LogRhythm_SIEM)
   - **Max Trace Entries**: 10000

3. Click **Apply**

> ⚠️ **Note**: Policy trace generates high log volume. Enable only if policy-level visibility is required for troubleshooting.

---

### Phase 6: Authentication Realm Logging

#### Step 13: Configure Authentication Logging

**Navigation**: `Configuration → Authentication → Realms`

For **each authentication realm** configured:

1. Select realm → **Edit**
2. Go to **Logging** tab
3. Enable:
   - ✅ Log authentication attempts
   - ✅ Log successful authentications
   - ✅ Log failed authentications
   - ✅ Log authorization failures
   - ✅ Include in access log

4. Click **OK**
5. **Repeat for all realms** (IWA, LDAP, SAML, etc.)

---

### Phase 7: Bandwidth Gain Logging

#### Step 14: Enable Cache and Compression Logging

**Navigation**: `Statistics → Advanced URL Statistics`

1. **Enable**: Bandwidth Gain Statistics
2. These will be included in access logs showing:
   - Cache hits/misses
   - Compression ratios
   - Bandwidth saved

---

### Phase 8: Configure Log Reliability Features

#### Step 15: Enable Local Log Backup

**Navigation**: `Configuration → Access Logging → Logs → main_access_log_to_logrhythm`

1. Edit the main access log
2. **Local Storage** section:
   - **Enable local file logging**: Yes
   - **Max log file size**: 100 MB
   - **Number of files to retain**: 20
   - **Location**: `/var/log/access`
   - **Compress old logs**: Yes

3. Click **OK**

> 💡 This ensures logs are stored locally if syslog transmission fails temporarily.

---

#### Step 16: Configure Syslog Buffering

**Navigation**: `Configuration → Access Logging → Syslog → LogRhythm_SIEM`

1. Edit LogRhythm syslog destination
2. **Advanced Settings**:
   - **Message Queue Size**: 50000 messages
   - **Queue Full Action**: Drop oldest
   - **Disk Buffer Path**: `/var/log/syslog_buffer`
   - **Max Disk Buffer Size**: 500 MB

3. Click **OK**

---

#### Step 17: Enable Syslog Monitoring

**Navigation**: `Configuration → Health Monitoring → Syslog`

1. Create new health check:
   - **Name**: `LogRhythm_Syslog_Health`
   - **Type**: Syslog Connection
   - **Monitor**: LogRhythm_SIEM destination
   - **Check Interval**: 60 seconds
   - **Alert on failure**: Yes

2. Configure alerting (email/SNMP) for syslog failures
3. Click **Apply**

---

### Phase 9: LogRhythm Configuration

#### Step 18: Configure LogRhythm System Monitor

**On LogRhythm System Monitor server:**

1. Open **LogRhythm Deployment Manager**
2. Navigate to System Monitor → Right-click → **Properties**
3. Verify **Syslog Server** service is running

---

#### Step 19: Add Blue Coat ProxySG Log Source

**In LogRhythm Deployment Manager:**

1. Right-click System Monitor → **Add Log Source**
2. Configure:

```plaintext
Log Source Type: Blue Coat ProxySG
    (or "Flat File - Syslog" if native parser unavailable)
Host: <ProxySG_IP_Address>
Name: BlueCoat_ProxySG_Production
Record Status: Enabled
```

3. **Flat File Settings** (if using generic syslog):
   - **File Path**: Syslog
   - **Log Format**: Syslog
   - **Time Zone**: (match ProxySG timezone)
   - **Time Format**: Auto
   - **Default Message Direction**: Local

4. **MPE Policy**:
   - Select: **Blue Coat ProxySG** (if available)
   - Or: **Create Custom** (see Step 20)

5. Click **Save**

---

#### Step 20: Configure Custom MPE Policy (if needed)

**If LogRhythm doesn't have native Blue Coat parser:**

1. Navigate to **MPE Rules** → **Log Message Processing Policies**
2. Click **New Policy**
3. **Name**: `Blue_Coat_ProxySG_Custom`
4. Add parsing rules

**Example Base Rule:**

```plaintext
Rule Name: BlueCoat_Web_Access
Pattern: <Check ProxySG syslog format>

Field Mappings:
- Source IP: Extract from c-ip field
- Destination URL: Extract from cs-uri-* fields
- Login: Extract from cs-username
- Command: Extract from cs-method
- Object: Extract from full URL
- Classification: Extract from cs-categories
```

5. Map fields to LogRhythm Common Event:
   - Origin (Source IP)
   - Impacted (Destination)
   - Login (Username)
   - Object (URL)
   - Process (Application)
   - Classification (Category)

6. Test with sample logs
7. Click **Save**

---

#### Step 21: Configure Syslog Listener

**On LogRhythm System Monitor:**

1. Navigate to **System Monitor Properties**
2. **Syslog Settings** tab:
   - **TCP Port**: 514 (or custom)
   - **UDP Port**: 514 (disable if using TCP only)
   - **Max Message Size**: 64 KB
   - **Enable TLS**: No (unless ProxySG configured for TLS)

3. Click **OK**
4. Restart System Monitor service if needed

---

#### Step 22: Configure Firewall Rules

**On LogRhythm server firewall:**

Allow inbound connection:
```plaintext
Source IP: <ProxySG_IP_Address>
Destination Port: 514/TCP
Protocol: TCP
Action: Allow
```

**On ProxySG firewall (if applicable):**

Allow outbound connection:
```plaintext
Destination IP: <LogRhythm_IP>
Destination Port: 514/TCP
Protocol: TCP
Action: Allow
```

---

### Phase 10: Verification & Testing

#### Step 23: Test Log Flow from ProxySG

**On ProxySG:**

1. Navigate to **Statistics → Access Logging**
2. View **Log Statistics**:
   - ✅ Entries logged
   - ✅ Entries uploaded
   - ❌ Failed uploads (should be 0)

3. Navigate to **Statistics → Syslog**
4. View **LogRhythm_SIEM** destination:
   - ✅ Connection Status: Connected
   - ✅ Messages Sent: (incrementing)
   - ❌ Messages Failed: 0
   - ✅ Queue Size: <50000

---

#### Step 24: Verify Logs in LogRhythm

**In LogRhythm Console:**

1. Open **Log Message Viewer** (`Tools → Log Message Viewer`)
2. Set filter:
   - **Log Source**: BlueCoat_ProxySG_Production
   - **Time Range**: Last 15 minutes

3. Click **Load**

4. **Verify you see:**
   - ✅ Web access logs (HTTP/HTTPS)
   - ✅ Blocked/denied requests
   - ✅ Authentication events
   - ✅ SSL certificate events
   - ✅ Security events
   - ✅ Admin events

---

#### Step 25: Validate All Log Types

**Create test traffic to verify each log type:**

| Test | Action | Expected Result |
|------|--------|-----------------|
| **Web Access** | Browse to http://www.google.com | Access log with all fields |
| **Blocked Content** | Browse to blocked category | Block action logged |
| **SSL Test** | Browse to https://www.microsoft.com | SSL fields populated |
| **Authentication** | Access auth-required resource | Username in logs |
| **File Download** | Download a file | File type and size logged |
| **Admin Event** | Make config change on ProxySG | Admin event in LogRhythm |

---

#### Step 26: Check for Missing Logs

**Compare ProxySG vs LogRhythm counts:**

1. **ProxySG**: Statistics → Access Logging
   - Note: Total entries logged (last hour)

2. **LogRhythm**: Search logs from ProxySG (last hour)
   - Count total messages

3. **Expected Result**:
   - LogRhythm count ≈ ProxySG count (within 1-2%)
   - If gap >5%, investigate network/parsing issues

---

### Phase 11: Create Monitoring & Alerting

#### Step 27: Create LogRhythm Dashboard

**In LogRhythm Web Console:**

1. Create dashboard: **Blue Coat ProxySG Monitoring**
2. Add widgets:

**Log Volume Over Time** (line chart)
- Filter: Log Source = BlueCoat_ProxySG
- Metric: Message count
- Interval: 5 minutes

**Top Blocked URLs** (table)
- Filter: Classification = "Blocked"
- Group by: Object (URL)

**Top Users by Traffic** (bar chart)
- Group by: Login
- Metric: Count

**Threat Detections** (table)
- Filter: cs-threat-risk > 0 OR x-virus-id != null
- Show: Time, User, URL, Threat

**Authentication Failures** (count)
- Filter: Event = "Authentication Failure"

**SSL Certificate Errors** (table)
- Filter: x-rs-certificate-validate-status != "ok"

3. Save dashboard

---

#### Step 28: Create Alarm for Log Gap Detection

1. Navigate to **Alarms** → **New Alarm**
2. Configure:

```yaml
Name: BlueCoat - Log Reception Gap
Description: No logs received from ProxySG in 10 minutes
Rule Type: Filter
Log Source: BlueCoat_ProxySG_Production
Time Window: 10 minutes
Threshold: < 1 log message
Severity: High
```

3. **Actions**:
   - Send email to: soc@company.com
   - Create case
   - Send notification

4. Click **Save**

---

#### Step 29: Create Alarm for Threat Detection

1. Create new alarm:

```yaml
Name: BlueCoat - Threat Detected
Rule Type: Filter
Filter: x-virus-id != null OR cs-threat-risk > 7
Severity: Critical
Actions: Email, Create Case, SNMP Trap
```

2. Click **Save**

---

#### Step 30: Create Alarm for Configuration Changes

1. Create new alarm:

```yaml
Name: BlueCoat - Admin Configuration Change
Rule Type: Filter
Filter: Log Message Type = "Admin Event"
    AND (Message contains "configuration" OR "policy")
Severity: Medium
Actions: Email admin team
```

2. Click **Save**

---

### Phase 12: Documentation & Maintenance

#### Step 31: Document Baseline

**Record the following:**

```yaml
Expected Log Volume:
  Average logs per hour: ___________
  Peak logs per hour: ___________
  Daily total: ___________

Log Sources Configured:
  - Main access log: ✓
  - Security events: ✓
  - Admin events: ✓
  - Health events: ✓
  - Attack detection: ✓
  - SSL events: ✓

Network Information:
  ProxySG IP: ___________
  LogRhythm IP: ___________
  Syslog Port: 514/TCP
  Backup log location: /var/log/access
```

---

#### Step 32: Create Maintenance Checklist

**Daily:**
- [ ] Verify logs flowing (check dashboard)
- [ ] Review any alarms triggered
- [ ] Check syslog connection status

**Weekly:**
- [ ] Compare ProxySG vs LogRhythm log counts
- [ ] Review local backup log size on ProxySG
- [ ] Check for parsing errors in LogRhythm

**Monthly:**
- [ ] Review and tune MPE parsing rules
- [ ] Update dashboard based on new requirements
- [ ] Test log flow after maintenance windows
- [ ] Review and optimize alarm thresholds

---

#### Step 33: Create Runbook for Log Gaps

**If logs stop flowing:**

1. **Check ProxySG syslog status**:
   - Statistics → Syslog → LogRhythm_SIEM
   - Check connection status

2. **Check network connectivity**:
   ```bash
   # From ProxySG CLI
   telnet <LogRhythm_IP> 514
   ```

3. **Check LogRhythm syslog service**:
   - Verify Syslog Server service running
   - Check Windows Event Log for errors

4. **Check firewall rules**:
   - Verify ProxySG can reach LogRhythm:514/TCP

5. **Review local backup logs**:
   - ProxySG: `/var/log/access`
   - Manually upload if needed

6. **Check LogRhythm disk space**:
   - Ensure sufficient space for log ingestion

---

## ✅ Verification Checklist

### Complete Visibility Checklist

**Web Traffic Logs**
- [ ] HTTP requests visible
- [ ] HTTPS requests visible
- [ ] All URLs captured (scheme, host, path, query)
- [ ] User-Agent strings captured
- [ ] Referer information captured

**Security Events**
- [ ] Blocked/denied requests logged
- [ ] URL category information present
- [ ] Threat risk scores visible
- [ ] Malware detections logged
- [ ] IPS/Attack detection events logged

**Authentication & Authorization**
- [ ] Usernames captured
- [ ] Authentication realm logged
- [ ] Authentication failures logged
- [ ] Authorization failures logged
- [ ] Authentication mechanism logged

**SSL/TLS Information**
- [ ] Certificate validation status
- [ ] Certificate errors logged
- [ ] Cipher suite information
- [ ] SSL handshake failures
- [ ] OCSP errors captured

**Application Control**
- [ ] Application names detected
- [ ] Application operations logged
- [ ] Application-layer protocols identified

**Bandwidth & Performance**
- [ ] Bytes sent/received logged
- [ ] Response times captured
- [ ] Time-taken for transactions
- [ ] DNS lookup time
- [ ] Connection time

**Administrative Events**
- [ ] Configuration changes logged
- [ ] Admin login/logout events
- [ ] Policy installations logged
- [ ] System events captured

**Health & System Events**
- [ ] System health warnings
- [ ] Service failures logged
- [ ] Resource threshold alerts

**Advanced Features**
- [ ] Transaction UUIDs for correlation
- [ ] Client IP reputation scores
- [ ] Geographic location data
- [ ] Exception IDs for troubleshooting

---

## 🔧 Troubleshooting

### Issue: Logs not appearing in LogRhythm

**Symptoms:**
- Log Message Viewer shows no ProxySG logs
- Log count in dashboard is zero

**Troubleshooting Steps:**

1. **Check ProxySG syslog status**
   ```plaintext
   Navigation: Statistics → Syslog
   Look for: Connection Status = Connected
   Check: Messages Sent > 0
   Check: Messages Failed = 0
   ```

2. **Test network connectivity**
   ```bash
   # From ProxySG CLI
   telnet <LogRhythm_IP> 514
   
   # Expected: Connection successful
   # If fails: Check firewall rules
   ```

3. **Verify LogRhythm syslog service**
   ```plaintext
   Windows Services → LogRhythm Syslog Server
   Status should be: Running
   ```

4. **Check firewall rules**
   - ProxySG → LogRhythm on port 514/TCP must be allowed
   - Check both inline firewalls and host-based firewalls

5. **Review LogRhythm log source**
   - Deployment Manager → Log Sources
   - Verify: Record Status = Enabled
   - Check: Log Messages Received count

---

### Issue: Fields not parsing correctly

**Symptoms:**
- Logs visible but fields empty or incorrect
- Source IP, usernames, URLs not extracted

**Troubleshooting Steps:**

1. **Verify log format**
   ```plaintext
   ProxySG: Configuration → Access Logging → Formats
   Ensure: LogRhythm_Complete_Format is selected
   Check: All 50+ fields configured
   ```

2. **Check MPE policy**
   ```plaintext
   LogRhythm: MPE Rules → Log Message Processing Policies
   Verify: Blue Coat policy assigned to log source
   Test: Use MPE debugger with sample logs
   ```

3. **Review sample logs**
   ```plaintext
   LogRhythm: Log Message Viewer
   View: Raw log message
   Compare: Expected format vs actual format
   ```

4. **Test regex patterns**
   - Use LogRhythm MPE debugger
   - Test field extraction with sample logs
   - Adjust regex patterns as needed

---

### Issue: High log volume causing issues

**Symptoms:**
- LogRhythm storage filling quickly
- Slow query performance
- Log indexing lag

**Solutions:**

1. **Increase message queue size**
   ```plaintext
   ProxySG: Configuration → Access Logging → Syslog
   Set: Message Queue Size = 100000
   ```

2. **Add more LogRhythm storage**
   - Expand storage volumes
   - Configure tiered storage (hot/warm/cold)

3. **Optimize parsing rules**
   - Remove unnecessary field extractions
   - Use more efficient regex patterns

4. **Consider log filtering** (last resort)
   - Filter out non-critical log types
   - Reduce verbosity of policy trace logging

5. **Implement log compression**
   ```plaintext
   ProxySG: Configuration → Access Logging → Logs
   Enable: Compress old logs
   ```

---

### Issue: Intermittent log gaps

**Symptoms:**
- Logs stop flowing periodically
- Gaps in log timeline
- Syslog connection drops

**Solutions:**

1. **Enable persistent connections**
   ```plaintext
   ProxySG: Configuration → Access Logging → Syslog
   Set: Persistent Connection = Yes
   Set: Reconnect Interval = 30 seconds
   ```

2. **Increase connection timeout**
   ```plaintext
   Connection Timeout = 60 seconds
   ```

3. **Configure local backup**
   ```plaintext
   Configuration → Access Logging → Logs
   Enable: Local file logging
   Set: Max files to retain = 50
   ```

4. **Monitor network stability**
   - Check for packet loss between ProxySG and LogRhythm
   - Monitor firewall session timeouts
   - Review network device logs

---

## 📚 Best Practices

### Security

1. **Use TCP over UDP** for syslog (guaranteed delivery)
2. **Enable TLS encryption** for syslog transmission (if supported)
3. **Restrict LogRhythm access** using RBAC
4. **Rotate local backup logs** regularly to prevent disk exhaustion
5. **Monitor for unauthorized configuration changes**

### Performance

1. **Baseline normal log volume** before and after configuration
2. **Monitor ProxySG CPU/memory** usage after enabling comprehensive logging
3. **Use log buffering** to handle traffic spikes
4. **Optimize MPE parsing rules** for efficiency
5. **Implement tiered storage** in LogRhythm (hot/warm/cold)

### Reliability

1. **Enable local backup logging** on ProxySG
2. **Configure health monitoring** for syslog connections
3. **Set up alerting** for log gaps and connection failures
4. **Test failover scenarios** periodically
5. **Document and test rollback procedures**

### Compliance

1. **Define log retention policy** based on regulatory requirements
2. **Enable audit logging** for all administrative actions
3. **Implement log integrity** controls (if required)
4. **Review and document** data privacy considerations
5. **Test restore procedures** from backups

### Operational

1. **Create runbooks** for common issues
2. **Schedule regular validation** of log completeness
3. **Review and tune alarms** quarterly
4. **Train team members** on troubleshooting procedures
5. **Maintain up-to-date documentation**

---

## ❓ FAQ

### Q: Do I need to use all 50+ fields in the log format?

**A:** Yes, for complete visibility. However, you can start with a subset and add fields incrementally. The comprehensive format ensures no activity is missed.

---

### Q: What's the impact on ProxySG performance?

**A:** Comprehensive logging typically adds 5-10% CPU usage and 5-15% memory usage. The impact is minimal on modern ProxySG appliances with adequate resources.

---

### Q: Should I use TCP or UDP for syslog?

**A:** Always use **TCP** for production environments. TCP guarantees delivery and provides connection state awareness. UDP may lose logs during network congestion.

---

### Q: How much storage do I need in LogRhythm?

**A:** Calculate using: `(Daily Requests × Avg Log Size × Retention Days) / 1024`

Example: 1M requests/day × 1KB × 90 days = ~85 GB

Plan for 20-30% growth buffer.

---

### Q: Can I filter certain log types to reduce volume?

**A:** Yes, but not recommended for security/compliance environments. If needed, filter verbose logs like policy trace while keeping all security-relevant logs.

---

### Q: How do I handle log gaps during maintenance?

**A:** Enable local backup logging on ProxySG. Logs are stored locally and automatically re-transmitted when syslog connection is restored.

---

### Q: What if LogRhythm doesn't have a native Blue Coat parser?

**A:** Use the "Flat File - Syslog" log source type and create a custom MPE policy. See [Step 20](#step-20-configure-custom-mpe-policy-if-needed) for details.

---

### Q: How often should I validate log completeness?

**A:** 
- **Daily**: First week after deployment
- **Weekly**: First month
- **Monthly**: Ongoing

Always validate after maintenance windows or configuration changes.

---

### Q: Can I send logs to multiple SIEM systems?

**A:** Yes! Create additional syslog destinations in ProxySG and link the same access log to multiple destinations.

---

### Q: What's the recommended syslog port?

**A:** Default is 514/TCP. Some organizations use custom ports (1514, 6514) for security. Match your LogRhythm configuration.

---

## 🤝 Contributing

We welcome contributions! Here's how you can help:

### Ways to Contribute

- 🐛 **Report bugs** or issues you encounter
- 💡 **Suggest enhancements** or new features
- 📝 **Improve documentation** with clarifications or examples
- 🧪 **Share test results** from different ProxySG/LogRhythm versions
- 🎨 **Add diagrams** or visual aids
- 📊 **Contribute dashboards** or alarm templates

### Contribution Guidelines

1. **Fork** the repository
2. **Create a branch** for your feature (`git checkout -b feature/amazing-feature`)
3. **Commit your changes** (`git commit -m 'Add amazing feature'`)
4. **Push to branch** (`git push origin feature/amazing-feature`)
5. **Open a Pull Request**

### Code of Conduct

Please be respectful and constructive. We're all here to help each other succeed.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🆘 Support

### Community Support

- **GitHub Issues**: [Open an issue](https://github.com/yourusername/bluecoat-logrhythm/issues)
- **Discussions**: [Join the discussion](https://github.com/yourusername/bluecoat-logrhythm/discussions)
- **Wiki**: [Browse the wiki](https://github.com/yourusername/bluecoat-logrhythm/wiki)

### Vendor Support

- **Broadcom (Blue Coat)**: [Support Portal](https://support.broadcom.com)
- **LogRhythm**: [Support Portal](https://community.logrhythm.com)

### Professional Services

For professional implementation assistance, contact your Broadcom or LogRhythm representative.

---

## 🙏 Acknowledgments

- Broadcom/Symantec for Blue Coat ProxySG documentation
- LogRhythm for SIEM platform and support materials
- Community contributors and testers
- Security and compliance teams who provided feedback

---

## 📊 Project Stats

![GitHub stars](https://img.shields.io/github/stars/yourusername/bluecoat-logrhythm?style=social)
![GitHub forks](https://img.shields.io/github/forks/yourusername/bluecoat-logrhythm?style=social)
![GitHub issues](https://img.shields.io/github/issues/yourusername/bluecoat-logrhythm)
![GitHub pull requests](https://img.shields.io/github/issues-pr/yourusername/bluecoat-logrhythm)
![Last commit](https://img.shields.io/github/last-commit/yourusername/bluecoat-logrhythm)

---

## 🗺️ Roadmap

### Completed ✅
- [x] Complete configuration guide
- [x] Pre-flight checklist
- [x] Troubleshooting section
- [x] Verification procedures

### In Progress 🚧
- [ ] LogRhythm dashboard templates (JSON)
- [ ] MPE policy examples
- [ ] Automated testing scripts
- [ ] Docker test environment

### Planned 📅
- [ ] Splunk integration guide
- [ ] QRadar integration guide
- [ ] Elasticsearch integration guide
- [ ] Automated deployment scripts (Ansible)
- [ ] Prometheus exporter for metrics
- [ ] Grafana dashboards

---

## 📞 Contact

**Project Maintainer**: Your Name
- GitHub: [@yourusername](https://github.com/yourusername)
- Email: your.email@example.com
- LinkedIn: [Your Profile](https://linkedin.com/in/yourprofile)

---

<div align="center">

**⭐ Star this repo if you found it helpful!**

**Made with ❤️ by the community**

[Back to Top](#blue-coat-proxysg-to-logrhythm-siem-integration-guide)

</div>