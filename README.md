# Kioptrix-2014-Level5-walk-Through

note : before starting the machine remove and add the network card and choose  the network to be bridged 

# Enumeration 

Nmap -sc -sV 192.168.1.0/24 --script vuln 

shows that there are only 2 open ports 80 and 8080 

# web server on Port 80 

shows only this !! 

![image](https://user-images.githubusercontent.com/52453415/127748838-356b42b9-5381-4177-8c7d-2b9474a86afe.png)

and on port 8080 forbids every thing  every thing 

so i tried to brute force the directories on port 80 through wfuzz using large wordlist but it didn't work 

seems like this server has nothing to show us 

so what could be useful lead in this situation !!! , i thought that directory traversal might be 

# LFI 

i tired to fuzz for some parameters from this list and using a good wordlist nothing appeared too !!

![image](https://user-images.githubusercontent.com/52453415/127749025-2087b6d5-28a0-457e-9c11-f6240af990d8.png)

so i went back to webserver and viewed the source code of the page hoping find any thing useful , and i was lucky 

![image](https://user-images.githubusercontent.com/52453415/127749086-4386be30-8cdb-48f0-8720-66aee9861f76.png)

i went to this directory but nothing is useful there , just a bunch of charts 

so i searched for vulnerabilities for this Pchart 

![image](https://user-images.githubusercontent.com/52453415/127749152-28291234-d13c-4ba3-af07-cbb2d4c3b7c7.png)

![image](https://user-images.githubusercontent.com/52453415/127749189-da1c54db-6a72-4de6-b439-8807eb4cec6d.png)

the parameter was action then :D , any way used the payload shown above and the passwd showed up , but since the ssh is closed and there is no remote connection this credential is worthless 


![image](https://user-images.githubusercontent.com/52453415/127749220-73c79205-5c59-4e71-8247-bfc21eb72a43.png)

i searched metasploit for vulnerabilities for freebsd directory traversal ,I've found a module but for reasons i don't know it didn't exploit


![image](https://user-images.githubusercontent.com/52453415/127749465-ec8071de-5eec-4650-94fb-46c7cc58e292.png)

any way this cve is in 2019 and the machine was deployed in 2014 so i wasn't suppose to use this cve any way :D 


now i need another sensitive files to read , so i searched for possible files directory for target os 

![image](https://user-images.githubusercontent.com/52453415/127749326-805a3664-1d5c-4e2e-ab90-1ddc08df3bbf.png)

i found the configuration file for the appache server 

![image](https://user-images.githubusercontent.com/52453415/127749530-94a60fd1-0cd9-41c8-b221-ac7cfddc7631.png)

sounds like this is why 8080 forbids every thing 

![image](https://user-images.githubusercontent.com/52453415/127749542-041d3af9-d407-4c26-b99c-a2e5c36f506d.png)

we need to create this environment , so i used burpsuite to intercept and modify the requests to :8080

![image](https://user-images.githubusercontent.com/52453415/127749575-4d50323a-61ac-4531-898d-8ed1b4007a08.png)

![image](https://user-images.githubusercontent.com/52453415/127749580-490434d1-a74b-42ae-b253-155b1e18c9fb.png)

![image](https://user-images.githubusercontent.com/52453415/127749617-7c8dd671-6673-42e3-bb13-c735e14dd49e.png)

sounds like the server uses this function to edit pdf files , so i searched online i found that this function is vulnerable  to command injection in here 

https://www.exploit-db.com/exploits/21665 

![image](https://user-images.githubusercontent.com/52453415/127749684-e9e60da6-59df-47ab-9643-30006d38c101.png)

but i didn't work since it's different version of the function 

so i searched MSF for any exploits , i found this module 

![image](https://user-images.githubusercontent.com/52453415/127749730-287ac80c-d250-477e-8c56-0ae53b9fc32d.png)

exploit and here we go , we got shell , a limited one 

![image](https://user-images.githubusercontent.com/52453415/127749770-7b1516fe-bf9c-4b0a-a943-61345c5fc2c7.png)

# privilege escalation 

search for exploit of the OS freeBSD that can escalate my privilege 

i found metasploit module , but requires presence of meterpreter session which i couldn't provide due to segmentation errors , any way

i searched exploit -db i found useful exploit , download it and get it transferred to target via netcat , since the target has gcc compiler 

![image](https://user-images.githubusercontent.com/52453415/127749864-19d84a0e-26f0-42d9-b534-26d4904e18a8.png)

compile it gcc exploit.c -o exploit , make it excutable and run 

![image](https://user-images.githubusercontent.com/52453415/127749882-30536d32-f9d1-447e-85ab-ef21eafdaf1d.png)

and here we got root 

Open the flag cat /root/congrats.txt , and enjjoy the auther advices 

![image](https://user-images.githubusercontent.com/52453415/127749900-e7a939a0-6dde-49cf-bd76-871cbb35dfc5.png)


# thanks for reading this , i hope it was useful

