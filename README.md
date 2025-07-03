# 🛡️ Durgaai Secure Communications  
**Secure Agent Communication System Built On Asterisk PBX**

Durgaai Secure Communications Is A Purpose-Built, Encrypted Communication Platform Designed For Agents, Field Operatives, And Command Centers. It Features End-To-End Encrypted Voice Calling, Secure Messaging, Mission-Specific Conferencing, And A Hardened SIP Infrastructure — All Built On Top Of Asterisk PBX.

---

## 🚀 Features At A Glance

- 🔐 **Encrypted Voice Communication (SRTP + TLS)**
- 🗨️ **Secure Instant Messaging With Logging**
- 📞 **Multi-Agent Conferencing (With PIN & Emergency Access)**
- 🎯 **Self-Destructing Mission Messages**
- 📼 **Voicemail Support For Each Agent**
- 🔔 **Broadcast Alerts To All Field Agents**
- 🛠️ **Role-Based Configuration (Agents, Command Center)**
- 🎧 **Custom Audio Cues For Immersive Experience**

---

## 🧱 System Architecture

```
+-----------------+           +------------------+
| Field Agents    |           | Command Center   |
| Linphone Clients|           | SIP Console UI   |
+--------+--------+           +--------+---------+
         |                             |
         |     Secure SIP/RTP (TLS)    |
         v                             v
+-----------------------------------------------+
|             ASTERISK SERVER                   |
|-----------------------------------------------|
| SIP Proxy (PJSIP)   |  Dialplan Routing       |
| Secure Messaging    |  Conference Bridging    |
| Voicemail           |  Audio Playback         |
| Custom Recordings   |  Agent Logic            |
+-----------------------------------------------+
         |                             |
         |     Custom Sounds & Logs    |
         v                             v
 /var/lib/asterisk/sounds/custom/   /var/log/asterisk/
```

> Core: Asterisk PBX  
> Transport: UDP, TLS  
> Media: RTP, SRTP  
> Messaging: PJSIP Messaging Contexts  
> Access Control: Role-Based Endpoints (Agents, Command)

---

## 📂 Setup Guide

### 1. 🔧 Ubuntu Server Preparation
Follow [Ubuntu Server Setup](UBUNTU_SERVER_SETUP.md) For Installing Essential Packages.

### 2. 🛠️ Asterisk Installation & Configuration
Use [SIP Server Configuration](SIP_SERVER_CONF.md) For Setting Up:

- Agent Extensions (PJSIP)
- Dialplan Logic
- Secure Messaging Routes
- Voicemail Configuration
- Conference Bridges

### 3. 🔥 Security Best Practices
Please Review The [Security Policy](SECURITY_POLICY.txt) To Harden Your Deployment:
- UFW/Fail2Ban Setup
- Physical Isolation
- Port Lockdowns
- Password Rotation

### 4. 📜 Legal Compliance
This System Is For Authorized Personnel Only. All Interactions May Be Logged.  
Full Legal Disclosures Are In [Legal Notice](LEGAL_NOTICE.txt).

---

## 🧑‍💼 Agent Quick Reference

| Role            | Extension   | Notes                        |
|-----------------|-------------|------------------------------|
| Agent Alpha     | `10765432`  | Full Messaging + Voice      |
| Agent Beta      | `24567890`  | Full Messaging + Voice      |
| Command Center  | `99887766`  | All Access + Broadcast       |
| Secure Conf     | `80085000`  | PIN: `4721`                  |
| Emergency Conf  | `80086000`  | No PIN                      |
| Voicemail       | `82000000`  | PIN: `5309`                 |
| Mission Msg     | `86000000`  | Self-Destructing Message     |
| Broadcast Alert | `88000000`  | Message All Agents           |

---

## 🧪 Testing & Verification

### Call Flow Test
```
Agent A → Asterisk PBX → Agent B  
✓ Voice Encrypted ✓ Call Logged ✓ RTP Secure
```

### Messaging Test
```
Agent → Asterisk → Agent  
✓ PJSIP Message Routing ✓ Audit Logs ✓ Messaging Context Active
```

---

## 🧰 Maintenance Commands

```bash
# Asterisk CLI
sudo asterisk -rvvv

# Reload Configuration
dialplan reload
pjsip reload

# Monitor
core show channels
pjsip list endpoints
```

---

## 🧱 System Design Overview

```
+-----------------+           +------------------+
| Field Agents    |           | Command Center   |
| Linphone Clients|           | SIP Console UI   |
+--------+--------+           +--------+---------+
         |                             |
         |     Secure SIP/RTP (TLS)    |
         v                             v
+-----------------------------------------------+
|             ASTERISK SERVER                   |
|-----------------------------------------------|
| SIP Proxy (PJSIP)   |  Dialplan Routing       |
| Secure Messaging    |  Conference Bridging    |
| Voicemail           |  Audio Playback         |
| Custom Recordings   |  Agent Logic            |
+-----------------------------------------------+
         |                             |
         |     Custom Sounds & Logs    |
         v                             v
 /var/lib/asterisk/sounds/custom/   /var/log/asterisk/
```

---

### 3. Security Configuration
Review [`SECURITY_POLICY.txt`](SECURITY_POLICY.txt) For Security Hardening Practices:

- Fail2Ban, UFW, TLS, Password Rotation
- Disable Root Access, Lockouts, Local Network Binding

---


## 🧪 Call & Messaging Flows

### 📞 Basic Call Flow
```
Agent A                Asterisk                  Agent B
   |---- INVITE ------->|                          |
   |                    |---- INVITE ------------->|
   |<--- 180 Ringing ---|                          |
   |<--- 200 OK --------|                          |
   |---- ACK ---------->|                          |
   |<== Two-Way Audio ==>                          |
   |---- BYE ---------->|                          |
   |                    |---- BYE ---------------->|
```

### 💬 Secure Message Flow
```
Agent A                Asterisk                  Agent B
   |---- MESSAGE ------>|                          |
   |<--- 202 Accepted --|                          |
   |                    |---- MESSAGE ----------->|
   |                    |<--- 200 OK -------------|
```

---

## 🎨 Customization

### 🎧 Custom Voice Prompts
- Located In: `/var/lib/asterisk/sounds/custom`
- Tools: `espeak`, `sox`
- Examples: `agent-connect.wav`, `commandcenter-connect.wav`, `emergency-conference.wav`

### 🔐 Add New Agents
Update:
- `pjsip.conf`
- `voicemail.conf`
- `extensions.conf`

---

## 🔐 Security Policy Highlights

- Use On Secured LAN / VPN Only  
- Never Expose SIP Port Publicly  
- Use Fail2Ban + Firewall  
- Disable Root SSH  
- Perform Regular OS & Asterisk Updates

Read Full Policy : [SECURITY_POLICY.txt](SECURITY_POLICY.txt)

---

## 🧾 Legal & Jurisdiction

Durgaai Secure Communications Is Protected By Copyright And Trademark.  
Operated Under The Jurisdiction Of India.

See [LEGAL_NOTICE.txt](LEGAL_NOTICE.txt) For Full Disclosures.  
Inquiries : [legal@durgaaisolutions.com](mailto:legal@durgaaisolutions.com)

---

## 👨‍💻 Contribution

This Project Is Built For Internal Use But May Be Open-Sourced On Request.  
Contact: [security@durgaaisolutions.com](mailto:security@durgaaisolutions.com)
