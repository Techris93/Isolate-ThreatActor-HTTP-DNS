# Isolate-ThreatActor-HTTP-DNS

### Objectives
In this lab, we will review logs of exploitation of documented HTTP and DNS vulnerabilities.

- Part 1: Investigate an SQL Injection Attack
- Part 2: Investigate DNS Data Exfiltration
  
### Background / Scenario

MySQL is a popular database used by numerous web applications. Unfortunately, SQL injection is a common web hacking technique. It is a code injection technique where an attacker executes malicious SQL statements to control a web application’s database server.

Domain name servers (DNS) are directories of domain names, and they translate the domain names into IP addresses. This service can be used to exfiltrate data.

Cybersecurity personnel have determined that an exploit has occurred, and data containing PII may have been exposed to threat actors. In this lab, we will use Kibana to investigate the exploits to determine the data that was exfiltrated using HTTP and DNS during the attacks.

### Required Resources

- Security Onion virtual machine
  
### Instructions

**Part 1: Investigate an SQL Injection Attack**

In this part, we will investigate an exploit in which unauthorized access was made to sensitive information that is stored on a web server. We will use Kibana to determine the source of the attack and the information accessed by the attacker.

**Step 1: Change the timeframe.**

It has been determined that the exploit happened at sometime during June 2020. Kibana defaults to displaying data for the last 24 hours. We will need to change the time settings to see the data for June 2020.

a. Start the Security Onion VM and login.

b. Enter the sudo `so-status` command to check the status of services.
```
sudo so-status
```

c. After you log in, open Kibana using the shortcut on the Desktop. Login with the username analyst and the password cyberops.


d. In the upper-right corner of the window, click Last 24 hours to change the sample Time Range size. Expand the time range to include interesting alerts. An SQL injection attack took place in June 2020 so that is what you need to target. Select Absolute under Time Range and edit the From and To times to include the entire month of June in 2020. Click Go to continue.

![image](https://i.imgur.com/NxkEBLz.png)

e. Notice the total number of logs for the entire month of June 2020.

![image](https://i.imgur.com/X7F2q1f.png)

**Step 2: Filter for HTTP traffic.**

a. Because the threat actor assessed data that is stored on a web server, the HTTP filter is used to select the logs associated with HTTP traffic. Select HTTP under the Zeek Hunting heading, as shown in the figure.

![image](https://i.imgur.com/ZpCtGKo.png)

**Note:**

- The source IP address is 209.165.200.227.
- The destination IP address is 209.165.200.235.
- The destination port is 80.

b. Scroll down to the HTTP Logs. The results list the first 10 results.

![image](https://i.imgur.com/dVbOUWU.png)

c. Expand the details of the first result by clicking the arrow that is next to the log entry timestamp.

**Note:**

- The timestamp is June 12th 2020, 21:30:09.445.
- The event type is bro_http. Kibana still refers to Zeek using its old name of Bro.
- The message included in the message field username, ccid, ccnumber, ccv, expiration, and password.
- It appears to be a request for credit card information.

**Step 3: Review the results.**

a. Some of the information for the log entries is hyperlinked to other tools. Click the value in the alert _id field of the log entry to get a different view on the event.

![image](https://i.imgur.com/91VO2Ai.png)

b. The result opens in a new web browser tab with information from capME!. capME! tab is a web interface that allows you to view a pcap transcript. The blue text contains HTTP requests that are sent from the source (SRC). The red text is responses from the destination web server (DST).

![image](https://i.imgur.com/0IqfAPd.png)

c. In the Log entry section, which is at the beginning of the transcript, notice the portion username=’+union+select+ccid,ccnumber,ccv, expiration,null+from+credit_cards+–+&password= indicates that someone may have tried to attack the web browser using SQL injection to bypass authentication. The keywords, union and select, are commands that are used in searching for information in a SQL database. If the input boxes on a web page are not properly protected from illegal input, threat actors can inject SQL search strings or other code that can access data contained in databases that are linked to the web page.

d. Find for the keyword username in the transcript. Use Ctrl-F to open a search box. Use the down arrow button in the search box to scroll through the occurrences that were found.

![image](https://i.imgur.com/OF0Xttb.png)


You can see where the term username was used in the web interface that is displayed to the user. However, if you look farther down, something unusual can be found.

- There appears to be a list of usernames and passwords that are part of the information that was returned in response to the HTTP GET request. This is unusual.
- some examples of a username, password, and signature that was exfiltrated are.
```
Username              Password    Signature
4444111122223333      745         2012-03-01
7746536337776330      722         2015-04-01
8242325748474749      461         2016-03-01
7725653200487633      230         2017-06-01
1234567812345678627   627         2018-11-01
```

e. Close the capME! tab and return to Kibana.

**Part 2: Analyze DNS exfiltration.**

A network administrator has noticed abnormally long DNS queries with strange-looking subdomains.

**Step 1: Filter for DNS traffic.**

a. From the top of the Kibana Dashboard, clear any filters and search terms and click Home under the Navigation section of the Dashboard. The Time period should still include June 2020.

b. In the same area of the Dashboard, click DNS in the Zeek Hunting section. Notice the DNS Log Count metrics and Destination Port horizontal bar chart.

![image](https://i.imgur.com/HLe9ySE.png)

Step 2: Review the DNS-related entries.
a. Scroll down the window. You can see the top DNS query types. You may see address records (A record), IPv6 address Quad A records (AAAA), NetBIOS records (NB) and a pointer records for resolving the hostnames (PTR). You can also see the DNS response codes.

b. By Scrolling further down, you can see a list of the top DNS clients and DNS Servers based on their request and response counts. There is also a metric for number of DNS Phishing attempts, which are also known as DNS pharming, spoofing, or poisoning.

![image](https://i.imgur.com/Bdr1zsg.png)

c. Scrolling further down the window you can see a listing of the top DNS queries by domain name. Notice how some of the queries have unusually long subdomains attached to ns.example.com. The domain example.com should be investigated further.

![image](https://i.imgur.com/cHVKF49.png)

d. Scroll back to the top of the window and enter com in the search bar to filter for example.com and click Update. Note that the number of entries in the Log Count is smaller because the display is now limited to requests to the example.com server.

![image](https://i.imgur.com/dgX6BCb.png)

**Note:**

- The DNS client is 192.168.0.11 and the DNS server is 209.165.200.235.

**Step 3: Determine the exfiltrated data.**

a. Continue to scroll further down to see four unique log entries for DNS queries to example.com. Notice how the queries are to suspiciously long subdomains attached to ns.example.com. The long strings of numbers and letters in the subdomains look like text encoded into hexadecimal (0-9, a-f) rather than legitimate subdomain names. Click the Export: Raw download link to download the queries to an external file. A CSV file is downloaded to the /home/analyst/Downloads folder.

![image](https://i.imgur.com/YanMGzh.png)

b. Navigate to the /home/analyst/Downloads Open the file using a text editor, such as gedit. Edit the file by deleting the text surrounding the hexadecimal portion of the subdomains, leaving only the hexadecimal characters. Be sure to remove the quotes too. The contents of your file should look like the information below. Save the edited text file with the original file name.

![image](https://i.imgur.com/oMEEhPO.png)
![image](https://i.imgur.com/uzFFUlD.png)

c. In a terminal, use the xxd command to decode the text in the CSV file and save it to a file named secret.txt. Use cat to output the contents of secret.txt to the console.
```
xxd -r -p "DNS - Queries.csv” > secret.txt
cat secret.txt
```
![image](https://i.imgur.com/k8Lpqmj.png)

Note:
- The subdomains from the DNS queries are not subdomains, they are the text below:
```
CONFIDENTIAL DOCUMENT
DO NOT SHARE
This document contains information about the last security breach.
```

### Conclusion

- The results indicate that the DNS requests were separate, coordinated requests containing hidden content. The larger significance of the result is that DNS queries could be used to hide the sending of files and bypass network security.
- It is possible that malware is creating these requests by cycling through documents on the host and encoding their contents in hexadecimal and then creating DNS queries that use the hexadecimal strings as DNS subdomains. DNS requests are very commonly sent out of a network to the internet, so DNS requests may not be monitored.

### References
https://contenthub.netacad.com/cyberops/
