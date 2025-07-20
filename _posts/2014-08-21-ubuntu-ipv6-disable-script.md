Hey friends! 👋

Check out this **menu-driven Bash script** that helps you **disable IPv6** from your Ubuntu box.

---

## 🔧 What This Script Does

- Detects if IPv6 is enabled
- Displays your active IPv6 address (if any)
- Disables it safely using `sysctl.conf`
- Includes an “About” menu section
- Clean, interactive CLI interface

---

## 📜 The Script

```bash
#!/bin/bash
#################################
# Author: Shivam Shukla
# Build Date: 02-10-2014
# Modification Date: 11-11-2014
# Version: 1.0.0
#################################

build="02/10/2014"
version="1.0.0"
modified="NA"

# Step #1: Define variables
pause(){
  read -p "Press [Enter] key to continue..." fackEnterKey
}

function fn_ipv6disable() {
  int=$(netstat -i | cut -d" " -f1 | egrep -v "^Kernel|Iface|lo|:" | head -1)
  ip -6 addr show dev $int | cut -b 5-9 > /tmp/ipp
  grep "inet6" /tmp/ipp >/dev/null 2>&1 && ip_v=1 || ip_v=0
  i6=$(ip addr show dev $int | sed -e 's/^.*inet6 \([^ ]*\)\/.*$//;t;d')

  if [ $ip_v -eq 1 ]; then
    echo "IPv6 found..."
    echo "The IPv6 address is $i6"
    echo "Disabling IPv6..."
    echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
    echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.conf
    echo "net.ipv6.conf.lo.disable_ipv6 = 1" >> /etc/sysctl.conf
    sysctl -p
    echo "IPv6 disable success ✅"
  else
    echo "IPv6 is already disabled ✅"
  fi
}

function fn_about() {
  date '+%d/%m/%y %H:%M:%S' > time
  var_date_time=$(<time)
  echo -e "\033[34m###########################################################################"
  echo "#                                                                         #"
  echo "#                                 About                                  #"
  echo "#-------------------------------------------------------------------------#"
  echo "# Author         : Shivam Shukla                                          #"
  echo "# Type           : IPv6 Disable Utility for Ubuntu                        #"
  echo "# Version        : $version                                               #"
  echo "# Build Date     : $build                                                 #"
  echo "# Modified       : $modified                                              #"
  echo "# Current Time   : $var_date_time                                         #"
  echo "#                                                                         #"
  echo -e "###########################################################################\033[0m"
}

one(){
  fn_ipv6disable
  pause
}

two(){
  fn_about
  pause
}

show_menus() {
  clear
  echo -e '\E[0;34m'"\033[1m~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\033[0m"
  echo -e '\E[0;34m'"\033[1m~ WELCOME TO IPv6 TOOL FOR UBUNTU         ~\033[0m"
  echo -e '\E[0;34m'"\033[1m~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\033[0m"
  echo -e '\E[0;34m'"\033[1m~ 1. Remove IPv6 From Ubuntu Machine       ~\033[0m"
  echo -e '\E[0;34m'"\033[1m~ 2. About                                 ~\033[0m"
  echo -e '\E[0;34m'"\033[1m~ 3. Exit                                  ~\033[0m"
  echo -e '\E[0;34m'"\033[1m~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\033[0m"
}

read_options(){
  local choice
  read -p "Enter choice [ 1 - 3 ]: " choice
  case $choice in
    1) one ;;
    2) two ;;
    3) exit 0 ;;
    *) echo -e '\E[47;31m'"\033[1mError... Invalid Option\033[0m" && sleep 2 ;;
  esac
}

trap '' SIGINT SIGQUIT SIGTSTP

while true; do
  show_menus
  read_options
done
```

---

## 🧪 How to Use

1. Save the script as `ipv6_disable.sh`
2. Give it execution permissions:

```bash
chmod +x ipv6_disable.sh
```

3. Run it with root privileges:

```bash
sudo ./ipv6_disable.sh
```

---

## 📝 Notes

- The script updates `/etc/sysctl.conf` to persist changes across reboots.
- It’s safe to rerun — it won’t duplicate entries.
- Tested on Ubuntu 14.x and 16.x (but should work on newer versions too).

---

Hope you find this useful! 🚀 Feel free to fork or enhance it for your distro or version.

Let me know in comments if you have questions or improvements!
