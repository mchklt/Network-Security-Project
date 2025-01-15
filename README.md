# 1. Installing the Firewall with pfSense

## Download:
- Download the ISO file from [pfSense Download](https://www.pfsense.org/download/) and decompress it.

## Create a Virtual Machine:
1. Open VirtualBox and create a new VM with the following requirements:
   - **Disk**: At least 15 GB.
   - **RAM**: At least 2 GB.
2. Add the pfSense ISO to the VM.

## Installation:
1. Start the VM with the ISO mounted.
2. Proceed with the installation like any other system.

## Network Configuration:
In VirtualBox, configure the network settings:
- **Adapter 1**: **WAN**: Bridged Adapter (represents the public network, takes an IP in the range of my local machine's IP).
- **Adapter 2**: **LAN**: Host-Only Adapter ("vmbox network" represents our internal network).

### Enabling SSH in pfSense

1. Navigate to **System → Advanced → Admin Access**.
2. Locate the **Secure Shell** option and enable the **Secure Shell Server**.

The default credentials are:  
- **Username:** admin  
- **Password:** pfsense  

Once SSH is enabled, test the connection to ensure it is functioning correctly.

![image](https://github.com/user-attachments/assets/62eea733-d925-436e-aa6c-33585d03de8f)

# 3. Configuring the Firewall in pfSense

After accessing the pfSense panel via the IP assigned for LAN (in this case, `192.168.56.103`), log in using the following credentials:  
- **Username**: `admin`  
- **Password**: `pfsense`  

Upon logging in, the first page displays a panel with graphs representing usage statistics. To configure the firewall and set up rules, there are many functions available. The initial step involves managing services and filtering some from the network.

### Blocking All Incoming Traffic  
We will start by blocking all incoming (input) traffic related to all services except for commonly used ones:  
- **HTTP (80)**  
- **HTTPS (443)**  
- **SSH (22)**  

In this case, we will use a whitelist approach since the number of allowed protocols is fewer than those to be blocked. We will use the **Pass** action to whitelist ports `80`, `443`, and `22`.

### Steps to Configure Rules in pfSense

Navigate to:
**Firewall → Rules → WAN**

#### Block All Traffic  
1. Click **Add** to create a new rule.  
2. Configure the following:  
   - **Action**: `Block`.  
   - **Protocol**: `Any`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Log**: Enable.

![image](https://github.com/user-attachments/assets/61cca87f-aa7f-4f41-9cff-499b7a5e7bc8)

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
![image](https://github.com/user-attachments/assets/f37dda1b-69c1-41d0-a67b-64132d1cbb2e)

2. **SSH**:  
   - **Action**: `Pass`.  
   
   - **Protocol**: `TCP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `SSH`  
     - **To**: `SSH`.  
   - **Log**: Enable.  

![image](https://github.com/user-attachments/assets/2a3ef5c1-0752-4e16-8870-56443ec15e0b)

---
#### Allow Only Outgoing Connections on Specific Ports (80, 443, 53)  

Navigate to:  
**Firewall → Rules → WAN**

1. **Allow HTTP (Port 80)**:  
   - Click **Add** to create a new rule.  
   - Configure the following:  
     - **Action**: `Pass`.  
      
     - **Protocol**: `TCP`.  
     - **Source**: `Any`.  
     - **Destination**: `Any`.  
     - **Destination Port Range**:  
       - **From**: `80`  
       - **To**: `80`.  
     - **Log**: Enable.  

2. **Allow HTTPS (Port 443)**:  
   - Configure similarly to **HTTP**, but set the port range to:  
     - **From**: `443`  
     - **To**: `443`.  

![image](https://github.com/user-attachments/assets/3694282e-6b88-483e-be47-fa64ffaaae1b)

3. **Allow DNS (Port 53)**:  
   - Configure similarly to **HTTP**, but set:  
     - **Protocol**: `UDP`.  
     - **Destination Port Range**:  
       - **From**: `53`  
       - **To**: `53`.

![image](https://github.com/user-attachments/assets/8d57f787-f611-4366-b0d3-282fc75b9702)

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

![image](https://github.com/user-attachments/assets/745b39c4-d8cd-428e-a77b-a622d16ef678)

2. **FTP (Port 21)**:  
   - **Action**: `Block`.  
   
   - **Protocol**: `TCP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `20`  
     - **To**: `21`.  
   - **Log**: Enable.  

![image](https://github.com/user-attachments/assets/35f78dd5-1291-434a-976a-e83661572231)

3. **SMB (Ports 137-139, 445)**:  
   - **Action**: `Block`.  
   
   - **Protocol**: `TCP`.  
   - **Source**: `Any`.  
   - **Destination**: `Any`.  
   - **Destination Port Range**:  
     - **From**: `137`  
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

![image](https://github.com/user-attachments/assets/f7ea6b9a-5beb-46b8-a253-fffe9d8833b0)

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

![Screenshot From 2025-01-15 23-12-51](https://github.com/user-attachments/assets/43132d0c-3960-418d-a448-2d3f00c39d3c)

### Enable and Configure Connection Logs  

We enabled logs earlier by selecting Log in each rule.

![image](https://github.com/user-attachments/assets/a5ce9ec2-8e01-4377-a26d-feabaedde871)

# Setting Up an IDS/IPS Solution

To level up security, I deploy an IDS (Intrusion Detection System) and IPS (Intrusion Prevention System). These tools help detect and prevent attacks in real-time.

in pfsense we have to download snort as ids

1. Navigate to System → Package Manager → Available Packages
2. search snort.
3. then click install 

![image](https://github.com/user-attachments/assets/f6e58b41-2c63-468f-bccd-ebdae2c599c7)

snort configuration in pfsense  
go to services → SNORT

then we are in snort pannel it contains alot of function and we can manage everything on it 

 configure interfaces
in our case we are focusing on preventing or detecting all what it comes through WAN 
first of all we have to create interfaces in **Snort Interfaces** 
then click Add 

	tick Enable 
	chose interface in our case it's **WAN**	
	then click save 

![image](https://github.com/user-attachments/assets/6b54be2e-4c04-4af0-b7fc-593378f902a3)

### Detecting Port Scanning 
we have to detect any threat by attackers as first phae of an attacker it's reconnaissance which incluse portscanning we have to start by detecting portscanning through the most famous tool which is NMAP 
to download rules we have togo to WAN Categories then select the needed ruleset in our case we need to detect scanning phase of the atacker we gonna tick **emerging-scan.rules** then Save 
then go to WAN Rules → then select **emerging-scan.rules* via Category Selection then Enable all (to enable all rules of that ruleset) → Apply ( to apply that rules) 

![image](https://github.com/user-attachments/assets/7aeac72a-aa82-4d4e-8d83-6f731962e206)

also in snort by default give us some predefined rules which have just to tick to activate it after click on the edit button on the interface then **WAN Preprocs**  → **Portscan Detection** → tick the **use Portscan Detection...** 

![image](https://github.com/user-attachments/assets/fb14fb09-9a6e-4407-8cc3-aa292c71f971)

now if an attacker used nmap or tried to scan ports of server in our network,

![Screenshot From 2025-01-06 18-38-37](https://github.com/user-attachments/assets/73bd320f-48ff-4396-818f-e52530084f54)

we can detect it 

![Screenshot From 2025-01-06 14-22-22](https://github.com/user-attachments/assets/50800378-24c8-42e5-9f1f-2cab23f7b6cd)

### Detecting SSH BruteForce 

also i created a rule that detect ssh bruteforce to prevent like that attacks 

we can add our rule on WAN RULES → Category Selection → chose costum.rules → and add 
```
alert tcp any any -> $HOME_NET 22 (msg:"SSH brute force detected"; flags: S; threshold: type both, track by_src, count 10, seconds 60; sid:10000006; rev:1;)
```
then if attacker tried to bruteforce a ssh server in our network,

![Screenshot From 2025-01-06 18-32-31](https://github.com/user-attachments/assets/7ab6b7a4-a059-4692-93fa-c804e10a654f)

we gonna detect it 

![Screenshot From 2025-01-06 18-41-32](https://github.com/user-attachments/assets/44e3a9bf-21a6-433f-81a7-baf5f1434d2d)

### Configuring OpenVPN on pfSense

By default, pfSense comes with various VPN services. In this guide, we’ll be setting up OpenVPN as a remote access server.

1. **Set Up the OpenVPN Server:**
   - Go to `VPN` → `OpenVPN` → `Wizards`.
   - Select **Type of Server**: `Local User Access` and click **Next**.
   - Fill in the form:
     - **Descriptive Name**: `network_OpenVPN`
     - The country info is optional.
     - Click **Add New CA** and add the server certificate as you did previously. Then click **Add New CA** again.
   - Now fill in the form:
     - **Description**: `Network_OpenVPN`
     - **Endpoint Configuration**:
       - **Protocol**: Select `TCP IPv4 and IPv6 on all interfaces (multihome)`.
       - **Interface**: Choose `LAN` for VPN.
     - In **Tunnel Settings**, configure:
       - **IPv4 Tunnel Network**: `192.168.100.0/24` (IP range for the VPN user).
       - **Local Network**: `192.168.56.0/24` (your local network).
   - Click **Next**, and tick both **Firewall Rule** and **OpenVPN Rule**. Then click **Next** and **Finish**.

   OpenVPN should now be running.

2. **Create a User for VPN Access:**
   - Go to `System` → `User Manager` → `Users`.
   - Click **Add** and fill in the user details (username, full name, password).
   - Enable **Certificate** to create a certificate for the user. 
   - Fill in the form with:
     - **Descriptive Name**: Choose a name for the certificate.
     - **Certificate Authority**: Select the VPN name you created earlier (`network_OpenVPN`).
   - Keep other values as defaults and click **Save**.

3. **Export the VPN Configuration:**
   - The VPN installer is not included by default in pfSense. To install it:
     - Go to `System` → `Package Manager` → `Available Packages`.
     - Install the package `openvpn-client-export`.
   - Once installed, go back to `VPN` → `OpenVPN` → `Client Export`.
   - Select the remote access server you created earlier.
   - Scroll to the bottom of the page and export the VPN configuration for a Linux user:
     - Choose **Bundled Configurations** → **Inline Configurations** → **Most VPN**.
   - Download the `.ovpn` file.

4. **Connect Using OpenVPN:**
   - To connect using the `.ovpn` file, use the OpenVPN command line:
     ```bash
     openvpn pfSense-UDP4-1194-vpnuser-config.ovpn
     ```
   - Normally, this will connect and assign an IP from the **Tunnel Network** you configured earlier. You should be able to ping devices on the network.

**Note**: In my case, this setup didn’t work, so I had to switch to another VPN service, Twingate.

### Let's Configure Twingate

For this setup, we need a machine in the same network as the one we want to access remotely. In my case, I have an Ubuntu machine in VirtualBox on a `vboxnet` network, which contains the same pfSense machine.

1. **Create an Account:**
   - On the machine, browse to [Twingate Signup](https://auth.twingate.com/signup-v2).
   - Sign up for an account. During signup, you’ll be asked to provide a network name. Choose a name, like "networkproject" (this name is crucial for anyone wanting to connect to your network via VPN).
   - After signing up, log in to your account.

   During account creation, youll be prompted for some information and given multiple choices to connect your machine with Twingate. I chose the bash script command, which was simple and effective. I copied the script and pasted it into the terminal to complete the process.

2. **Configure the Network:**
   - In the Twingate panel, go to **Resources** and click **+Resource** to add the network you want to be accessible via the VPN.
   - Set the following:
     - **Label**: `name_of_the_resource`
     - **Address**: `192.168.56.0/24`
   - Click **Create Resource**.

3. **Configure the Twingate Server:**
   - On the Ubuntu machine, after installing Twingate (as per the previous step), run the command:
     ```bash
     twingate setup
     ```
   - The setup will ask for information, including the network name you created earlier (`networkproject`). You’ll also be asked to approve some agreements.

4. **Test Connectivity on the Client:**
   - To connect, download the Twingate VPN app on the client machine. I tested connectivity via my iPhone:
     - Go to the App Store and download the Twingate mobile app.
     - After opening the app, enter the network name (`networkproject`) and grant the app VPN permissions.
     - The phone will connect to the pfSense network, even while using mobile data (not the local network). The VPN works fine.

   **Note**: The first attempt didn’t work. After troubleshooting, I realized there was no routing on the Twingate server. To fix this, I added routing with the following command:
   ```bash
   ip route add 192.168.56.103 via 192.168.56.101 dev enp0s8
   ```
   
• 192.168.56.103 → pfSense IP

• 192.168.56.101 → My machine IP as the gateway

• enp0s8 → Network interface
