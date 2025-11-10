
## **Scenario 1:** "Saskatoon": counting IPs

### Soal
```Markdown
**Scenario:** "Saskatoon": counting IPs.

**Level:** Easy

**Description:** There's a web server access log file at /home/admin/access.log. The file consists of one line per HTTP request, with the requester's IP address at the beginning of each line.  
  
Find what's the IP address that has the most requests in this file (there's no tie; the IP is unique). Write the solution into a file /home/admin/highestip.txt. For example, if your solution is "1.2.3.4", you can do echo "1.2.3.4" > /home/admin/highestip.txt

**Test:** The SHA1 checksum of the IP address sha1sum /home/admin/highestip.txt is 6ef426c40652babc0d081d438b9f353709008e93 (just a way to verify the solution without giving it away.)

**Time to Solve:** 15 minutes.

**OS:** Debian 11

**Root (sudo) Access:** No
```


### Penyelesaian 


Cari IP yang paling banyak mengakses server dengan melihat file access log nya dengan kombinasi command
```bash
cat /home/admin/access.log | grep -Eo '([0-9]{1,3}\.){3}[0-9]{1,3}' | sort | uniq -c
```
Kemudian simpan hasil IP yang tertinggi di file `/home/admin/highestip.txt`


# **Scenario 2:** "Saskatoon": counting IPs

## Soal
```Markdown
**Scenario:** "Saint John": what is writing to this log file?

**Level:** Easy

**Description:** A developer created a testing program that is continuously writing to a log file _/var/log/bad.log_ and filling up disk. You can check for example with tail -f /var/log/bad.log.  
This program is no longer needed. Find it and terminate it. Do not delete the log file.

**Test:** The log file size doesn't change (within a time interval bigger than the rate of change of the log file).  
  
The "Check My Solution" button runs the script _/home/admin/agent/check.sh_, which you can see and execute.

**Time to Solve:** 10 minutes.

**OS:** Debian 11

**Root (sudo) Access:** Yes
```

## Penyesaiaan

 Liat proses yang sedang berjalan, karena proses nulis log masih berlan berkali-kali. Jalakan perintash  `ps aux`  kemudian liat proses yang relate dengan script nulis log

# Scenario 3: 
