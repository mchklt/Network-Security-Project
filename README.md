# Network-Security-Project
# 1. installing the firewall with opnsense

## download:
- download the iso file from [opnsense.iso](https://mirror.marwan.ma/opnsense/releases/24.7/OPNsense-24.7-dvd-amd64.iso.bz2) and decompress it.

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

### enabling ssh in opnsense

before proceeding with testing, ssh must be activated. by default, opnsense allows you to select which networks can access the secure shell (ssh) service. 
1. Navigate to: **System → Settings → Administration**.
2. Locate the **Secure Shell** section.  
3. Check the box for **Enable Secure Shell**.

in this case, the following interfaces were selected to meet the exercise requirements:
- **opt1 (LAN)**
- **WAN**

![Screenshot From 2025-01-02 17-02-17](https://github.com/user-attachments/assets/edaaedfa-ceff-43e6-b1ee-8388ee0d9c52)


once ssh was enabled, it was tested successfully and confirmed to be working as expected.

# 3. configuring the firewall in opnsense

after accessing the opnsense panel via the ip assigned for nat (in this case, `192.168.11.200`), log in using the following credentials:  
- **username**: `root`  
- **password**: `opnsense`  

upon logging in, the first page displays a panel with graphs representing usage statistics. to configure the firewall and set up rules, there are many functions available. the initial step involves managing services and filtering some from the network.

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
   - **Direction**: `In`.  
   - **Protocol**: `Any`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Log**: Enable.

after blocking all traffics ssh it didn't work which means our rule it works good !

![Screenshot From 2025-01-02 17-15-24](https://github.com/user-attachments/assets/c9a616cb-41ef-4b33-aff5-82e48c9a5602)


---

#### Allow HTTP, HTTPS, and SSH  
Navigate to **WAN** and create an **Allow** rule for each service:  

1. **HTTP and HTTPS**:  
   - **Action**: `Pass`.  
   - **Direction**: `In`.  
   - **Protocol**: `TCP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `HTTP`  
     - **To**: `HTTPS`.  
   - **Log**: Enable.  

2. **SSH**:  
   - **Action**: `Pass`.  
   - **Direction**: `In`.  
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
**OPNsense → Firewall → Rules → LAN**

1. **Block All Outgoing Traffic**:  
   - Click **Add (+)** to create a new rule.  
   - Configure the following:  
     - **Action**: `Block`.  
     - **Direction**: `Out`.  
     - **Protocol**: `Any`.  
     - **Source**: `Any`.  
     - **Destination**: `Any`.  
     - **Log**: Enable.  

2. **Allow HTTP (Port 80)**:  
   - Click **Add (+)** to create a new rule.  
   - Configure the following:  
     - **Action**: `Pass`.  
     - **Direction**: `Out`.  
     - **Protocol**: `TCP`.  
     - **Source**: `Any`.  
     - **Destination**: `Any`.  
     - **Destination Port Range**:  
       - **From**: `80`  
       - **To**: `80`.  
     - **Log**: Enable.  

3. **Allow HTTPS (Port 443)**:  
   - Configure similarly to **HTTP**, but set the port range to:  
     - **From**: `443`  
     - **To**: `443`.  

4. **Allow DNS (Port 53)**:  
   - Configure similarly to **HTTP**, but set:  
     - **Protocol**: `UDP`.  
     - **Destination Port Range**:  
       - **From**: `53`  
       - **To**: `53`.  

---

#### Block Telnet, FTP, and SMB Protocols  

Navigate to **WAN** and create a **Block** rule for each protocol:  

1. **Telnet (Port 23)**:  
   - **Action**: `Block`.  
   - **Direction**: `In`.  
   - **Protocol**: `TCP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `23`  
     - **To**: `23`.  
   - **Log**: Enable.  

2. **FTP (Ports 20, 21)**:  
   - **Action**: `Block`.  
   - **Direction**: `In`.  
   - **Protocol**: `TCP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `20`  
     - **To**: `21`.  
   - **Log**: Enable.  

3. **SMB (Ports 137-139, 445)**:  
   - **Action**: `Block`.  
   - **Direction**: `In`.  
   - **Protocol**: `TCP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `137`  
     - **To**: `139`.  
   - **Log**: Enable.  

   - Create another rule for port `445`:  
     - **Action**: `Block`.  
     - **Direction**: `In`.  
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
   - **Direction**: `In`.  
   - **Protocol**: `TCP/UDP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `6881`  
     - **To**: `6889`.  
   - **Log**: Enable.
  
### Enable and Configure Connection Logs  
  We enabled logs earlier by selecting Log in each rule.
