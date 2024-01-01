[Back to home](../../README.md)

## Verify UFW Status

```bash
sudo ufw status
Status: inactive # output
```

The output will indicate if your firewall is active or not.

## Enable UFW

```bash
sudo ufw enable
```
```
Firewall is active and enabled on system startup
```

To see what is currently blocked or allowed, you may use the verbose parameter when running ufw status, as follows:

```bash
sudo ufw status
```
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip
```

## Disable UFW

```bash
sudo ufw disable
```

## Block an IP Address

```bash
sudo ufw deny from 203.0.113.100
```
```
Rule added
```

Check again

```bash
sudo ufw status
```
```
Status: active

To                         Action      From
--                         ------      ----
Anywhere                   DENY        203.0.113.100   
```

## Block a Subnet

```bash
sudo ufw deny from 203.0.113.0/24
```

## Allow an IP Address

```bash
sudo ufw allow from 203.0.113.101
```

## Delete UFW Rule

List rules

```bash
sudo ufw status numbered
```
```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] Anywhere                   DENY IN     203.0.113.100             
[ 2] Anywhere on eth0           ALLOW IN    203.0.113.102    
```

Delete UFW rule number 1

```bash
sudo ufw delete 1
```

## List Available Application Profiles

```bash
sudo ufw app list
```
```
Available applications:
  OpenSSH
```

## Enable Application Profile

```bash
sudo ufw allow "OpenSSH"
```
```
Rule added
Rule added (v6)
```

## Disable Application Profile

List all profiles

```bash
sudo ufw status
```
```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                               
Nginx Full                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)                   
Nginx Full (v6)            ALLOW       Anywhere (v6)  
```

```bash
sudo ufw allow "Nginx HTTPS"
sudo ufw delete allow "Nginx Full"
```

## Allow MySQL Connection from Specific IP Address or Subnet

```bash
sudo ufw allow from 203.0.113.103 to any port 3306
```

```bash
sudo ufw allow from 203.0.113.0/24 to any port 3306
```

## Allow PostgreSQL Connection from Specific IP Address or Subnet

```bash
sudo ufw allow from 203.0.113.103 to any port 5432
```

```bash
sudo ufw allow from 203.0.113.0/24 to any port 5432
```

[Back to home](../../README.md)