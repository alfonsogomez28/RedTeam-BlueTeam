# Blue Team: Summary of Operations

## Table of Contents
- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic & Behavior
- Suggestions for Going Further

### Network Topology
_TODO: Fill out the information below._

The following machines were identified on the network:
- ELK Server
  - **Ubuntu Linux**:
  - **ELK Cluster**:
  - **192.168.1.100**:
- Capstone
  - **Ubuntu Linux**:
  - **Capstone**:
  - **192.168.1.105**:
- TARGET1
  - **Debian Linux**:
  - **Web Server**:
  - **192.168.1.110**:
- TARGET2
  - **Debian Linux**:
  - **Web Server**:
  - **192.168.1.115**:
- Kali
  - **Kali Linux**:
  - **Attacking Machine**:
  - **192.168.1.90**:

### Description of Targets

The target of this attack was: `Target1` at IP address: 192.168.1.110.

Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:


### Monitoring the Targets

Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

#### HTTP Error Alert
![http-error-alert](/images/alert1.png)
Alert 1 is implemented as follows:
  - **Metric**: `WHEN count() GROUPED OVER top 5 'http.response.status_code`
  - **Threshold**: `IS ABOVE 400`
  - **Vulnerability Mitigated**: Enumeration or Brute Force Attack
  - **Reliability**: This alert is highly reliable. When measuring error codes with status 400 and above, this will filter out the normal status responses.

#### CPU Usage Monitor
![cpu-usage-alert](/images/alert2.png)
Alert 2 is implemented as follows:
  - **Metric**: `WHEN max() of system.process.cpu.total.pct OVER all documents`
  - **Threshold**: `IS ABOVE 0.5`
  - **Vulnerability Mitigated**: Malware or keylogger taking up system resources
  - **Reliability**: This alert has a medium reliability. This alert may, on occation flag as a false positive if user runs resource intense applications.

#### HTTP Request Size Monitor
![http-monitor](/images/alert3.png)
Alert 3 is implemented as follows:
  - **Metric**: `WHEN sum() of http.request.bytes OVER all documents`
  - **Threshold**: `IS ABOVE 3500`
  - **Vulnerability Mitigated**: Code injection in requests or denial of service attack (ex: CRLF, DDoS and XSS)
  - **Reliability**: This alert has a medium reliability. It may cause a false positive if requests are legitimate traffic.

### Suggestions for Going Further

The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats, identified by the alerts above. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:

- Vulnerability 1
  - **Patch**: Wordpress Core updates and hardening
  - **Why It Works**: By installing latest Wordpress updates, it can fix security issues and known vulnerabilities. Also Wordpress can be hardenened by using tools like Wordfence and disabling unused features and plugins. Additionally, if REST API is enabled, but not needed, WPScan takes advantage of this and is used to enumerate Wordpress. Disabling this feature can reduce this vulnerability.

- Vulnerability 2
  - **Patch**: Antivirus/Antimalware software
  - **Why It Works**: High CPU usage is usually a good indicator that the device has been infected with malware or a keylogger virus/software. Having a good AV product installed on the host device can help mitigate this type of attack.

- Vulnerability 3
  - **Patch**: Deploy a WAF (Web Application Firewall)
  - **Why It Works**: WAFs are good at detecting, blocking and sanitizing HTTP traffic. Security rules can be configured to filter out specific traffic patterns that are consistent with attacks such as SQL-Injections, XSS, and DDoS.
