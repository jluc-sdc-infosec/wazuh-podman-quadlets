
|No.| What Broke / Error  | Why It Happened  | How I Fixed It|
|:---------- | :------------------ | :--------------- | :------------ |
| 1. | `Error: statfs /opt/wazuh/wazuh-docker/single-node/config/wazuh_indexer_ssl_certs: no such file or directory` | `podman run` expects the host folder to already exist before mounting it into the container. | Made the folder first using `sudo mkdir -p /opt/wazuh/wazuh-docker/single-node/config/wazuh_indexer_ssl_certs`. |
| 2. | `Checking https://wazuh.com ... certificates were not created` | Leaving `CERT_TOOL_VERSION` blank broke the download URL, and putting `4.14.10` failed because Wazuh only lists major/minor versions (`4.14`). | Added `-e CERT_TOOL_VERSION=4.14` to the command using `wazuh-certs-generator:0.0.5`.|
| 3. | SELinux permission errors on shared SSL certificates (`:Z` vs `:z`)|Uppercase `:Z` locks a mounted file to *one* container only, so the indexer, manager, and dashboard locked each other out of the shared root CA certs.|Used lowercase `:z` on the shared SSL cert mounts so all three containers can read them, and kept uppercase `:Z` only on single-container config files.|
| 4. |`Wazuh dashboard server is not ready yet` & `unable to verify the first certificate` (`CN=demo.indexer`)|`wazuh.indexer.yml` was locked to `root` only, so the non-root user inside the indexer container couldn't read it and fell back to its built-in demo cert.| Opened up read permissions on the config folder (`sudo chmod -R a+rX /opt/wazuh/wazuh-docker/single-node/config/`) and restarted `wazuh-indexer.service`. |


## 
**2.** 
```bash
sudo podman run --rm \
-e CERT_TOOL_VERSION=4.14 \
-v /opt/wazuh/wazuh-docker/single-node/config/wazuh_indexer_ssl_certs:/certificates:Z \
-v /opt/wazuh/wazuh-docker/single-node/config/certs.yml:/config/certs.yml:Z \
docker.io/wazuh/wazuh-certs-generator:0.0.5
```
**3.** 
<img width="1452" height="500" alt="screenshot_20261115_192943-annotated" src="https://github.com/user-attachments/assets/cf63d38b-fcb8-4fcb-a096-6edcb8693c27" />

<img width="1444" height="443" alt="screenshot_20261115_192730-annotated" src="https://github.com/user-attachments/assets/5b2a7c88-e1d5-49f3-b50f-26253101b499" />
<img width="1162" height="723" alt="screenshot_20261115_193224-annotated" src="https://github.com/user-attachments/assets/99535690-ea0b-4726-8fbb-0806df2c68ef" />

**4.**

  -  Checked Container Logs
The dashboard refused to load with `Wazuh dashboard server is not ready yet`, so I checked both container logs to see what was failing:
```bash
sudo podman logs --tail 30 wazuh.indexer
sudo podman logs --tail 20 wazuh.dashboard
```

  - Isolated the Root Cause

The raw `wazuh.indexer` and `wazuh.dashboard` logs were noisy, so I pulled the error lines and used an AI tool to help parse them. Scolded the AI for hallucinating and pointing me towards `/config/certs/` paths and `securityadmin.sh` commands. 
<img width="611" height="314" alt="2026-11-15-014638_hyprshot" src="https://github.com/user-attachments/assets/b9dfb44d-94ef-4772-b734-ebc4aef9a2f9" />

The guy is a tool—helpful "sometimes", I guess, most of the time—so instead of taking its word for it, I had to manually check the actual `docker-compose.yml` and `wazuh-indexer.container` mount paths myself:


```bash
grep -E "volumes:|certs/|opensearch|wazuh.yml" -A 15 /opt/wazuh/wazuh-docker/single-node/docker-compose.yml
```

<img width="1162" height="757" alt="screenshot_20261115_205420-annotated" src="https://github.com/user-attachments/assets/58cc88f6-3fb6-4c9d-adac-6659b15c25c6" />
<img width="1061" height="779" alt="screenshot_20261115_205540-annotated" src="https://github.com/user-attachments/assets/f7684cf3-4659-4fdb-bc13-aa9cafb1bed8" />




Checking the files directly showed the volume mounts were fine—the real problem was that `wazuh.indexer.yml` was owned by `root` with restricted permissions, so the non-root user inside the indexer container couldn't read it and fell back to its built-in demo certificate (`CN=demo.indexer`).

  - Applied the Real Fix
I opened up read/execute permissions on the config directory and restarted the indexer service:
```bash
sudo chmod -R a+rX /opt/wazuh/wazuh-docker/single-node/config/
sudo systemctl restart wazuh-indexer.service
```

