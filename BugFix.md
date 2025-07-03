# Fix Log Rotation for Asterisk

To Fix Log Rotation For Asterisk (Especially For `/var/log/asterisk/messages.log`, `selfdestmsgs.log`, etc.), You Should Create A Custom `logrotate` Configuration.

---

## 1. Create Asterisk Logrotate Config File

Create The Dile `/etc/logrotate.d/asterisk`:

```bash
sudo nano /etc/logrotate.d/asterisk
```

Paste The Following Content:

```nginx
/var/log/asterisk/messages.log
/var/log/asterisk/selfdestmsgs.log
/var/log/asterisk/full
{
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 asterisk asterisk
    sharedscripts
    postrotate
        /bin/systemctl reload asterisk > /dev/null 2>/dev/null || true
    endscript
}
```

---

## 2. Set Permissions and Ownership

Ensure Asterisk Owns The Log Files:

```bash
sudo chown asterisk:asterisk /var/log/asterisk/*.log
sudo chmod 640 /var/log/asterisk/*.log
```

---

## 3. Test Logrotate

Manually Test If The Rotation Works:

```bash
sudo logrotate -f /etc/logrotate.d/asterisk
```

After Running, You Should See Rotated Files Like:

```
/var/log/asterisk/messages.log.1.gz
/var/log/asterisk/selfdestmsgs.log.1.gz
```

---
