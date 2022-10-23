<h1>Using a SIEM (Microsoft Sentinel) to Analyze Failed RDP Login Attempts on a Honeypot</h1>




<h2>Project Summary</h2>

I set up a virtual machine as a honeypot using Microsoft Azure. After altering firewall rules on the vm, I used a Powershell script to collect IP addresses for each failed login attempt. The script used the [ipgeolocation API](https://ipgeolocation.io/) to also collect the location of each of the IP addresses. With the information, I created a Microsoft Sentinel workbook to plot the IP addresses from the failed logins on a map. I began to get curious about the usernames the attackers most commonly used so I added a bar chart to the workbook to visualize the number of usernames that were used. I also added a chart to break down which usernames were used most by country.
<br/>
<p align="center">
<br/>
<img src="https://i.imgur.com/iE6tCGO.png" height="80%" width="80%" alt="Failed RDP Login Analysis"/>
<br />
</p>

<h2>How I Did It</h2> 

1. I started by creating and managing my personal Microsoft Azure account (Link for free account set up: https://azure.microsoft.com/en-us/free/)

2. Set up a Windows 10 Pro virtual machine (VM). I added a top-priority rule to the VM’s firewall to allow all inbound traffic from any host so the VM could act as a honeypot.

<p align="center">
<br/>
<img src="https://imgur.com/LgBz7wK.png" height="80%" width="80%" alt="Failed RDP Login Analysis"/>
<br />
</p>

3. Set up Log Analytics Workspace (LAW) so I could track the number of login attempts. Connected the LAW to the virtual machine.

<p align="center">
<br/>
<img src="https://imgur.com/0V9sbrV.png" height="80%" width="80%" alt="Failed RDP Login Analysis"/>
<br />
</p>

4. Once the VM and LAW were connected, I logged into the VM and set up a Powershell script ([the script is from Josh Madakor](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1)) to capture the failed logins. In order to get geographical data for the IP addresses, I used the [ipgeolocation API](https://ipgeolocation.io/), adding my API Key to the Powershell script.

5. I tested the script by attempting to log into the VM with incorrect credentials to confirm that the script was indeed collecting logs for my attempts - it was!
   
    - Something cool to note -  within minutes of me running the script, it collected data on a failed login attempt from France, which was really encouraging to see so soon in the project setup.

<p align="center">
<br/>
<img src="https://imgur.com/XQHJ3FF.png" height="80%" width="80%" alt="Failed RDP Login Analysis"/>
<br />
</p>

6. I created a custom log in Log Analytics Workspace to capture the logs from the PowerShell script. This included extracting elements from the logs to create custom fields. The Powershell script included a batch of sample logs. I used these samples (and the actual logs that started to come through at this point) to train the extraction algorithm so that it would consistently pick the correct element from each log.

<p align="center">
<br/>
<img src="https://imgur.com/rjFTGlk.png" height="80%" width="80%" alt="Failed RDP Login Analysis"/>
<br />
</p>

7. Once the extraction algorithm was trained, I went to Microsoft Sentinel to create a workspace that would show a heat map with login attempts from around the globe. This included creating a query that would pull the IP geolocation information from the logs, as well as setting up the visualization of the query results to be a map based on the Longitude and Latitude coordinates.

<p align="center">
<br/>
<img src="https://imgur.com/x9PerKi.png" height="80%" width="80%" alt="Failed RDP Login Analysis"/>
<br />
</p>

8. After reviewing the map, I started to wonder about which usernames are being used most frequently for these login attempts. I also wondered if there was any correlation between the username used and the attacking IP’s country. I decided to add a couple of bar charts to the SIEM workbook to help visualize the data I already had.

9. Finally, I added the Microsoft Sentinel workbook to a Dashboard in Microsoft Azure:

<p align="center">
<br/>
<img src="https://imgur.com/iE6tCGO.png" height="80%" width="80%" alt="Failed RDP Login Analysis"/>
<br />
</p>

<h2>Observations and Notes</h2>

- Within 5 minutes of adding the Powershell script to the vm, there was a login attempt from an IP address in France, which was exciting. This tells me that it doesn’t take long for an attacker to find an endpoint.

- It’s clear most login attempts are from Russia and the Netherlands. At first glance, someone could assume that there are a lot of attackers from these countries, however, there is only one IP address from each country. Since there are thousands of attempts from each IP, I believe the attackers were using brute force techniques in an attempt to gain access to the honeypot.

- “Administrator” (and variants like “admin”) was the most frequently used usernames for attacks. This makes sense because if an attacker wants to own a machine, the administrator account is the one to get access to.

- At least a couple IP addresses only have 1 failed login attempt. This left me with several questions
  - Could these have been accidental? For example, a user tried to access a different VM, but entered my VM’s IP accidentally?
  - Could my VM’s IP just be one in a series of IPs these attackers were trying to gain access to?
  - Did the attacker attempt once then user some sort of proxy or VPN to try to mask their IP?
  - Did the attacker truly only want to try one login attempt?

<h2>Conclusion (After 48 hours)</h2>
This was an exciting project that let me get hands-on experience with Microsoft Sentinel. Training the extraction algorithm to consistently get correct information from the logs and creating the heatmap were the parts I found the coolest during this project. I also learned more about Powershell and about the [ipgeolocation API](https://ipgeolocation.io/). I spent a lot of time reviewing the dashboard I created and thinking of ways to get more out of the data I collected. I highly recommend this project to anyone interested in becoming more familiar with SIEMs or Microsoft Azure.





