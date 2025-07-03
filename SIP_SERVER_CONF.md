# SECRET AGENT COMMUNICATION SYSTEM
## Asterisk PBX Configuration Guide - CORRECTED VERSION

### 1. System Setup and Installation

```
sudo apt update && sudo apt upgrade -y

sudo apt install -y build-essential git wget subversion \
  libjansson-dev libxml2-dev uuid-dev libncurses5-dev \
  libssl-dev libedit-dev libsqlite3-dev libsrtp2-dev \
  libspeexdsp-dev libcurl4-openssl-dev libopus-dev \
  libavcodec-dev libavformat-dev libswscale-dev \
  libvpx-dev libspandsp-dev python3-dev lua5.3 \
  liblua5.3-dev libnewt-dev libusb-1.0-0-dev \
  libgmime-3.0-dev libical-dev libneon27-dev \
  libiksemel-dev liburiparser-dev libpq-dev \
  libiksemel-utils cmake cmake-curses-gui \
  libunbound-dev unixodbc-dev uuid uuid-dev
```

### 2. Asterisk Installation

```
cd /usr/src
sudo wget https://downloads.asterisk.org/pub/telephony/asterisk/asterisk-22-current.tar.gz
sudo tar -xvf asterisk-22-current.tar.gz
cd asterisk-22.3.0

# Install PreRequisites
sudo contrib/scripts/install_prereq install

# Configure
sudo ./configure
sudo make menuselect

# In menuselect, Ensure These Are Enabled :
# - res_srtp
# - codec_opus
# - app_voicemail
# - res_pjsip
# - res_pjsip_messaging
# - app_sms
# - res_crypto

# Compile And Install
make -j$(nproc)
sudo make install
sudo make samples
sudo make config
sudo ldconfig
```

### 3. Agent Communication System Configuration

#### 3.1 PJSIP Configuration (/etc/asterisk/pjsip.conf)

```
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0

; SPECIAL AGENT : ALPHA
[10765432]
type=endpoint
context=agents
disallow=all
allow=ulaw,alaw,vp8,h264,opus
auth=10765432-auth
aors=10765432
send_rpid=yes
trust_id_outbound=yes
direct_media=no
allow_subscribe=yes
message_context=encrypt_msgs
outbound_auth=10765432-auth
callerid="Agent Alpha" <10765432>
device_state_busy_at=1
user_eq_phone=yes
rtp_symmetric=yes
rewrite_contact=yes
force_rport=yes

[10765432-auth]
type=auth
auth_type=userpass
username=10765432
password=cqF7!k9#pRt6

[10765432]
type=aor
max_contacts=5
qualify_frequency=60

; SPECIAL AGENT : BETA
[24567890]
type=endpoint
context=agents
disallow=all
allow=ulaw,alaw,vp8,h264,opus
auth=24567890-auth
aors=24567890
send_rpid=yes
trust_id_outbound=yes
direct_media=no
allow_subscribe=yes
message_context=encrypt_msgs
outbound_auth=24567890-auth
callerid="Agent Beta" <24567890>
device_state_busy_at=1
user_eq_phone=yes
rtp_symmetric=yes
rewrite_contact=yes
force_rport=yes

[24567890-auth]
type=auth
auth_type=userpass
username=24567890
password=mNb6!v9@zXc3

[24567890]
type=aor
max_contacts=5
qualify_frequency=60

; COMMAND CENTER
[99887766]
type=endpoint
context=commandcenter
disallow=all
allow=ulaw,alaw,vp8,h264,opus
auth=99887766-auth
aors=99887766
send_rpid=yes
trust_id_outbound=yes
direct_media=no
allow_subscribe=yes
message_context=encrypt_msgs
outbound_auth=99887766-auth
callerid="Command Center" <99887766>
device_state_busy_at=1
user_eq_phone=yes
rtp_symmetric=yes
rewrite_contact=yes
force_rport=yes

[99887766-auth]
type=auth
auth_type=userpass
username=99887766
password=cM$8pW@nT!5z

[99887766]
type=aor
max_contacts=5
qualify_frequency=60
```

#### 3.2 Dialplan Configuration (/etc/asterisk/extensions.conf)

```
[agents]
; Standard Agent-To-Agent Calls
exten => _10XXXXXX,1,Answer()
 same => n,Wait(0.5)
 same => n,Playback(custom/durgaaisolutions-welcome)
 same => n,Playback(custom/agent-connect)
 same => n,Dial(PJSIP/${EXTEN},30)
 same => n,Voicemail(${EXTEN}@agents,u)
 same => n,Hangup()

exten => _24XXXXXX,1,Answer()
 same => n,Wait(0.5)
 same => n,Playback(custom/durgaaisolutions-welcome)
 same => n,Playback(custom/agent-connect)
 same => n,Dial(PJSIP/${EXTEN},30)
 same => n,Voicemail(${EXTEN}@agents,u)
 same => n,Hangup()

; Command Center
exten => _99XXXXXX,1,Answer()
 same => n,Wait(0.5)
 same => n,Playback(custom/durgaaisolutions-welcome)
 same => n,Playback(custom/commandcenter-connect)
 same => n,Dial(PJSIP/${EXTEN},30)
 same => n,Voicemail(${EXTEN}@commandcenter,u)
 same => n,Hangup()

; Conference Room With PIN code
exten => 80085000,1,Answer()
 same => n,Playback(custom/secure-conference)
 same => n,Authenticate(4721)
 same => n,ConfBridge(missionconf,default_bridge,default_user)

; Emergency Conference (no PIN)
exten => 80086000,1,Answer()
 same => n,Playback(custom/emergency-conference)
 same => n,ConfBridge(emergency,default_bridge,default_user)

; Voicemail Access
exten => 82000000,1,Answer()
 same => n,Authenticate(5309)
 same => n,VoiceMailMain(@agents)
 same => n,Hangup()

; Self-Destructing Messages
exten => 86000000,1,Answer()
 same => n,Playback(custom/self-destruct-message)
 same => n,Record(/tmp/mission-${CALLERID(num)}-${EPOCH}.wav,3,30)
 same => n,Playback(/tmp/mission-${CALLERID(num)}-${EPOCH})
 same => n,Read(target,custom/enter-agent-id,8)
 same => n,System(echo "${EPOCH}: Agent ${CALLERID(num)} Sent Message To ${target}" >> /var/log/asterisk/selfdestmsgs.log)
 same => n,Dial(PJSIP/${target},5)
 same => n,Playback(/tmp/mission-${CALLERID(num)}-${EPOCH})
 same => n,System(rm /tmp/mission-${CALLERID(num)}-${EPOCH}.wav)
 same => n,Hangup()

[commandcenter]
; Command Center Can Call Any Agent
exten => _XXXXXXXX,1,Answer()
 same => n,Wait(0.5)
 same => n,Playback(custom/durgaaisolutions-welcome)
 same => n,Playback(custom/commandcenter-initiated)
 same => n,Dial(PJSIP/${EXTEN},45)
 same => n,Voicemail(${EXTEN}@agents,u)
 same => n,Hangup()

; CommandCenter Can Initiate Group Calls
exten => 88000000,1,Answer()
 same => n,Playback(custom/all-agents-alert)
 ; Corrected Page application with proper syntax
 same => n,Page(PJSIP/10765432&PJSIP/24567890,i,120)
 same => n,Hangup()

[encrypt_msgs]
; Secure Messaging Between Agents
exten => _10XXXXXX,1,NoOp(Secure message from ${MESSAGE(from)} to ${EXTEN})
 same => n,Set(CDR(userfield)=SecMsg)
 ; Corrected MessageSend syntax with proper parameters
 same => n,MessageSend(pjsip:${EXTEN},${MESSAGE(from)},${MESSAGE(body)})
 same => n,System(echo "${EPOCH}: SECURE MSG: ${MESSAGE(from)} -> ${EXTEN}" >> /var/log/asterisk/messages.log)
 same => n,Hangup()

exten => _24XXXXXX,1,NoOp(Secure message from ${MESSAGE(from)} to ${EXTEN})
 same => n,Set(CDR(userfield)=SecMsg)
 same => n,MessageSend(pjsip:${EXTEN},${MESSAGE(from)},${MESSAGE(body)})
 same => n,System(echo "${EPOCH}: SECURE MSG: ${MESSAGE(from)} -> ${EXTEN}" >> /var/log/asterisk/messages.log)
 same => n,Hangup()

exten => _99XXXXXX,1,NoOp(CommandCenter message from ${MESSAGE(from)} to ${EXTEN})
 same => n,Set(CDR(userfield)=CmdMsg)
 same => n,MessageSend(pjsip:${EXTEN},${MESSAGE(from)},${MESSAGE(body)})
 same => n,System(echo "${EPOCH}: COMMANDCENTER MSG: ${MESSAGE(from)} -> ${EXTEN}" >> /var/log/asterisk/messages.log)
 same => n,Hangup()

; Broadcast Message To All Agents
exten => 88000000,1,NoOp(BROADCAST from ${MESSAGE(from)})
 same => n,MessageSend(pjsip:10765432,${MESSAGE(from)},${MESSAGE(body)})
 same => n,MessageSend(pjsip:24567890,${MESSAGE(from)},${MESSAGE(body)})
 same => n,System(echo "${EPOCH}: BROADCAST: ${MESSAGE(from)} -> ALL" >> /var/log/asterisk/messages.log)
 same => n,Hangup()
```

#### 3.3 Voicemail Configuration (/etc/asterisk/voicemail.conf)

```
[general]
format=wav49|wav
maxmsg=100
maxsecs=180
minsecs=3
maxgreet=60
skipms=3000
maxlogins=3
review=yes
saycid=yes
sendvoicemail=yes
forcename=yes
forcegreetings=yes
hidefromdir=no
operator=yes
envelope=yes
sayduration=yes
saydurationm=1
pagerfromstring=Command
pagersubject=URGENT MESSAGE
pagerbody=New message from Agent ${VM_CALLERID}
nextaftercmd=yes

[agents]
10765432 => 1234,Agent Alpha
24567890 => 3456,Agent Beta

[commandcenter]
99887766 => 9999,Command Center
```

#### 3.4 Custom Sounds

Create Custom Audio Files For Your Secret Agent Experience:

```
# Create Directory For Custom Sounds
sudo mkdir -p /var/lib/asterisk/sounds/custom
cd /var/lib/asterisk/sounds/custom

# Create agent-connect Sound
espeak -v en-us -s 130 "Agent Connection Established. Line Secured." -w /tmp/agent-connect.wav
sox /tmp/agent-connect.wav -r 8000 -c 1 agent-connect.wav

# Create commandcenter-connect Sound
espeak -v en-us -s 130 "Command Center Connection Established. Priority Channel Activated." -w /tmp/commandcenter-connect.wav
sox /tmp/commandcenter-connect.wav -r 8000 -c 1 commandcenter-connect.wav

# Create secure-conference Sound
espeak -v en-us -s 130 "Secure Conference Initiated. Please Enter Your Access Code." -w /tmp/secure-conference.wav
sox /tmp/secure-conference.wav -r 8000 -c 1 secure-conference.wav

# Create emergency-conference Sound
espeak -v en-us -s 130 "Emergency Conference Protocol Activated. All Agents Standby." -w /tmp/emergency-conference.wav
sox /tmp/emergency-conference.wav -r 8000 -c 1 emergency-conference.wav

# Create self-destruct-message Sound
espeak -v en-us -s 130 "Self-Destructing Message System. Record After The Tone." -w /tmp/self-destruct-message.wav
sox /tmp/self-destruct-message.wav -r 8000 -c 1 self-destruct-message.wav

# Create enter-agent-id Sound
espeak -v en-us -s 130 "Enter Target Agent Identification Number." -w /tmp/enter-agent-id.wav
sox /tmp/enter-agent-id.wav -r 8000 -c 1 enter-agent-id.wav

# Create commandcenter-initiated Sound
espeak -v en-us -s 130 "Command Center Initiated Contact. Please Stand By." -w /tmp/commandcenter-initiated.wav
sox /tmp/commandcenter-initiated.wav -r 8000 -c 1 commandcenter-initiated.wav

# Create all-agents-alert Sound
espeak -v en-us -s 130 "Attention All Field Agents. This Is A Priority Broadcast." -w /tmp/all-agents-alert.wav
sox /tmp/all-agents-alert.wav -r 8000 -c 1 all-agents-alert.wav

# Create durgaaisolutions-welcome Sound
espeak -v en-us -s 130 "Welcome To Durgaai Secure Communications." -w /tmp/durgaaisolutions-welcome.wav
sox /tmp/durgaai-welcome.wav -r 8000 -c 1 /var/lib/asterisk/sounds/custom/durgaaisolutions-welcome.wav


# Set Correct Permissions
sudo chown -R asterisk:asterisk /var/lib/asterisk/sounds/custom
sudo chmod -R 644 /var/lib/asterisk/sounds/custom/*.wav
```


#### 3.5 Firewall Configuration

```
sudo ufw allow 5060/udp
sudo ufw allow 5061/tcp  # For TLS
sudo ufw allow 10000:20000/udp
sudo ufw reload
```


## 4. Configuration Files

### Create And Configure Necessary Files

#### rtp.conf
```bash
sudo nano /etc/asterisk/rtp.conf
```
Add:
```
[general]
rtpstart=10000
rtpend=20000
icesupport=no
```

#### confbridge.conf
```bash
sudo nano /etc/asterisk/confbridge.conf
```
Add:
```
[default_bridge]
type=bridge
max_members=10
record_conference=yes
record_file=/var/spool/asterisk/monitor/confbridge-${CONFBRIDGE_NAME}-${UNIQUEID}
video_mode=follow_talker

[default_user]
type=user
announce_join_leave=yes
music_on_hold_when_empty=yes
music_on_hold_class=default
announce_user_count=yes
announce_user_count_all=yes
```

#### sip_notify.conf for Device Control
```bash
sudo nano /etc/asterisk/sip_notify.conf
```
Add:
```
[polycom-check-cfg]
Event=>check-sync
Content-Length=>0

[cisco-restart]
Event=>check-sync
Content-Length=>0

[snom-reboot]
Event=>reboot
Content-Length=>0

[linphone-reboot]
Event=>reboot
Content-Length=>0
```

#### messaging.conf
```bash
sudo nano /etc/asterisk/messaging.conf
```
Add:
```
[general]
disable_system_mi=no

[pjsip]
accept=yes
default_from=commandcenter@localhost
```

## 7. Network Configuration

### Set up logging directories
```bash
sudo mkdir -p /var/log/asterisk
sudo touch /var/log/asterisk/selfdestmsgs.log
sudo touch /var/log/asterisk/messages.log
sudo chown -R asterisk:asterisk /var/log/asterisk
```

## 8. Quick Reference Guide for Agents

### Important Numbers
- **Agent Alpha**: 10765432
- **Agent Beta**: 24567890
- **Command Center**: 99887766
- **Secure Conference Room**: 80085000 (PIN: 4721)
- **Emergency Conference**: 80086000 (No PIN)
- **Voicemail Access**: 82000000 (PIN: 5309)  
- **Self-Destructing Messages**: 86000000
- **All Agents Alert**: 88000000


### Log Locations
- Message Logs: `/var/log/asterisk/messages.log`
- Secret Mission Logs: `/var/log/asterisk/selfdestmsgs.log`
- Call Detail Records: `/var/log/asterisk/cdr-csv/Master.csv`
- General Logs: `/var/log/asterisk/full`


### 9. CLI Management Commands

#### 9.1 Start/Stop/Restart Asterisk

```
# Start Asterisk
sudo systemctl start asterisk

# Stop Asterisk
sudo systemctl stop asterisk

# Restart Asterisk
sudo systemctl restart asterisk

# Enable Asterisk to start at boot
sudo systemctl enable asterisk

# Enter the Asterisk CLI with verbose output
sudo asterisk -rvvv
```

#### 9.2 Agent Management via CLI

```
# List All Connected Agents
pjsip list endpoints

# Show Status Of A Specific Agent
pjsip show endpoint 10765432

# Show All Registrations
pjsip show registrations

# Remove/Force Logout A Specific Agent
pjsip set contact status endpoint/10765432/sip:10765432@192.168.1.100:5060 Unavailable

# Monitor Calls In Progress
core show channels

# Eavesdrop On A Call (Replace PJSIP/10765432-00000001 With Actual Channel)
chanspy PJSIP/10765432-00000001
```

#### 9.3 Configuration Reload Commands

```
# Reload All Configurations
core reload

# Reload Just PJSIP
pjsip reload

# Reload Dialplan
dialplan reload

# Check Dialplan Syntax
dialplan show

# Reload Voicemail
voicemail reload
```

#### 9.4 Messaging and Logs

```
# View Message Logs
!tail -f /var/log/asterisk/messages.log

# View Call Logs
!tail -f /var/log/asterisk/cdr-csv/Master.csv

# Monitor SIP Transactions
pjsip set logger on

# View Active Message Sessions
message show sessions
```

### 10. Linphone Client Configuration

1. Download And Install Linphone On Agents' Devices
2. Configure New SIP Account With:
   - SIP Address: 10765432@your-server-ip (Substitute Your Actual Agent Number)
   - Password: As Configured In pjsip.conf
   - Transport: UDP or TLS (preferred for security)
   - Proxy: your-server-ip:5060 (for UDP) or your-server-ip:5061 (for TLS)
3. Enable Messaging In Account Settings
4. For Secure Calls, Enable SRTP and ZRTP in the Security Settings
5. Enable ICE for better NAT traversal

### 11. Special Features

#### 11.1 Conference Codes
- **Secure Briefing Room**: 80085000 (PIN: 4721)
- **Emergency Conference**: 80086000 (No PIN)

#### 11.2 Special Numbers
- **Voicemail Access**: 82000000 (PIN: 5309)
- **Self-Destructing Messages**: 86000000
- **All Agents Alert**: 88000000

#### 11.3 Messaging
- Direct Messages To Any Agent Using Their 8-Digit Number
- Send Broadcast Message To All Agents By Messaging 88000000

### 12. Troubleshooting

#### 12.1 Common Issues
- **Registration Failures**: Check Firewall, PJSIP Settings, And Network Connectivity  
- **One-Way Audio**: Check NAT Settings And Media Port Ranges  
- **Message Delivery Failure**: Verify Message_Context Is Properly Configured  
- **Conference Problems**: Check Confbridge.conf Settings

#### 12.2 Diagnostic Commands
```
# Check For SIP Errors
pjsip show registrations
pjsip set logger on

# View Call History
cdr show status

# Check System Resources
core show uptime

# Test Connectivity
ping -c 4 your-server-ip
traceroute your-server-ip
```

### 13. System Security

#### 13.1 Firewall Recommendations
```bash
# Set up Fail2ban for SIP protection
sudo apt install fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Add to jail.local:
```
[asterisk]
enabled = true
port = 5060,5061
protocol = udp,tcp
filter = asterisk
logpath = /var/log/asterisk/full
maxretry = 5
bantime = 3600
```

### 14. System Design Map

```
+-----------------+           +------------------+
| Game Players    |           | Game Master      |
|                 |           |                  |
| +-------------+ |           | +-------------+  |
| | Linphone    | |           | | Linphone    |  |
| | Client      | |           | | Client      |  |
| | Agent IDs:  | |           | | Admin:      |  |
| | 10765432    | |           | | 99887766    |  |
| | 24567890    | |           | +-------------+  |
| +-------------+ |           |                  |
+-----------------+           | SSH Access to    |
        |                     | Server for       |
        |                     | Management       |
        |                     +--------+---------+
        |                              |
        |                              |
        v                              v
+-----------------------------------------------+
|              SIP/RTP Protocol                 |
+-----------------------------------------------+
                       |
                       v
+-----------------------------------------------+
|             ASTERISK SERVER                   |
|-----------------------------------------------|
| +---------------+      +------------------+   |
| | PJSIP Module  |      | Dialplan         |   |
| | (SIP Stack)   |      | (Call Logic)     |   |
| +---------------+      +------------------+   |
|                                               |
| +---------------+      +------------------+   |
| | Message       |      | Conference       |   |
| | Module        |      | Bridge           |   |
| +---------------+      +------------------+   |
|                                               |
| +---------------+      +------------------+   |
| | Voicemail     |      | Recording        |   |
| | System        |      | System           |   |
| +---------------+      +------------------+   |
|                                               |
| +----------------------------------------+    |
| | Custom Audio Files                     |    |
| | /var/lib/asterisk/sounds/custom/      |    |
| +----------------------------------------+    |
+-----------------------------------------------+
```

### 15. Call Flows

#### 15.1 Basic Call Flow
```
Agent A                Asterisk                  Agent B
   |                      |                         |
   |---- INVITE --------->|                         |
   |                      |---- INVITE ------------>|
   |                      |<------- 180 Ringing ----|
   |<--- 180 Ringing -----|                         |
   |                      |<------- 200 OK ---------|
   |<------ 200 OK -------|                         |
   |------ ACK ---------->|                         |
   |                      |------- ACK ------------>|
   |                                                |
   |<============ Two-way RTP Audio =============>  |
   |                                                |
   |------ BYE ---------->|                         |
   |                      |------- BYE ------------>|
   |                      |<------- 200 OK ---------|
   |<------ 200 OK -------|                         |
```

#### 15.2 Messaging Flow
```
Agent A                Asterisk                  Agent B
   |                      |                         |
   |---- MESSAGE -------->|                         |
   |<---- 202 Accepted ---|                         |
   |                      |---- MESSAGE ----------->|
   |                      |<---- 200 OK ------------|
   |                      |                         |
   |---- MESSAGE -------->|                         |
   |<---- 202 Accepted ---|                         |
   |                      |---- MESSAGE ----------->|
   |                      |<---- 200 OK ------------|
```

### 16. Customization Guide

#### 16.1 Adding Custom Sounds
Create Your Own Sound Files To Enhance The Secret Agent Experience:

1. **Creating sound files with text-to-speech**:
   ```
   # Install espeak Or Other TTS
   sudo apt install espeak
   
   # Create New Sound
   espeak -w /tmp/mission-briefing.wav "Attention Agent. This Is Your Mission Briefing. The Target Is In Position."
   
   # Convert To Proper Format
   sox /tmp/mission-briefing.wav -r 8000 -c 1 /var/lib/asterisk/sounds/custom/mission-briefing.wav
   
   # Set Permissions
   sudo chown asterisk:asterisk /var/lib/asterisk/sounds/custom/mission-briefing.wav
   ```

---

**CONFIDENTIAL: FOR AUTHORIZED PERSONNEL ONLY**
