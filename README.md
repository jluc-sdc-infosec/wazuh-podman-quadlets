# wazuh-podman-quadlets

Host OS: openSUSE Leap 16.1, managed remotely via Tailscale SSH and Cockpit.
Host Specs: Headless HP Pavilion 15 Notebook PC (Intel Core i7-5500U @ 2.40GHz, 12GB RAM, 240GB storage).

## Steps:

First time installing this kind of program no less on Open Suse which has very little documentation for it compared to Ubuntu.
and from what i found, and how i understood it, i need to increase its 

vm.max_map_count from 65530 default to 262144 to prevent it from crashing
```ini
echo "vm.max_map_count=262144" | sudo tee /etc/sysctl.d/99-wazuh.conf
sudo sysctl --system
```
<img width="962" height="1025" alt="screenshot_20261114_140248-annotated" src="https://github.com/user-attachments/assets/0a373fd3-bccc-4ab5-8150-cce8561a507b" />

