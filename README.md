# 1. installing the firewall with opnsense

## download:
- download the iso file from [pfsense.iso](https://www.pfsense.org/download/) and decompress it.

## create a virtual machine:
1. open virtualbox and create a new vm with the following requirements:
   - **disk**: at least 15 gb.
   - **ram**: at least 2 gb.
2. add the opnsense iso to the vm.

## installation:
1. start the vm with the iso mounted.
2. proceed with the installation like any other system.

## network configuration:
in virtualbox, configure the network settings:
- **adapter 1**: NAT (represents the public network, takes an IP in the range of my local machine's IP).
- **adapter 2**:  LAN (represents our internal network). 

![Screenshot From 2025-01-07 12-20-40](https://github.com/user-attachments/assets/438834f1-58ad-4858-906a-f48cd5105fda)

## PfSense Panel

after accessing the opnsense panel via the ip assigned for nat (in this case, `192.168.11.200`), log in using the following credentials:  
- **username**: `root`  
- **password**: `opnsense`  

upon logging in, the first page displays a panel with graphs representing usage statistics. to configure the firewall and set up rules, there are many functions available. the initial step involves managing services and filtering some from the network.
### enabling ssh in opnsense

before proceeding with testing, ssh must be activated. by default, opnsense allows you to select which networks can access the secure shell (ssh) service. 

in this case, the following interfaces were selected to meet the exercise requirements:
- **LAN**
- **WAN**

once ssh was enabled, it was tested successfully and confirmed to be working as expected.

![image](https://github.com/user-attachments/assets/bcacc3d6-c960-4c08-af11-3c5578be6fb7)

# 3. configuring the firewall in opnsense

### blocking all incoming traffic  
we will start by blocking all incoming (input) traffic related to all services except for commonly used ones:  
- **http (80)**  
- **https (443)**  
- **ssh (22)**  

in this case, we will use a whitelist approach since the number of allowed protocols is fewer than those to be blocked. we will use the **pass** action to whitelist ports `80`, `443`, and `22`.

### Steps to Configure Rules in OPNsense  

Navigate to:  
**OPNsense → Firewall → Rules → WAN**

#### Block All Traffic  
1. Click **Add (+)** to create a new rule.  
2. Configure the following:  
   - **Action**: `Block`.  
   - **Protocol**: `Any`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Log**: Enable.

---

#### Allow HTTP, HTTPS, and SSH  
Navigate to **WAN** and create an **Allow** rule for each service:  

1. **HTTP and HTTPS**:  
   - **Action**: `Pass`.  
   - **Protocol**: `TCP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `HTTP`  
     - **To**: `HTTPS`.  
   - **Log**: Enable.  

2. **SSH**:  
   - **Action**: `Pass`.  
   - **Protocol**: `TCP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `SSH`  
     - **To**: `SSH`.  
   - **Log**: Enable.  

---

#### Allow Only Outgoing Connections on Specific Ports (80, 443, 53)  

Navigate to:  
**OPNsense → Firewall → Rules → WAN**

1. **Block All Outgoing Traffic**:  
   - Click **Add (+)** to create a new rule.  
   - Configure the following:  
     - **Action**: `Block`.  
     - **Protocol**: `Any`.  
     - **Source**: `Any`.  
     - **Destination**: `Any`.  
     - **Log**: Enable.  

2. **Allow HTTP/HTTPS (Port 80, 443)**:  
   - Click **Add (+)** to create a new rule.  
   - Configure the following:  
     - **Action**: `Pass`.  
     - **Protocol**: `TCP`.  
     - **Source**: `Any`. 
     - **Source Port Range**:  
       - **From**: `80`  
       - **To**: `443`.  
     - **Destination**: `Any`.  
     - **Log**: Enable.  

4. **Allow DNS (Port 53)**:  
   - Configure similarly to **HTTP**, but set:  
     - **Protocol**: `TCP/UDP`.  
     - **Source Port Range**:  
       - **From**: `53`  
       - **To**: `53`.  

---

#### Block Telnet, FTP, and SMB Protocols  

Navigate to **WAN** and create a **Block** rule for each protocol:  

1. **Telnet (Port 23)**:  
   - **Action**: `Block`.  
   - **Protocol**: `TCP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `23`  
     - **To**: `23`.  
   - **Log**: Enable.  

2. **FTP (Ports 20, 21)**:  
   - **Action**: `Block`.  
   - **Protocol**: `TCP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `20`  
     - **To**: `21`.  
   - **Log**: Enable.  

3. **SMB (Ports 135-139, 445)**:  
   - **Action**: `Block`.  
   - **Protocol**: `TCP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `135`  
     - **To**: `139`.  
   - **Log**: Enable.  

   - Create another rule for port `445`:  
     - **Action**: `Block`.  
     - **Protocol**: `TCP`.  
     - **Source**: `Any`.  
     - **Destination**: `Any`.  
     - **Destination Port Range**:  
       - **From**: `445`  
       - **To**: `445`.  
     - **Log**: Enable.  

---

#### Block Torrent Downloads  

1. **Torrent (Port Range 6881-6889)**:  
   - **Action**: `Block`.  
   - **Protocol**: `TCP/UDP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `6881`  
     - **To**: `6889`.  
   - **Log**: Enable.  


### Enable and Configure Connection Logs  

	We enabled logs earlier by selecting Log in each rule.

![image](https://github.com/user-attachments/assets/a8a7bc0b-85ca-43e1-ae3f-ffa5d91e5e73)

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
