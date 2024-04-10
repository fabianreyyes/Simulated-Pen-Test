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

To check if i have any screen sessions open
```bash
screen -ls
```
To start a screen session
```bash
screen -S main
```
