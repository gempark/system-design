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

Continue ...

[Back to home](../../README.md)