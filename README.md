
# wazuh-podman-quadlets
- Core Service: Wazuh SIEM stack (Indexer, Manager, Dashboard) deployed as Rootful containers managed via Podman Quadlets (`systemd`).
- Host OS: openSUSE Leap 16.1, managed remotely via Tailscale SSH and Cockpit.
- Host Specs: Headless HP Pavilion 15 Notebook PC (Intel Core i7-5500U @ 2.40GHz, 12GB RAM, 240GB storage).
- Security Role: Single-node SIEM and XDR platform handling host/agent log ingestion, File Integrity Monitoring (FIM), vulnerability detection, and alert triage.

## Steps on installing:

**Note**: First time installing this kind of program no less on Open Suse which has very little documentation for it compared to Ubuntu.
and from what i found, and how i understood it, i need to increase its 
``vm.max_map_count`` from 65530 default to 262144 to prevent OpenSearch (wazuh.indexer) from crashing
```bash
echo "vm.max_map_count=262144" | sudo tee /etc/sysctl.d/99-wazuh.conf
sudo sysctl -p /etc/sysctl.d/99-wazuh.conf
```
**Checking vm.max_map_count**

<img width="553" height="80" alt="screenshot_20261114_233227-annotated" src="https://github.com/user-attachments/assets/0227a65b-9e40-446d-9919-9905407a7d28" />

- Cloning the Official Wazuh Container Repository 
```bash
# create the install folder, grab the wazuh repo, and head into the single-node setup
sudo mkdir -p /opt/wazuh
cd /opt/wazuh

sudo git clone https://github.com/wazuh/wazuh-docker.git -b v4.14.10
cd wazuh-docker/single-node
```
- Generating Internal SSL/TLS Certificates
Creating the certificate output directory
```bash
# create the folder where the ssl certs will go
sudo mkdir -p /opt/wazuh/wazuh-docker/single-node/config/wazuh_indexer_ssl_certs

# generate ssl certs for wazuh using certs.yml
sudo podman run --rm \
  -e CERT_TOOL_VERSION=4.14 \
  -v /opt/wazuh/wazuh-docker/single-node/config/wazuh_indexer_ssl_certs:/certificates:Z \
  -v /opt/wazuh/wazuh-docker/single-node/config/certs.yml:/config/certs.yml:Z \
  docker.io/wazuh/wazuh-certs-generator:0.0.5
```
**Note:** `:Z` is needed on the volume mounts so SELinux doesn't block the container from reading/writing those host folders.
```bash
# give read permissions to the config folder so the containers can access the certs
sudo chmod -R a+rX /opt/wazuh/wazuh-docker/single-node/config/
```
