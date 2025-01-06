# Setting Up an IDS/IPS Solution

To enhance security, an IDS (Intrusion Detection System) and IPS (Intrusion Prevention System) are deployed. These tools detect and prevent attacks in real-time. In pfSense, we use **Snort** for IDS functionality.

## Installing Snort on pfSense

1. Navigate to **System → Package Manager → Available Packages**.
2. Search for **Snort**.
3. Click **Install** to download and install Snort.

## Snort Configuration in pfSense

1. Go to **Services → SNORT** to access the Snort configuration panel.
2. In this panel, you can manage all Snort settings and rules.

## Configuring Interfaces

For detecting or preventing threats through the **WAN**, we need to configure the interfaces.

1. Go to **Snort Interfaces** and click **Add**.
2. Tick **Enable** and select **WAN** as the interface.
3. Click **Save** to apply the changes.

## Detecting Port Scanning

As part of the reconnaissance phase, attackers often perform **port scanning**. We’ll start by detecting port scanning using **Nmap**.

1. Go to **WAN → Categories** and select the necessary ruleset.
2. For detecting port scanning, enable the **emerging-scan.rules** ruleset and click **Save**.
3. Then, navigate to **WAN Rules** and select **emerging-scan.rules** via Category Selection.
4. Enable all rules in that set and click **Apply** to activate them.

Additionally, Snort provides predefined rules that you can activate. To enable **port scan detection**:

1. Click **Edit** on the WAN interface in Snort.
2. Go to **WAN Preprocs → Portscan Detection**.
3. Tick **Use Portscan Detection** and save.

This configuration will allow Snort to detect **Nmap** or any other port scanning attempts targeting your server.

## Detecting SSH Brute Force Attacks

To protect your SSH server from brute-force attacks, you can create a custom rule to detect such attempts.

1. Go to **WAN Rules → Category Selection** and select **custom.rules**.
2. Add the following custom rule to detect SSH brute-force attempts:

```plaintext
alert tcp any any -> $HOME_NET 22 (msg:"SSH brute force detected"; flags: S; threshold: type both, track by_src, count 10, seconds 60; sid:10000006; rev:1;)
```

3. Save the rule, and Snort will now detect SSH brute-force attacks targeting your network.

## Final Test

After applying these configurations, Snort will monitor for port scanning activities and SSH brute-force attacks. You can test these by running a **Nmap port scan** or attempting an SSH brute force attack on the configured network to verify detection.

## Port-Scanning Test

![Screenshot From 2025-01-06 18-38-37](https://github.com/user-attachments/assets/fab2f375-d8f4-46d5-8b34-dfa93d4b1880)

by running nmap on server from our network using `nmap 192.168.11.200`

Then we got alerts in snort dashboard 

![Screenshot From 2025-01-06 14-22-22](https://github.com/user-attachments/assets/a74e1665-5c7d-434e-887c-adefdc8d22ba)

## SSh-BruteForce Test
![image](https://github.com/user-attachments/assets/54190b18-299e-45ed-bbc5-0943eda45969)

by running hydra we gonna bruterforce password for root user of ssh server in our network
```bash
hydra -l root -P /usr/share/seclists/Passwords/500-worst-passwords.txt ssh://192.168.11.200
```
Then we got alerts in snort dashboard 

![Screenshot From 2025-01-06 18-41-32](https://github.com/user-attachments/assets/b47bf092-96a5-4f57-a0b1-5d47967fbce9)
