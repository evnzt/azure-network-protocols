<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines (VMs)</h1>

This lab uses **Wireshark** to observe network traffic between **Azure VMs** and experiments with **Network Security Groups (NSGs)** to see how rules affect connectivity.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop Protocol (RDP)
- Wireshark (protocol analyzer)
- Command-Line Tools
- Common protocols: SSH, RDP, DNS, HTTP/S, ICMP
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2) - *Client VM (Wireshark + RDP)*
- Ubuntu Server 20.04 - *Linux VM (SSH target)*

<h2>High-Level Steps</h2>

- Create a Resource Group and two VMs (Windows 10 + Ubuntu) in the same VNet/subnet
- Use Wireshark on Windows to capture traffic
- Generate traffic (ICMP, SSH, DHCP, DNS, RDP) and observe filters
- Use NSGs to block/allow traffic and watch the impact in real time

<h2>Actions and Observations</h2>

__Step 1__: Set up your virtual environment

*Two VMs in the same Resource Group and VNet.*

1. **Create a Resource Group** (e.g., NSG-network). Choose a nearby region (e.g., West US 2)
2. **Create the Windows 10 VM** (client). Let Azure create a new VNet + subnet. Use password auth.
3. **Create the Ubuntu 20.04 VM** in the same Resource Group and VNet/subnet. Use password auth.
4. (*Optional*) Inspect the network topology in **Network Watcher**.
<img width="923" height="883" alt="image" src="https://github.com/user-attachments/assets/2fbe1241-1ff3-44df-a3b7-28cb98085fda" />

---

__Step 2__: Observe ICMP (Ping)

*See ping traffic in **Wireshark** and how an **NSG** rule changes it.*

1. **RDP** into the Windows 10 VM, install and open **Wireshark**.
2. Set the display filter to **ICMP**
3. Find the private IP of the **Ubuntu VM** and ping it from Windows:
   
    ```cmd
   ping <UBUNTU_PRIVATE_IP>
   ```
   Watch requests/replies in **Wireshark**.
4. Try pinging a public site:
   
    ```cmd
   ping www.google.com
   ```
   Watch requests/replies in **Wireshark**.
5. Start a continuous ping (leave it running):
   
    ```cmd
   ping -t <UBUNTU_PRIVATE_IP>
   ```
<img width="1064" height="807" alt="image" src="https://github.com/user-attachments/assets/a4326ce9-1606-4778-b58d-9051be3ddda1" />
</p>

6. Block **ICMP** to **Ubuntu** using its **NSG**:
   * Go to the Ubuntu VM’s Network interface → NSG → Inbound security rules → Add
   * **Source**/**Destination**: Any ➤ **Protocol**: ICMP ➤ **Action**: Deny ➤ **Priority**: (low number, e.g., 100) → Save
   * Back on Windows, observe ping timeouts and Wireshark drops
<img width="553" height="879" alt="image" src="https://github.com/user-attachments/assets/71a91656-3c6a-4aba-a5f9-d100ddf46f87" />
</p>

7. Re-enable **ICMP** (disable or remove the deny rule). Pings should resume.

---

__Step 3__: Observe SSH

*Watch encrypted **SSH** traffic to **Ubuntu**.*

<img width="680" height="91" alt="image" src="https://github.com/user-attachments/assets/86e42c0f-18ea-499d-8b16-7c379a4d0e66" />

1. In **Wireshark**, set filter to **SSH**
2. From **Windows**, connect to **Ubuntu**:
   
    ```cmd
   ssh <username>@<UBUNTU_PRIVATE_IP>
   ```
3. Run a few commands (ls, pwd, etc.) and watch the encrypted packets.
4. Exit with:
   
    ```cmd
   exit
   ```

---

__Step 4__: Observe DHCP

*See the DHCP lease workflow.*

<img width="680" height="91" alt="image" src="https://github.com/user-attachments/assets/460b8a31-2b69-458e-a08f-3caa993860da" />

1. In **Wireshark**, set filter to **BOOTP** (Wireshark labels DHCP as BOOTP)
2. On **Windows**:

    ```cmd
   ipconfig /renew
   ```
   Observe the DISCOVER/OFFER/REQUEST/ACK exchange.
   
---

__Step 5__: Observe DNS

*Watch DNS queries and responses.*

<img width="680" height="91" alt="image" src="https://github.com/user-attachments/assets/686e9519-9749-4964-88f4-c149695386cf" />

1. In **Wireshark**, set filter to **DNS**
2. On **Windows**:

    ```cmd
   nslookup google.com
   nslookup disney.com
   ```
3. See the A/AAAA lookups and responses.

---

__Step 6__: Observe RDP

*Recognize continuous RDP traffic.*

<img width="680" height="91" alt="image" src="https://github.com/user-attachments/assets/838e2f63-3d24-4954-9c8f-7ed94d8b1dc0" />

1. In **Wireshark**, set filter to **tcp.port==3389** (Wireshark label for RDP)
2. You’ll see a constant stream. **RDP** continuously sends screen updates/input events, so traffic persists even when idle.

---

Congratulations! Hopefully, you were able to inspect all traffic without any hiccups.

---

## Troubleshooting

* If your filters show nothing, confirm you’ve selected the right network interface in Wireshark and that both VMs are on the same VNet/subnet.

---

## Cleanup

* Delete the **Resource Group(s)** and **VMs** in Azure once done to avoid charges.

<h2>Closing Thoughts</h2>

Completing this NSG and traffic inspection lab made networking feel real. I captured ICMP, SSH, DHCP, DNS, and RDP in Wireshark, then tweaked NSG rules to allow/deny traffic and monitored the impact in real time. That end-to-end loop of *configure* → *test* → *observe* → *fix* mirrors how IT teams secure, troubleshoot, and operate networks every day.
</p>
<br />
