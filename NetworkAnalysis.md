# Network Analysis

### Time Thieves
At least two users on the network have been wasting time on YouTube. Usually, IT wouldn't pay much mind to this behavior, but it seems these people have created their own web server on the corporate network. So far, Security knows the following about these time thieves:

- They have set up an Active Directory network.
- They are constantly watching videos on YouTube.
- Their IP addresses are somewhere in the range 10.6.12.0/24.

You must inspect your traffic capture to answer the following questions:

1. **What is the domain name of the users' custom site?** <br>
Since we know what IP range the "Time Thieves" are using, a simple Wireshark filter can be used to filter out the packet captures. `ip.addr == 10.6.12.0/24`
Using this filter, we now see the domain name in use: `Frank-n-Ted-DC.frank-n-ted.com` <br>

![Domain Name](/images/pcap1.png)<br>

2. **What is the IP address of the Domain Controller (DC) of the AD network?**<br>

Opening a packet from source `Frank-n-Ted-DC.frank-n-ted.com` shows the IP address of : `10.6.12.203`<br>

![IP Address](/images/pcap2.png)<br>

3. **What is the name of the malware downloaded to the 10.6.12.203 machine? Once you have found the file, export it to your Kali machine's desktop.**<br>

Once again using filters will pinpoint what were are looking for. If we use a filter for the IP range in question and the http method, we will find what we are looking for: `ip.addr == 10.6.12.0/24 && http.request.method == GET`<br>

We should see a `GET` request for a suspicious file named `june11.dll`. From there we go to File > Export Objects > HTTP... Once Wireshark finishes searching the objects we can further filter using the dll name and save it to the desktop.<br>

![Found Malware](/images/pcap3.png)<br>

4. **Upload the file to VirusTotal.com. What kind of malware is this classified as?**<br>

VirusTotal is a useful tool that analyzes suspicious files and URLs and can help identify malicious files. We have one here! Once we upload the suspicious file we found **virustotal.com** gives us the results: `june11.dll` is a trojan virus.<br>

![Scan Results](/images/pcap4.png)

### Vulnerable Windows Machines
The Security team received reports of an infected Windows host on the network. They know the following:

- Machines in the network live in the range 172.16.4.0/24.
- The domain mind-hammer.net is associated with the infected computer.
- The DC for this network lives at 172.16.4.4 and is named Mind-Hammer-DC.
- The network has standard gateway and broadcast addresses.

Inspect your traffic to answer the following questions:

1. **Find the following information about the infected Windows machine:**<br>

Since we know the IP range and the IP address of the Domain Controller, we can use that in our filter to find the hostname, IP address and the MAC of the infected machine: `ip.addr == 172.16.4.4`. We can now see the details needed:
    - Hostname: `Rotterham-PC`
    - IP Address: `172.16.4.205`
    - MAC: `00:59:07:b0:63:a4`<br>

![Infected Machine](/images/wpcap1.png)<br>

2. **What is the username of the Windows user whose computer is infected?**<br>

Since we are looking for a username being used in Active Directory, we can use a filter that highlights packets with kerberos data. So the filter can include that source IP address  and kerberos string: `ip.src==172.16.4.205 && kerberos.CNameString`. Once we filter out the appropriate packets, we can see under the column **CNameString** the user that infected the device: **matthijs.devries**<br>

![User Responsible](/images/wpcap2.png)<br>

3. **What are the IP addresses used in the actual infection traffic?**<br>

Filterring the IP address in question and sorting the traffic to highest amount of packets, we can see that IP address `172.16.4.205` and `185.243.115.84` are talking the most. We can now use a filter to isolate these two IP addresses: `ip.addr == 172.16.4.205 && ip.addr == 185.243.115.84`. Checking IP traffic involving `185.243.115.84` we can see that there are many POST sent of empty.gif without GET requests that go with the POST. This is a red flag.<br>

![Infected Devices](/images/wpcap3.png)<br>

### Illegal Downloads
IT was informed that some users are torrenting on the network. The Security team does not forbid the use of torrents for legitimate purposes, such as downloading operating systems. However, they have a strict policy against copyright infringement.

IT shared the following about the torrent activity:
- The machines using torrents live in the range 10.0.0.0/24 and are clients of an AD domain.
- The DC of this domain lives at 10.0.0.2 and is named DogOfTheYear-DC.
- The DC is associated with the domain dogoftheyear.net.

Your task is to isolate torrent traffic and answer the following questions:

1. **Find the following information about the machine with IP address 10.0.0.201:**<br>

Using a filter can help find the device and user information. Since we know the IP address, we can use it with a kerberos filter: `ip.addr == 10.0.0.201 && kerberos.CNameString`. This gives us the MAC, Hostname and username: 
    - MAC address: `00:16:17:18:66:c8`
    - Windows username: `elmer.blanco`
    - Host Name (OS version): `BLANCO-DESKTOP`<br>
![User and Device](/images/ipcap1.PNG)<br>

2. **Which torrent file did the user download?**<br>

We can filter the same IP address and combine that with a filter that looks for a "torrent" file: `ip.addr == 10.0.0.201 && http.request.uri contains "torrent"`. We can now see which file the user downloaded: **Betty_Boop_Rythm_on_the_Reservation.avi.torrent**.<br>

![Torrent File](/images/ipcap2.png)