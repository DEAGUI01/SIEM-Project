<h1>Azure SIEM Project</h1>


<h2>Description</h2>
In this project I setup Microsoft Sentinel (SIEM) and connect it to a honeypot VM that sends custom logs which include geolocation extracted from attacker's IP addresses to a Log Analytics Workspace from which the SIEM pulls data. Specifically, I observe and analyze RDP attacks from all over the world and draw some key takeaways.
<br />


<h2>Languages and Utilities Used</h2>

- <b>Microsoft Sentinel</b> 
- <b>Log Analytics Workspace</b>
- <b>Powershell</b>
- <b>KQL (Kusto Query Language)</b>

<h2>Environments Used </h2>

- <b>Microsoft Azure
- <b>Windows 10 Pro</b> 

<h2>Setup Diagram </h2>
<img src="https://i.imgur.com/24H8QnW.png" height="80%" width="80%" alt="diagram"/>
<br />
<br />

<h2>Project walk-through:</h2>

<p align="center">

On Azure, create Azure Virtual Machine <br/>
<img src="https://i.imgur.com/23jq2Ql.png" height="80%" width="80%" alt="create VM"/>
<br />
<br />
Create network security group for VM  <br/>
<img src="https://i.imgur.com/1V1U6FU.png" height="80%" width="80%" alt="network security group"/>
<br />
<br />
Create new inbound rule to make the virtual machine vulnerable (Allow traffic from all ports and into all ports using any protocols)<br/>
<img src="https://i.imgur.com/3b7sD2r.png" height="80%" width="80%" alt="inbound rule"/>
<br />
<br />
Create a LAW (Log Analytics Workspace) (This is where our custom logs will be sent and where we'll be able to see and analyze the data)  <br/>
<img src="https://i.imgur.com/2AFA5Zj.png" height="80%" width="80%" alt="create LAW"/>
<br />
<br />
Under Microsoft Defender for cloud, go to Environment Settings and turn on Foundational CSPM and Servers on your workspace, keep SQL servers off since we wont be using any<br/>
<img src="https://i.imgur.com/OyRE9TW.png" height="80%" width="80%" alt="change settings"/>
<br />
<br />
Under Data Collection, select All Events <br/>
<img src="https://i.imgur.com/0vOmL6h.png" height="80%" width="80%" alt="change settings"/>
<br />
<br />
Go back to the LAW, select the VM, and connect it to the workspace <br/>
<img src="https://i.imgur.com/3MKwukT.png" height="80%" width="80%" alt="connect VM"/>
<br />
<br />
Set up Microsoft Sentinel for the LAW <br/>
<img src="https://i.imgur.com/vLVGJJs.png" height="80%" width="80%" alt="setup Microsoft Sentinel"/>
<br />
<br />
On the VM, turn Windows Firewall completely off for both domain and public profiles <br/>
<img src="https://i.imgur.com/fxWZh3k.png" height="80%" width="80%" alt="turn firewall off"/>
<br />
<br />
Check that the VM can now be pinged externally <br/>
<img src="https://i.imgur.com/70PRVx7.png" height="80%" width="80%" alt="install DHCP"/>
<br />
<br />
Create powershell script that will take the data from windows logs (event viewer), send the IP address to an external geolocator API to get the geolocation of attackers and put this information on a custom log file to be sent to the LAW. We specify some sample data in order to train the LAW.<br/>
<img src="https://i.imgur.com/IppJnSJ.png" height="80%" width="80%" alt="powershell script"/>
<br />
<br />
Verify that the script runs and that it creates the custom log file in the specified location with the specified sample data <br/>
<img src="https://i.imgur.com/nH7Koag.png" height="80%" width="80%" alt="verification"/>
<br />
<br />
Check that the script works as intented and detects a failed log on attempt by trying to RDP into the VM with the wrong username or password <br/>
<img src="https://i.imgur.com/j05tVcZ.png" height="80%" width="80%" alt="test"/>
<br />
<br />
Back on the LAW, create a custom log using the file created by the powershell script <br/>
<img src="https://i.imgur.com/GNtbObN.png" height="80%" width="80%" alt="create custom log"/>
<br />
<br />
Review and create custom log <br/>
<img src="https://i.imgur.com/KqV5UWJ.png" height="80%" width="80%" alt="review custom log"/>
<br />
<br />
Run custom log query and check that raw data is pulled from the custom log file <br/>
<img src="https://i.imgur.com/lnBmHGH.png" height="80%" width="80%" alt="run custom log query"/>
<br />
<br />
Write and run a KQL query to turn the raw data into usable fields like SourceHost, Country, Latitude, Longitude, etc.<br/>
<img src="https://i.imgur.com/cHSiHC7.png" height="80%" width="80%" alt="KQL query for custom log"/>
<br />
<br />
On Microsoft Sentinel, add a new workbook (We will use this to create a map to better visualize the data coming into the LAW) <br/>
<img src="https://i.imgur.com/v4C2rnd.png" height="80%" width="80%" alt="create Workbook"/>
<br />
<br />
Write a KQL query to pull log data into workbook <br/>
<img src="https://i.imgur.com/ROU3bEb.png" height="80%" width="80%" alt="KQL query for workbook"/>
<br />
<br />
Change map settings to visualize country, IP address, and event count on the map <br/>
<img src="https://i.imgur.com/GPuEIRM.png" height="80%" width="80%" alt="change map settings"/>
<br />
<br />
Wait for attackers to discover your virtual machine and attack it <br/>
<img src="https://i.imgur.com/FOCAq5d.png" height="80%" width="80%" alt="wait for attackers"/>
<br />
<br />
After roughly 48 hours, we can see attackers from all over the world have tried to brute force their way into our VM through RDP<br/>
<img src="https://i.imgur.com/DMLoszV.png" height="80%" width="80%" alt="wait for attackers"/>
<br />
<br />
Back on the LAW, analyze some of the attempts, we can see all the usernames attackers have tried, see that administrator and admin seem to be some of the most common guessed usernames<br/>
<img src="https://i.imgur.com/gOp56l6.png" height="80%" width="80%" alt="analyze attempts"/>
<br />
<br />

<h2>Key Takeaways </h2>
- It doesn't matter what the device is or whether it holds high value or confidential data, as soon as it becomes routable through the internet (Public IP address) and vulnerable through a lack of security such as a firewall, it will become the target of thousands of attacks in a matter of hours, because there are millions of bots out there looking for new devices to join their botnet. So it becomes a necessity to keep our devices secure even if we believe we have nothing of value on them.
<br />
- By analyzing the usernames used on many of the attacks, we realize the importance of having complex and long passwords/usernames. Most of the usernames tried by attackers were common dictionary attack usernames such as: User, admin, terminal, backups, etc. These are all easily guessable usernames that will most likely be found on a dictionary brute force attack.
<br />
- Access to remote access protocols like RDP should be restricted to the internet by the use of private IP addresses or NAT, or disabled all together. An internal host should not be routable from the internet.
</p>
