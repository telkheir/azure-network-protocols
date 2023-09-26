<h1>Inspecting Traffic between Virtual Machines in Azure</h1>

This tutorial will examine various network traffic protocols between two virtual machines created in Azure.


<h2>Environments and Technologies Used</h2>
    <ul>
      <li>Microsoft Azure</li>
      <li>Virtual Machines (Azure)</li>
      <li>Remote Desktop</li>
      <li>WireShark</li>
      <li>Network Security Groups</li>
      <li>Various Network Protocols (RDP, ICMP, SSH, etc)</li>
    </ul>

<h2>Operating Systems Used</h2>
    <ul>
      <li>Windows 10 (22H2)</li>
      <li>Ubuntu Server 20.04</li>
    </ul>

<h2>Navigation</h2>
    <ol>
      <li><a href = "#step_1">Setting up resources</a></li>
      <li><a href = "#step_2">Observing ICMP traffic</a></li>
      <li><a href = "#step_3">Observing SSH traffic</a></li>
      <li><a href = "#step_4">Observing DHCP traffic</a></li>
      <li><a href = "#step_5">Observing DNS traffic</a></li>
      <li><a href = "#step_6">Observing RDP traffic</a></li>
      <li><a href = "#step_7">Cleaning up</a></li>
    </ol>

<h2>Tutorial</h2>
    <ol>
      <li><h3 id = "step_1">Setting up resouces</h3>
          For this lab, we need to create a resource group to set up 2 virtual machines in Azure, one running Windows and one running Ubuntu, I named them "Win10VM" and "LinuxVM". They need to be in the same resource group, region, and running on the same network. I assigned both machines 2 vcpus and at least 8GB of memory. Designate each machine a username and password. Note that when creating the Linux virtual machine, the default authentication option is an SSH public key but for this lab, we just want to use a password(see image below). All other settings can be left on default. If you allow the first machine to complete deployment, the second one will be placed in the same network by default. 
          <br><br>
          <img width="626" alt="linux-vm-user-pass" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/c5efb898-147a-4de1-ad4b-98fd74da2466">
          <br><br>
          Once both machines are done being created, there should be a total of 11 files in the resource group:
              <ul>
                  <li>2 disks</li>
                  <li>2 network interface cards</li>
                  <li>2 network security groups</li>
                  <li>2 public IP addresses</li>
                  <li>2 virtual machines</li>
                  <li>1 virtual network</li>
              </ul>
          <br>
          <img alt="network-protocols-vm-files" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/c7047550-4830-456b-ac8b-1a0767678c15">
          <br>
      </li>
      <li><h3 id = "step_2">Observing ICMP traffic</h3>
          Using Remote Desktop, I accessed the virtual machine running Windows and downloaded WireShark onto it.
          <br><br>
          <img width="960" alt="wireshark" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/8624b1a9-27a5-42cf-911c-f6904bbb5081">
          <br><br>
          WireShark is an open-source network protocol analyzer. We are going to use it to examine how some network protocols work between the two virtual machines we have. Install and open WireShark and click on the blue fin icon in the top left corner to start capturing packet. There should be an influx of packets being sent and recieved. You can click the red square to stop capturing packet, but we want to continue capturing to move on to the next step.
          <br><br>
          <img width="537" alt="wireshark-start-cap" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/ca28f4ce-6c84-4513-bed0-176021ce193f">
          <img width="537" alt="wireshark-stop-cap" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/0673ccfd-028e-41c6-953b-9bc20b49030b">
          <br><br>
          First, we are going to filter out ICMP traffic only. You can do this by typing "icmp" in the search bar. There should be no packets under this protocol category yet.
          <br><br>
          <img width="618" alt="wireshark-icmp-empty" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/31acfdb9-9373-4330-9b4d-44b6b83b73f8">
          <br><br>
          Using the command prompt, ping the private IP of the Ubuntu virtual machine and observe the actions of the ping request in WireShark. You can get the private IP address from the Virtual Machines Directory in Azure.
          <br><br>
          <img width="960" alt="wireshark-icmp-cmd1" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/5d699a27-60eb-453b-bc00-0a6b4cd8fdf9">
          <blockquote>
              Note: The Ubuntu VM is not connected to the internet, so you can't ping its public IP address.
          </blockquote>
          <br>
          After examining how the computers send the ICMP data back and forth between each other after a single ping request, I initiated a continuous ping and returned to observe the data packets in wireshark.
          <br><br>
          <img width="960" alt="wireshark-icmp-cmd-cont" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/818e4d4b-caa0-4d86-9289-78f2682d2bfc">
          <br><br>
          Next, I set up a firewall to block any incoming ICMP requests to the Ubuntu virtual machine. To do this, return to Azure and open the Networking tab of the Linux VM. Click on "Add inbound port rule". Set the protocol to ICMP, the action to Deny, and add the rule.
          <br><br>
          <img width="960" alt="network-linux-nav" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/a18d0033-2e17-4191-8fe0-0cd41ced8688">
          <img width="710" alt="network-linux-icmp-rule" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/3386f92c-2182-47f4-8038-863949ddcb58">
          <br><br>
          Return to the Windows VM to see the ping fail to return any data in the command line and WireShark no longer recieving any data packets back from the Ubuntu VM.
          <br><br>
          <img width="960" alt="wireshark-icmp-blocked" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/3454c96c-a04a-4371-b93e-78ae5a49df50">
          <br><br>
          Turn off the new rule in Azure by setting the action back to Allow, click Add, and observe the echo requests going through once again. Turn off the continuous ping in the command line with CTRL + C.
          <br><br>
          <img width="960" alt="wireshark-icmp-allow" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/786350fa-51fe-496f-8116-02214d23fbd1">
          <br>
      </li>
      <li><h3 id = "step_3">Observing SSH traffic</h3>
          In WireShark, filter the packets by SSH. This can be done by typing either "tcp.port == 22" or simply "ssh" in the search bar.
          <br>
          We're going to use SSH to access the Linux virtual machine. To do so, we need the username and password we assigned it, as well as its private IP address. In the command prompt, type in "ssh username@privateip" and enter the password when prompted.
          <br><br>
          <img width="960" alt="wireshark-ssh-linux" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/bf435d85-3cb0-4316-bd6d-1bd7992b8ecb">
          <br><br>
          We can observe the data traffic in WireShark whenever we type in commands through SSH. Type 'Exit' in the command prompt to end the SSH connection.
      </li>
      <li><h3 id = "step_4">Observing DHCP traffic</h3>
          Now we will filter for DHCP, or UDP ports 67 and 68, traffic in WireShark. In the command prompt, type "ipconfig/renew". This command tells our Windows virtual machine to broadcast on the network a request to the DHCP servers for a new IP address. In the image below, we can see two packets in Wireshark, one for our request and a second for the server's acknowledgement of our request.
          <br><br>
          <img width="960" alt="wireshark-dhcp" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/b68f1d95-7e41-48cb-8fe6-8dc8e8276417">
      </li>
      <li><h3 id = "step_5">Observing DNS traffic</h3>
          Filter by DNS (UDP port 53) traffic in WireShark and use nslookup in the command line to obtain the IP address(es) of a public website. I chose disney.com for my example.
          <br><br>
          <img width="960" alt="wireshark-dns" src="https://github.com/telkheir/azure-network-protocols/assets/145223639/08f82405-c2e2-4a49-a503-4d4d6f3fc59e">
      </li>
      <li><h3 id = "step_6">Observing RDP traffic</h3>
          Filter by RDP traffic (TCP port 3389) in WireShark. Traffic should be non-stop because Remote Desktop Protocol is continuously showing us a livestream of the Windows virtual machine we are presently working on.
      </li>
      <li><h3 id = "step_7">Cleaning up</h3>
          End the remote connection and return to Azure to delete both virtual machines and the resource group they reside in.
      </li>
    </ol>
