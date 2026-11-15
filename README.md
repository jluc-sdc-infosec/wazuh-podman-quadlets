
# wazuh-podman-quadlets

- Host OS: openSUSE Leap 16.1, managed remotely via Tailscale SSH and Cockpit.
- Host Specs: Headless HP Pavilion 15 Notebook PC (Intel Core i7-5500U @ 2.40GHz, 12GB RAM, 240GB storage).

## Steps:

First time installing this kind of program no less on Open Suse which has very little documentation for it compared to Ubuntu.
and from what i found, and how i understood it, i need to increase its 

vm.max_map_count from 65530 default to 262144 to prevent OpenSearch (wazuh.indexer) from crashing
```ini
echo "vm.max_map_count=262144" | sudo tee /etc/sysctl.d/99-wazuh.conf
sudo sysctl -p /etc/sysctl.d/99-wazuh.conf
```
<img width="553" height="80" alt="screenshot_20261114_233227-annotated" src="https://github.com/user-attachments/assets/0227a65b-9e40-446d-9919-9905407a7d28" />

- Cloning the Official Wazuh Container Repository 
```ini
sudo mkdir -p /opt/wazuh && cd /opt/wazuh
cd /opt/wazuh
sudo git clone https://github.com/wazuh/wazuh-docker.git -b v4.14.10
cd wazuh-docker/single-node
```
- Generating Internal SSL/TLS Certificates
