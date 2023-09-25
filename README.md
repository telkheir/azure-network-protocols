[IMAGE - banner - some network concept map or wireshark or w/e]

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
          For this lab, we need to create a resource group to set up 2 virtual machines in Azure, one running Windows and one running Ubuntu. They need to be in the same resource group, region, and running on the same network. I assigned both machines 2 vcpus and at least 8GB of memory. Designate each machine a username and password. All other settings can be left on default. If you allow the first machine to complete deployment, the second one will be placed in the same network by default. 
          <br>
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
          [IMAGE - wireshark]
          <br><br>
          WireShark is an open-source packet-analyzer, we will use it to examine how some network protocols work between the two virtual machines we have. Install and open WireShark to allow it to start capturing packets. First, we are going to filter out ICMP traffic only.
          <br><br>
          [IMAGE - icmp filter]
          <br><br>
          Using the command, ping the private IP of the Ubuntu virtual machine and observe the actions of the ping request in WireShark.
          <br><br>
          [IMAGE - cmd line and maybe wireshark results]
          <blockquote>
              Note: The Ubuntu VM is not connected to the internet, so you can't ping its public IP address.
          </blockquote>
          <br>
          After examining how the computers send the ICMP data back and forth between each other after a single ping request, I initiated a continuous ping and returned to observe the data packets in wireshark.
          <br><br>
          [IMAGE- cmd line continuous ping]
          <br><br>
          Next, I set up a firewall to block any incoming ICMP requests to the Ubuntu virtual machine. To do this, return to Azure and open the Network Security Group of the Ubuntu virtual machine. Navigate to the Inbound Security Rules page and click "Add". 
          <br><br>
          [IMAGE - navigation & rules overview]
          <br><br>
          Return to the Windows VM to see the ping fail to return any data in the command line and WireShark no longer recieving any data packets back from the Ubuntu VM.
          <br><br>
          [IMAGE - failed ping in cmd and wireshark]
          <br><br>
          Turn off the new rule and observe the requests going through once again. Turn off the continuous ping in the command line with CTRL + C.
          <br>
      </li>
      <li><h3 id = "step_3">Observing SSH traffic</h3>
          In WireShark, filter the packets by SSH (TCP port 22). This can be done by typing "tcp port == 22J" in the search bar.
          <br><br>
          [Image - filtered by SSH/port 22 data packets]
          <br><br>
          Now we will use the SSH command to access the Ubuntu virtual machine. To do so, we need the username and password we assigned it, as well as its private IP address. In the command prompt, type in "ssh username@privateip" and enter the password when prompted.
          <br><br>
          [Image - command prompt]
          <br><br>
          We can observe the data traffic in WireShark whenever we type in commands through SSH. Type 'Exit' in the command prompt to end the SSH connection.
      </li>
      <li><h3 id = "step_4">Observing DHCP traffic</h3>
          Now we will filter for DHCP, or UDP ports 67 and 68, traffic in WireShark. In the command prompt, type "ipconfig / renew". This command tells our Windows virtual machine to broadcast on the network a request to the DHCP servers for a new IP address.
          <br><br>
          [image - cmd prompt and also some wireshark same image only one]
      </li>
      <li><h3 id = "step_5">Observing DNS traffic</h3>
          Filter by DNS (port 53) traffic in WireShark and use nslookup on the command line to obtain the IP address(es) of Google.
          <br><br>
          [image - command line and wireshark in back]
      </li>
      <li><h3 id = "step_6">Observing RDP traffic</h3>
          Filter by RDP traffic (TCP port 3389) in WireShark. Traffic should be non-stop because Remote Desktop Protocol is continuously showing us a livestream of the Windows virtual machine we are presently working on.
          <br><br>
          [image - wireshark ig]
      </li>
      <li><h3 id = "step_7">Cleaning up</h3>
          End the remote connection and return to Azure to delete both virtual machines and the resource group they reside in.
      </li>
    </ol>
