## Deploy command
```bash
curl -sSL https://raw.githubusercontent.com/Momen-Amin/Dongle-Manager-V1/main/DongleManagerV1.1 | bash
```

# Mowjat Dongle Manager - Deployment Guide

## Overview
This is an automated deployment script for the **Mowjat Dongle Manager**, a production-ready web interface for monitoring and controlling Asterisk Chan_Dongle USB devices.

## Features
✅ **Real-time Monitoring** - Live status updates every 3 seconds
✅ **Dongle Control** - Start, stop, restart individual dongles
✅ **Call Tracking** - Monitor active calls with duration and direction
✅ **Signal Strength** - Visual signal strength indicators
✅ **Telegram Notifications** - Get alerts when dongles go online/offline
✅ **Responsive Design** - Works on all screen sizes (desktop, tablet, mobile)
✅ **Asterisk Management** - Reload and restart Asterisk from the interface

## System Requirements
- **OS:** CentOS 7 x64 (or compatible)
- **Web Server:** Apache (already installed and configured)
- **PHP:** Version 7.4 or higher
- **Asterisk:** Version 19 with Chan_Dongle module
- **FreePBX:** Version 16 (optional, for GUI)
- **Root Access:** Required for installation

## Quick Installation

### Step 1: Download the Deployment Script
```bash
# Upload the deploy_dongle_manager.sh script to your server
# Or download directly from your repository
```

### Step 2: Make it Executable
```bash
chmod +x deploy_dongle_manager.sh
```

### Step 3: Run the Deployment Script
```bash
./deploy_dongle_manager.sh
```

That's it! The script will automatically:
- Create all necessary directories
- Download assets (logo and favicon)
- Generate all PHP files (index, API, Telegram config)
- Set proper permissions
- Configure cron job for Telegram monitoring
- Restart Apache

## Installation Directory Structure

After deployment, your directory structure will be:

```
/var/www/html/dongle-manager/
├── index.php                    # Main web interface
├── images/
│   ├── mowjat-logo.png         # Company logo
│   ├── mowjat-favicon.ico      # Browser favicon
│   └── index.php               # Directory protection
├── api/
│   ├── api.php                 # Backend API
│   └── index.php               # Directory protection
└── tg/
    ├── telegram_config.php     # Telegram configuration
    ├── telegram_monitor.php    # Monitoring script
    ├── dongle_states.json      # State tracking (auto-created)
    └── index.php               # Directory protection
```

📱 Telegram Setup (3 Steps):
Step 1: Create Bot

Open Telegram → Search @BotFather
Send: /start
Send: /newbot
Send: XX_Alerts_Bot
Get your bot token: 123456789:ABCdefGHIjklMNOpqrsTUVwxyz

Step 2: (Optional)
Send: /setprivacy
Click: @XX_Alerts_Bot
Click: Disable

Step 3: Get Chat ID
Create a Group > XX_Alerts_Bot_Group > Add XX_Alerts_Bot + IDBot
Search your Bot: XX_Alerts_Bot
Send: /start
Search your Group: XX_Alerts_Bot_Group
Send: /getgroupid@myidbot
Your group ID is: -123456789

# Test if you can reach Telegram API
curl https://api.telegram.org/bot123456789:ABCdefGHIjklMNOpqrsTUVwxyz/getMe


## Post-Installation Configuration

### 1. Configure Telegram Notifications (Optional)

Edit the Telegram configuration file:
```bash
nano /var/www/html/dongle-manager/tg/telegram_config.php
```

You'll need:
- **Bot Token**: Get from @BotFather on Telegram
- **Chat ID**: Get from @userinfobot or @getmyid_bot

```php
return [
    'TELEGRAM_BOT_TOKEN' => 'YOUR_BOT_TOKEN_HERE',
    'TELEGRAM_CHAT_ID' => 'YOUR_CHAT_ID_HERE',
    'NOTIFICATIONS_ENABLED' => true,
    'NOTIFY_ON_ONLINE' => true,
    'NOTIFY_ON_OFFLINE' => true,
];
```

### 2. Verify Apache is Running
```bash
systemctl status httpd
# or
systemctl status apache2
```

### 3. Verify Asterisk is Running
```bash
asterisk -rx "core show version"
```

### 4. Test Chan_Dongle Module
```bash
asterisk -rx "dongle show devices"
```

## Accessing the Interface

Open your web browser and navigate to:
```
http://YOUR_SERVER_IP/dongle-manager/
```

Replace `YOUR_SERVER_IP` with your actual server IP address.

## Interface Features

### Dashboard Statistics
- **Total Dongles**: Shows total number of connected dongles
- **Online**: Number of dongles currently online
- **Active Calls**: Number of dongles currently in a call
- **Last Updated**: Timestamp of last data refresh

### Dongle Table
Each dongle displays:
- Dongle ID
- Online/Offline status
- IMEI number
- IMSI number
- Network provider
- Device model
- Call status (Idle, Ringing, In-Call, Out-Call)
- Signal strength (visual bar + percentage)
- Action buttons (Enable, Disable, Restart)

### Filtering & Search
- **All**: Show all dongles
- **Online**: Show only online dongles
- **Offline**: Show only offline dongles
- **Active Calls**: Show only dongles currently in a call
- **Search**: Search by ID, IMEI, IMSI, or Provider

### System Controls
- **Reload Asterisk**: Reload Chan_Dongle module
- **Restart Asterisk**: Restart entire Asterisk service

## Telegram Notifications

The system can send Telegram notifications when:
- A dongle goes offline (was online, now offline)
- A dongle comes online (was offline, now online)

Notifications include:
- Dongle ID
- Status (ONLINE/OFFLINE)
- Current state
- Timestamp

### Cron Job
The monitoring script runs every minute via cron:
```
* * * * * root /usr/bin/php /var/www/html/dongle-manager/tg/telegram_monitor.php >> /var/log/telegram-monitor.log 2>&1
```

### Viewing Logs
```bash
tail -f /var/log/telegram-monitor.log
```

## Troubleshooting

### Issue: Page shows "No dongles found"
**Solution:**
1. Verify Asterisk is running: `systemctl status asterisk`
2. Check Chan_Dongle is loaded: `asterisk -rx "module show like dongle"`
3. Verify dongles are connected: `asterisk -rx "dongle show devices"`

### Issue: API not responding
**Solution:**
1. Check Apache error logs: `tail -f /var/log/httpd/error_log`
2. Verify PHP is installed: `php -v`
3. Check file permissions: `ls -la /var/www/html/dongle-manager/`

### Issue: Telegram notifications not working
**Solution:**
1. Verify bot token is correct
2. Verify chat ID is correct
3. Check monitoring logs: `tail -f /var/log/telegram-monitor.log`
4. Test manually: `/usr/bin/php /var/www/html/dongle-manager/tg/telegram_monitor.php`

### Issue: Permission denied errors
**Solution:**
```bash
cd /var/www/html
chown -R apache:apache dongle-manager/
chmod -R 755 dongle-manager/
```

### Issue: Page not loading (403/404)
**Solution:**
1. Verify Apache is running: `systemctl status httpd`
2. Check Apache config: `httpd -t`
3. Restart Apache: `systemctl restart httpd`

## Security Recommendations

1. **Firewall**: Restrict access to the web interface
   ```bash
   firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="YOUR_IP" port protocol="tcp" port="80" accept'
   firewall-cmd --reload
   ```

2. **HTTPS**: Consider setting up SSL/TLS certificate

3. **Authentication**: Add Apache basic authentication
   ```bash
   htpasswd -c /etc/httpd/.htpasswd admin
   ```

4. **File Permissions**: Already set by the script, but verify:
   ```bash
   chmod 644 /var/www/html/dongle-manager/tg/telegram_config.php
   ```

## Updating the System

To update the system:
1. Backup your current installation
   ```bash
   cp -r /var/www/html/dongle-manager /root/dongle-manager.backup
   ```

2. Run the deployment script again
   ```bash
   ./deploy_dongle_manager.sh
   ```

3. Reconfigure Telegram if needed

## Uninstallation

To completely remove the system:
```bash
# Remove files
rm -rf /var/www/html/dongle-manager

# Remove cron job
sed -i '/telegram_monitor.php/d' /etc/crontab

# Remove log file
rm -f /var/log/telegram-monitor.log
rm -f /var/log/dongle-manager-deployment.log
```

## API Endpoints

The system provides the following API endpoints:

### GET /api/api.php?action=list
Returns list of all dongles with details

### POST /api/api.php
Available actions:
- `start_dongle` - Enable a dongle
- `stop_dongle` - Disable a dongle
- `restart_dongle` - Restart a dongle
- `reload_asterisk` - Reload Asterisk
- `restart_asterisk` - Restart Asterisk
- `asterisk_status` - Get Asterisk status

## Technical Details

### Auto-Refresh
- Dongle data: Every 3 seconds
- Asterisk status: Every 10 seconds

### Call Duration Tracking
- Client-side tracking for accurate durations
- Synchronized with server-side data

### Responsive Breakpoints
- Desktop: > 1024px
- Tablet: 768px - 1024px
- Mobile: < 768px

## Support & Logs

### Deployment Log
```bash
tail -f /var/log/dongle-manager-deployment.log
```

### Apache Error Log
```bash
tail -f /var/log/httpd/error_log
```

### Telegram Monitor Log
```bash
tail -f /var/log/telegram-monitor.log
```

## Credits

**Mowjat Dongle Manager**
- Version: 1.0
- Developed for Asterisk Chan_Dongle Management
- Compatible with CentOS 7, Asterisk 19, FreePBX 16

---

## Quick Reference Commands

```bash
# Check service status
systemctl status asterisk
systemctl status httpd

# View dongles in Asterisk
asterisk -rx "dongle show devices"

# Restart services
systemctl restart asterisk
systemctl restart httpd

# View logs
tail -f /var/log/telegram-monitor.log
tail -f /var/log/httpd/error_log

# Test Telegram monitor
/usr/bin/php /var/www/html/dongle-manager/tg/telegram_monitor.php

# Check cron jobs
cat /etc/crontab | grep telegram

# Verify file permissions
ls -la /var/www/html/dongle-manager/
```

---

**For issues or questions, refer to the troubleshooting section above.**
