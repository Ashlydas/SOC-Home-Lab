# SOC-Home-Lab
Home lab for security monitoring using Wazuh, Sysmon, and Windows 11.

INSTALL 
WAZUH.OVM, VMWARE, KALI LINUX

1. Network configuration 
This is important for your SOC lab.

We'll eventually have:
                 Internet
                    │
                  NAT
                    │
             ┌─────────────┐
             │    Wazuh    │
             │     VM      │
             └──────┬──────┘
                    │
             PRIVATE SOC LAN
                    │
             ┌──────┴──────┐
             │             │
           Kali         Windows
          Attacker        Agent

Wazuh Adapter 1
Go to:

Wazuh → Settings → Network Adapter

Set:
Network connection: NAT

This gives Wazuh internet access for updates.
2. Create your private SOC network
In VMware Workstation:

Edit → Virtual Network Editor

Create/use a Host-only network.

For example:
VMnet2
Type: Host-only
Subnet: 192.168.56.0
Mask: 255.255.255.0

You can enable VMware's DHCP.

lab will then be isolated from your normal network.
3. Add second network adapter to Wazuh
Go to:

Wazuh VM → Edit virtual machine settings

Click:

Add → Network Adapter

Set:
Network connection:
Custom: VMnet2

Now Wazuh has:
Adapter 1 → NAT
Adapter 2 → VMnet2

4. Configure Kali
Open your Kali VM settings.

For Kali, add/use a second network adapter:
Network Adapter 1 → NAT
Network Adapter 2 → Custom: VMnet2

So:
Kali
 ├── NAT       → Internet
 └── VMnet2    → SOC Lab

Now Kali and Wazuh can communicate through the private lab network.
5. Check Kali's IP
Start Kali and run:
ip addr

You should see an address on the VMnet2 interface such as:
192.168.56.128

The exact IP will probably be different.
6. Start Wazuh
Start the Wazuh VM.

Login using the credentials provided in the current Wazuh VM documentation.

Then run:
ip addr

Find the IP belonging to the VMnet2 interface.

For example:
192.168.56.129

7. Test Kali → Wazuh
From Kali:
ping 192.168.56.129

Replace the address with your Wazuh VM's actual VMnet2 IP.

You want:
64 bytes from 192.168.56.129

If you get replies, your private SOC network is working. ✅
8. Access Wazuh Dashboard
From your Windows browser:
https://WAZUH-IP

For example:
https://192.168.56.129

You'll get a certificate warning because the default installation uses a self-signed certificate.

Proceed to the Wazuh dashboard.
Your final setup
                    WINDOWS 11
                    8 GB RAM
                        │
          ┌─────────────┴─────────────┐
          │                           │
       VMware                     VMware
          │                           │
    ┌─────┴─────┐               ┌─────┴─────┐
    │   Kali    │               │   Wazuh   │
    │    VM     │               │    VM     │
    │  ~2 GB    │               │  ~2-3 GB  │
    └─────┬─────┘               └─────┬─────┘
          │                            │
          └──────── VMnet2 ────────────┘
                192.168.56.0/24
                   PRIVATE LAN


Step 1 — Test connectivity

On your Windows 11 host, open PowerShell and run:
ping 192.168.217.131

You should see replies like:
Reply from 192.168.217.131: bytes=32 time<1ms TTL=...

Then test the Wazuh dashboard:
https://192.168.217.131

Open that in your Windows browser.
Step 2 — Check the network

Wazuh Windows Agent

Download the Windows 64-bit MSI installer.
During installation
You'll see fields for the Wazuh manager:

Wazuh Manager address:
192.168.217.131

For the agent name, use something simple:
Windows-11-Lab

Finish the installation.
Step 3 — Start the Wazuh Agent
Open PowerShell as Administrator and run:
NET START WazuhSvc

You should get something similar to:
The Wazuh service was started successfully.

Step 4 — Check Wazuh Dashboard
Go to your Wazuh dashboard:
https://192.168.217.131

Then go to:

Agents → Agent management

You should eventually see:
Windows-11-Lab
Status: Active


Since you're doing a lab, let's generate it from the Wazuh VM.
1. On the Wazuh VM
Open the terminal and run:
sudo /var/ossec/bin/manage_agents

You'll get a menu.

Choose:
A

to Add an agent.

When asked for the agent name, enter:
Windows-11-Lab

For the IP address, enter your Windows machine's IP if you know it.

Then you'll see something like:
Agent ID: 001
Agent name: Windows-11-Lab
Agent IP: ...

Choose:
E

to Extract the key.

Enter:
001

(or whatever agent ID Wazuh gave you).

Wazuh will display a long authentication key.
2. Copy that key
On Windows, open:

Wazuh Agent → Manage → Authentication

or the agent enrollment/configuration screen you're currently looking at.

Paste the key there.

3. Restart the agent
After saving the key, open PowerShell as Administrator:
Restart-Service WazuhSvc

Then check the Wazuh dashboard:

Agents → Agent management

You want:
Windows-11-Lab
Active

Windows → Wazuh 

Now we're adding:

Windows → Sysmon → Wazuh

Step 1 — Download Sysmon
Go to Microsoft's official Sysinternals page:

Microsoft Sysmon download

Download the ZIP.

Step 2 — Extract it
Extract the ZIP somewhere easy, for example:

C:\Sysmon
Inside that folder you should see:

Sysmon.exe
Sysmon64.exe
Sysmon64a.exe


Step 3 — Open PowerShell as Administrator
On Windows:

Start → search PowerShell → right-click → Run as administrator

Then type:

cd C:\Sysmon

Press Enter.

Then check that Sysmon is there:

dir

You should see Sysmon64.exe.

Step 4 — Install Sysmon
For now, let's install Sysmon without a custom configuration. This keeps things simple.

Run:

.\Sysmon64.exe -accepteula -i




<img width="1919" height="975" alt="Screenshot 2026-09-09 163726" src="https://github.com/user-attachments/assets/318b3c42-936d-4857-a737-53de9e22b7e7" />

 
<img width="1910" height="915" alt="Screenshot 2026-10-26 163516" src="https://github.com/user-attachments/assets/05a6845f-e5e1-4d31-80f9-6f9899758b11" />


<img width="1907" height="973" alt="Screenshot 2026-10-26 163535" src="https://github.com/user-attachments/assets/0274034f-c28c-4172-a2ff-10768d0641e8" />
















