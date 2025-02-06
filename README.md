#!/bin/bash

# Ensure firewalld is installed and running
if ! command -v firewall-cmd &>/dev/null; then
    echo "Installing firewalld..."
    apt update && apt install -y firewalld
    systemctl enable --now firewalld
fi

# Configure firewalld
ICMP_TYPE="echo-request"  # Modify as needed
SERVICE_NAME="ssh"  # Modify as needed
PORT_NUMBER="443"  # Modify as needed
PORT_TYPE="tcp"  # Modify as needed

echo "Configuring firewalld..."
firewall-cmd --add-icmp-block=$ICMP_TYPE
firewall-cmd --add-icmp-block-inversion
firewall-cmd --runtime-to-permanent
firewall-cmd --list-services
firewall-cmd --add-service=$SERVICE_NAME
firewall-cmd --add-port=$PORT_NUMBER/$PORT_TYPE
firewall-cmd --remove-port=$PORT_NUMBER/$PORT_TYPE
firewall-cmd --set-log-denied=all

# Disable and remove cron
echo "Disabling and removing cron..."
systemctl stop cron
systemctl disable cron
apt remove -y cron

# Harden SSH configuration
SSHD_CONFIG="/etc/ssh/sshd_config"
echo "Configuring SSH..."
sed -i 's/^#*PubkeyAuthentication.*/PubkeyAuthentication yes/' $SSHD_CONFIG
sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' $SSHD_CONFIG
sed -i 's/^#*AuthenticationMethods.*/AuthenticationMethods publickey,gssapi-with-mic publickey,keyboard-interactive/' $SSHD_CONFIG

# Restart SSH service
echo "Restarting SSH service..."
systemctl restart sshd

# Generate SSH key and copy it to the server
USER_SERVER="<user>@<server>"  # Modify as needed
echo "Generating SSH key..."
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
ssh-copy-id -i ~/.ssh/id_rsa.pub $USER_SERVER

# Enable syslog
echo "Enabling syslog..."
systemctl enable --now rsyslog

echo "Configuration completed."
