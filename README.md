# 🚀 OneDrive to OneDrive Transfer via rclone + Azure Container Instances

**TL;DR**  
I migrated all 295 GB / 400k files from a Microsoft 365 Business Standard OneDrive (**source**) into my personal Microsoft 365 OneDrive (**destination**) for less than ten bucks using an Azure Container Instance (ACI) running an rclone image from my Docker Hub repo. The entire process ran in the cloud — no Azure CLI, no local VM — just a template pasted into the Azure Portal. Large Language Models (LLMs) helped with development, KQL troubleshooting, and regex filtering.

## 🧰 Repo Contents

This repo contains:

- `template.json` – Azure Resource Manager template to create the ACI
- `parameters.json` – Parameter file with all required values (tokens removed)
- Sample `commandOverrideArray` for both `rclone copy` and `rclone check`
- README and blog link

## 🏗️ Prerequisites

- Azure subscription (Pay-as-you-go works fine)
- Two OneDrive accounts (Business Standard + Personal M365)
- Base64-encoded `rclone.conf` file with configured remotes
- VPN disabled during `rclone config` OAuth login step

## 🚀 How It Works

1. Use rclone locally to run `rclone config` and create remotes for source and destination.
2. Base64-encode your rclone config and paste it into `parameters.json` as `environmentVariable0`.
3. In the Azure Portal, go to **Deploy a custom template**, paste in the `template.json`, provide the parameters, and launch.
4. Monitor the job via Azure Container Instance Bash shell and Azure Log Analytics (KQL).
5. After the copy, spin up a second container to run `rclone check` to validate file integrity.

### Sample Command

```json
"commandOverrideArray": {
  "value": [
    "sh",
    "-c",
    "mkdir -p /root/.config/rclone && echo \"$RCLONE_CONF_B64\" | base64 -d > /root/.config/rclone/rclone.conf && unset RCLONE_CONF_B64 && rclone copy source: destination: --progress --transfers 6 --tpslimit 8 --log-level INFO && tail -f /dev/null"
  ]
}
