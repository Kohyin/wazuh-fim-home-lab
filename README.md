# Wazuh SIEM Home Lab: File Integrity Monitoring on Windows

A hands-on lab where I deployed a Wazuh manager, connected a Windows 11 agent, and used File Integrity Monitoring (FIM) to detect changes in `C:\Windows\System32`. It also covers VirusTotal integration and vulnerability detection concepts.

## Lab Environment

| Component | Details |
|---|---|
| SIEM / Manager | Wazuh 4.12.0 (OVA appliance) |
| Endpoint | Windows 11 Pro, Wazuh agent v4.12.0 |
| Network | Private lab network (manager `192.168.100.22`, agent `192.168.100.7`) |
| Access | Wazuh dashboard over HTTPS from the Windows host |

## What I Did

1. **Deployed the Wazuh manager** from the OVA and logged in to the VM console.
2. **Opened the dashboard** by finding the VM's IP with `ip a` and browsing to `https://<WAZUH_IP>` from the host.
3. **Installed the Wazuh agent on Windows** from an elevated PowerShell session, pointing it at the manager IP and giving it an agent name:

   ```powershell
   Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.12.0-1.msi -OutFile $env:tmp\wazuh-agent
   msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='<MANAGER_IP>' WAZUH_AGENT_NAME='<AGENT_NAME>'
   NET START WazuhSvc
   ```

4. **Verified the agent** showed as **Active** on the Endpoints page of the dashboard.
5. **Configured FIM** by editing the agent config (Wazuh Agent Manager → View → View Config) and adding `C:\Windows\System32` to the `<syscheck>` section:

   ```xml
   <syscheck>
     <directories check_all="yes" realtime="yes">C:\Windows\System32</directories>
   </syscheck>
   ```

6. **Restarted the agent** to apply the change.
7. **Tested detection** by placing a dummy file in System32 and checking the dashboard.

## Results

Creating a file in System32 produced a File Integrity Monitoring alert on the dashboard showing:

- New file creation event
- Full file path
- Timestamp of the change
- Agent name
- Rule severity level

<!-- Add screenshots here: agent list, FIM alert, alert details -->

## Key Concepts

**File Integrity Monitoring (FIM)** watches files and directories for creation, deletion, modification, and permission changes. Attackers and malware often alter system files to persist or hide, so FIM helps detect unauthorized changes and supports compliance requirements.

**Why System32?** It holds critical Windows executables and system processes. Malware can hide among legitimate files, replace system binaries, or use the folder to persist and run automatically.

**VirusTotal integration.** With `<group>syscheck</group>` in the integration block, Wazuh sends file hashes from FIM alerts to VirusTotal. A known test file such as EICAR should then produce two alerts: the FIM alert and a VirusTotal malware detection alert.

**Vulnerability detection.**

| Term | Meaning |
|---|---|
| CVE | Public identifier for a known vulnerability (maintained by MITRE) |
| CVSS | 0.0–10.0 severity score: Low 0.1–3.9, Medium 4.0–6.9, High 7.0–8.9, Critical 9.0–10.0 |
| `interval` | How often Wazuh runs incremental vulnerability checks (e.g. `5m`) |
| `min_full_scan_interval` | Minimum time between full scans (e.g. `6h`) |

Wazuh reports the CVE ID, severity, CVSS score, description, affected package, installed and fixed versions, detection time, and reference links.

## Security Notes

- The OVA ships with default credentials (`wazuh-user` for the CLI, `admin` for the dashboard). **Change them** before using the lab for anything beyond local testing.
- Keep the lab on an isolated network and never expose the dashboard to the internet.
- Agent authentication keys are secrets. Do not commit them or screenshots that show them.

## Skills Demonstrated

SIEM deployment · Endpoint agent management · File Integrity Monitoring · Alert triage · Threat intelligence (VirusTotal) · Vulnerability management (CVE/CVSS) · Windows and Linux administration

## Related: Linux Permissions Exercise

I also practiced Linux user and permission management: creating users and groups (`useradd`, `groupadd`, `usermod -aG`), setting ownership with `chown`, applying `chmod 775` to shared project directories and `chmod 660` to a sensitive source file, and testing that cross-user access is denied.

## Author

**Richard Ayansoga** · Cybersecurity Intern, GOMYCODE · Security+ (SY0-701) · Lagos, Nigeria
