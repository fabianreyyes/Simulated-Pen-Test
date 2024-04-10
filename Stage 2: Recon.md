# Stage 2: Recon

The reconnaissance (recon) stage in penetration testing involves gathering information about the target system or network to identify potential entry points and vulnerabilities. This phase typically consists of passive and active techniques.

## Passive

Involves gathering information without directly interacting with the target system. Techniques include searching public sources such as social media, company websites, search engines, and public databases for information like employee names, email addresses, technology in use, and organizational structure.

--- **Everything beyond this point can be considered a felonly offense if you do not have consent from the victim you are targeting.** ---

## Active 

Entails direct interaction with the target system to collect more detailed information. This can include techniques such as network scanning to identify open ports, services, and operating systems, DNS enumeration to discover subdomains and associated IP addresses, and whois queries to obtain information about domain registration.

### Screen
 To start an engagment it is important to utilized tools such as screen. Screen allows you to create multiple terminal sessions within a single SSH session or local terminal window. This feature is useful for managing multiple tasks simultaneously, such as running different commands, scripts, or tools in separate sessions without the need to open multiple terminal windows. Screen sessions continue running even if the SSH connection is interrupted or terminated. This means that if the connection drops, the pen tester can reconnect to the server and resume their work without losing any progress. Lastly , screen sessions can log all output from terminal sessions to a file, providing an audit trail of commands executed and output generated during the penetration testing process. This logging feature is valuable for documenting findings, troubleshooting issues, and providing evidence to clients or stakeholders.
 
![image](https://github.com/fabianreyyes/Simulated-Pen-Test/blob/main/media/screen-2.gif)

To check if I have any screen sessions open.
```bash
screen -ls
```
To start a screen session.
```bash
screen -S main
```
HOT KEYS USED

ctl+a:c = to create a new screen

ctl+a:spacebar = to switch between screens

### Engagment Directory

Creating a dedicated directory for a penetration testing engagement serves several important purposes. Having a designated directory keeps all relevant files, scripts, tools, and documentation for the engagement in one central location. This makes it easier for the penetration tester to locate and access the necessary resources during the testing process. With all files and data organized in one place, the penetration tester can work more efficiently. They spend less time searching for files or switching between directories, allowing them to focus more on the testing itself. Keeping all engagement-related data confined to a single directory helps maintain data integrity and security. It reduces the risk of accidentally leaving sensitive information scattered across different locations on the system, minimizing the chances of data leakage or unauthorized access.

![image](https://github.com/fabianreyyes/Simulated-Pen-Test/blob/main/media/mkdir_pentest.gif)

I've also included a scope file in my pentest directory, containing the target IP address.

### Scanning the Target

![image](https://github.com/fabianreyyes/Simulated-Pen-Test/blob/main/media/fast_scan.gif)

```bash
sudo nmap -iL scope -F | tee fast_scan
```
sudo: Runs the Nmap command with superuser privileges, which may be necessary to perform certain types of scans or access privileged ports.

nmap: Invokes the Nmap tool, which is a network scanner used for discovering hosts and services on a computer network.

-iL scope: Specifies the input file containing a list of target IP addresses or hostnames. In this case, the file is named "scope".

-F: Initiates a fast scan mode, which scans only the most common ports (100 most common TCP ports).

|: Pipe symbol, which redirects the output of the preceding command to the input of the following command.

tee fast_scan: Writes the output of the Nmap scan both to the terminal (standard output) and to a file named "fast_scan".

### Understanding the Scan

Observing that port 22, 53, 80, 445, 5432, and 10000 are accessible on the target IP, determining which port to prioritize for further investigation or exploitation depends on various factors, including objectives, the services running on those ports, and the potential vulnerabilities associated with them. Among these ports, port 80 emerges as the most straightforward for conducting further reconnaissance. Typically, port 80 is linked with web servers, offering a promising avenue for exploration.

### Interacting With the Target

![image](https://github.com/fabianreyyes/Simulated-Pen-Test/blob/main/media/http_firefox.gif)

To engage with port 80, I entered the target IP into the browser, revealing a web server with a cat picture. As I explored the site, it became evident that it consisted solely of the image. Intrigued, I went into the source code yet found nothing of interest. However, upon further investigation into the server's script, I discovered it was operating on Apache/2.4.52 (Debian), potentially unveiling a promising avenue for exploitation.

Apache/2.4.52 CVE's: https://www.cvedetails.com/vulnerability-list/vendor_id-45/product_id-66/version_id-782030/Apache-Http-Server-2.4.52.html

Following a thorough review of the vulnerability list, we reached the conclusion that there were no easily obtainable CVE entries available to facilitate our quest for Remote Code Execution (RCE) on the target system.

Apache 2.4.52 is a specific version of the Apache web server software. By revealing the exact version number, an attacker gains insight into the software stack used by the target organization. Each software version may have its own set of vulnerabilities and security weaknesses. Attackers can use the disclosed version number to search public vulnerability databases or exploit repositories for known vulnerabilities. Armed with knowledge of the web server version, attackers can tailor their attacks to specifically target vulnerabilities. We would consider the discovery of the web server running apache a low level data leak because it is hard to gain RCE through known vulnerabilities of that apache version.

Remediation: Security Through Obscurity- Organizations often rely on security through obscurity by concealing specific details about their software and infrastructure. The disclosure of the Apache version number undermines this security practice, providing attackers with critical information about the organization's technology stack.

### Finding Hidden Endpoints

The dirb command is a tool used for web application directory and file enumeration. It is often employed in penetration testing or security assessments to discover hidden or unlinked resources on a web server. When you run dirb with a specified target URL or IP address, it recursively scans the web server directory structure, attempting to find files and directories that are accessible. It does this by trying various common directory and file names, typically based on a predefined wordlist or dictionary, and then checking the server response for each request. This process helps in identifying potential entry points or vulnerabilities within the web application.

```bash
dirb http://192.168.122.47
```

![image](https://github.com/fabianreyyes/Simulated-Pen-Test/blob/main/media/dirb_output.gif)

We interacted with several hidden endpoints during our exploration. Upon visiting these endpoints, we encountered a lack of clickable elements or revealing source code, just more images awaited us. After extensive searching, our team discovered an admin portal equipped to read files, provided one possesses appropriate credentials. We tried logging in using default credentials, which unfortunately granted us access. Upon further investigation, we uncovered a critical flaw: the broken web application. We found no user credentials were necessary to access files within the web application. The View.php web app was designed to give administrators file access, but the AdminName and AdminID credential fields remain inactive within the code. This lack of code constitutes a Broken Authentication vulnerability, affording us unauthorized access to sensitive files on the target system without requiring authentication. 

![image](https://github.com/fabianreyyes/Simulated-Pen-Test/blob/main/media/ez_admin.gif)

Remediation: Utilize AdminName and AdminID in view.php code alongside if statement that allows filename interaction if correct credentials are provided.

Our group was able to read all the sensitive files, but nothing of value was discovered to gain RCE.

### Command Injection

Using the low-level data leak we found earlier, we know that the web server uses Linux in the background. When we type a file name in the file name field, it runs a command in the background in a specific directory on the Linux machine, which gives us a "cat" command response to read the file. In Linux, we can use characters such as (; & |) to run a follow-on command after the first command runs. In this case we can use those characters to execute command injection on the webserver. 

![image](https://github.com/fabianreyyes/Simulated-Pen-Test/blob/main/media/cmdi.gif)

This vulnerability is critical and must be patched immediately because an attacker can quickly gain RCE through this broken web application through command injection.
