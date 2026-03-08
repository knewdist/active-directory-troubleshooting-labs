# Lab 05 – DC Locator, SRV Records, and DNS Misconfiguration

## Overview

This lab demonstrates how **Active Directory clients discover Domain Controllers** using DNS SRV records and how domain connectivity breaks when clients use an **external DNS server instead of the Active Directory DNS server**.

The lab also explores how the **DC Locator process works** and how DNS misconfiguration affects domain authentication.

---

# Lab Environment

| System | Role |
|------|------|
| DC01 | Domain Controller + DNS |
| DC02 | Domain Controller + DNS |
| FS01 | Tier 1 Server |
| CL01 | Domain Client |
| PAWs | Privileged Access Workstation |

Domain: corp.local


---

# Step 1 – Verify SRV Records in Active Directory DNS

Domain Controllers automatically register service records in DNS when the **Netlogon service starts**.

These records allow domain clients to locate authentication services.

Open **DNS Manager** and navigate to:

    _msdcs.corp.local
    └── dc
    └── _tcp

You should see LDAP SRV records pointing to the domain controllers.

![DNS SRV Records](../screenshots/lab05_dns_srv_records.png)

These records tell clients: Which servers provide LDAP directory services
                            Which servers provide Kerberos authentication


---

# Step 2 – Query SRV Records from the Client

On **CL01**, query the SRV records used by the DC Locator process.

nslookup
set type=SRV
_ldap._tcp.dc._msdcs.corp.local


Example output:

![SRV Query](../screenshots/lab05_srv_query.png)

Results show both domain controllers: dc01.corp.local
                                      dc02.corp.local
These SRV records are what clients use to locate domain controllers.

---

# Step 3 – Run DC Locator

Run the DC Locator command:

```powershell
nltest /dsgetdc:corp.local

Example output:

Important fields:

Field	        Meaning
DC   	        Domain Controller selected
Address	IP      address of the DC
Dom Name	    Active Directory domain
Forest Name	    Forest root domain
Flags	        Services provided by the DC

Example:

DC: \\DC02.corp.local
Address: \\192.168.1.12

This shows the client selected DC02 as the domain controller.

___

# Step 4 – Verify Client DNS Configuration

Check which DNS servers the client is using:

ipconfig /all

Example:

The client should be using Active Directory DNS servers:

192.168.1.12
192.168.1.11

These correspond to:DC02
                    DC01

___

# Step 5 – Break DNS Resolution

Now simulate a common misconfiguration by changing the client DNS server to a public DNS server.

Set-DnsClientServerAddress -ServerAddresses ("8.8.8.8")

This simulates a workstation incorrectly configured to use Google DNS.

___

# Step 6 – Force DC Discovery

Run the DC Locator command again, forcing a new lookup:

nltest /dsgetdc:corp.local /force

Result:

Output: Getting DC name failed: Status = 1355
ERROR_NO_SUCH_DOMAIN

This occurs because the client is now querying Google DNS, which does not host internal Active Directory zones.

___

# Step 7 – Verify SRV Lookup Failure

Run the SRV lookup again:

nslookup
set type=SRV
_ldap._tcp.dc._msdcs.corp.local

Result:

Output:dns.google can't find _ldap._tcp.dc._msdcs.corp.local: Non-existent domain

This confirms that external DNS servers do not contain Active Directory service records.

# Root Cause
Active Directory depends heavily on DNS SRV records.
Domain clients locate authentication services by querying: _ldap._tcp.dc._msdcs.corp.local

Public DNS servers such as: 8.8.8.8

**do not contain these records.**

# Resolution

Configure domain clients to use Active Directory DNS servers.

Example:192.168.1.11 (DC01)
            192.168.1.12 (DC02)

After correcting DNS: ipconfig /flushdns
                      nltest /dsgetdc:corp.local

Domain controller discovery should succeed again.

# Key Concepts Learned

This lab demonstrates:
Active Directory DNS dependency
SRV record usage
DC Locator process
DNS misconfiguration troubleshooting
Domain controller discovery

# Useful Commands
ipconfig /all
ipconfig /flushdns
nltest /dsgetdc:corp.local
nltest /dsgetdc:corp.local /force
nslookup
set type=SRV
_ldap._tcp.dc._msdcs.corp.local
Set-DnsClientServerAddress -ServerAddresses ("8.8.8.8")
