#  LDAP. Централизованная авторизация и аутентификация.

Задание:

1. Установить FreeIPA;
2. Написать Ansible playbook для конфигурации клиента;
3 *. Настроить аутентификацию по SSH-ключам;
4 **. Firewall должен быть включен на сервере и на клиенте.

Выполнение:

В реализованных ролях "проблема" с идемпотентностью, так при повторном деплое необходимо применять роль передеплоя (ее-то и нужно поправить).
```shell
cd ../../
cd ./034/vm/
vagrant destroy -f
vagrant up
python3 v2a.py -o ../ansible/inventories/hosts # Это уже как кредо
cd ../../
cd ./034/ansible/
```

Фаервол включен все время.

### Настройка сервера

```shell
ansible-playbook playbooks/server.yml --tags deploy > ../files/server.yml.txt
```


<details><summary>см. лог `playbooks/server.yml --tags deploy`</summary>

```text

PLAY [Playbook of server initialization] ***************************************

TASK [Gathering Facts] *********************************************************
ok: [server]

TASK [../roles/server : Check firewall-cmd state] ******************************
changed: [server]

TASK [../roles/server : Store to file firewall-cmd state] **********************
changed: [server -> localhost]

TASK [../roles/server : Install python-pip for pexpect promt answering] ********
changed: [server]

TASK [../roles/server : Pip install pexpect] ***********************************
changed: [server]

TASK [../roles/server : Set time zone] *****************************************
changed: [server]

TASK [../roles/server : Synchronize datetime | Install chrony] *****************
ok: [server]

TASK [../roles/server : Synchronize datetime | Turn on chronyd] ****************
changed: [server]

TASK [../roles/server : Set hostname] ******************************************
changed: [server]

TASK [../roles/server : Set hostname | In order to apply the new hostname] *****
changed: [server]

TASK [../roles/server : Configure iptables] ************************************
changed: [server]

TASK [../roles/server : /etc/sysctl.d/99-sysctl.conf | get content] ************
changed: [server]

TASK [../roles/server : /etc/sysctl.d/99-sysctl.conf | set net.ipv6.conf.lo.disable_ipv6=0 | if did not set] ***
changed: [server]

TASK [../roles/server : /etc/sysctl.d/99-sysctl.conf | set net.ipv6.conf.lo.disable_ipv6=0 | if 1] ***
ok: [server]

TASK [../roles/server : Unconfigure ipv6 at grub] ******************************
changed: [server]

TASK [../roles/server : Reboot VM] *********************************************
changed: [server]

TASK [../roles/server : Install ipa-server] ************************************
changed: [server]

TASK [../roles/server : Install ipa-server-dns] ********************************
changed: [server]

TASK [../roles/server : Configure ipa-server] **********************************
changed: [server]

TASK [../roles/server : Save ipa_server_configure_log] *************************
changed: [server -> localhost]

TASK [../roles/server : Check kinit admin] *************************************
changed: [server]

TASK [../roles/server : Store kinit admin log to file] *************************
changed: [server -> localhost]

TASK [../roles/server : Check klist] *******************************************
changed: [server]

TASK [../roles/server : Store klist log to file] *******************************
changed: [server -> localhost]

RUNNING HANDLER [../roles/server : systemctl daemon reload] ********************
ok: [server]

RUNNING HANDLER [../roles/server : Pip uninstall pexpect] **********************
changed: [server]

RUNNING HANDLER [../roles/server : Uninstall python-pip] ***********************
changed: [server]

PLAY RECAP *********************************************************************
server                     : ok=27   changed=23   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```

</details>


<details><summary>см. server-лог `firewall_cmd --state`</summary>

```text
running
```

</details>


<details><summary>см. процесс автоматизированного конфигурирования IPA-сервера посредством Ansible</summary>

```text
[?1034h
The log file for this installation can be found in /var/log/ipaserver-install.log
==============================================================================
This program will set up the IPA Server.

This includes:
  * Configure a stand-alone CA (dogtag) for certificate management
  * Configure the Network Time Daemon (ntpd)
  * Create and configure an instance of Directory Server
  * Create and configure a Kerberos Key Distribution Center (KDC)
  * Configure Apache (httpd)
  * Configure the KDC to enable PKINIT

To accept the default shown in brackets, press the Enter key.

WARNING: conflicting time&date synchronization service 'chronyd' will be disabled
in favor of ntpd

Do you want to configure integrated DNS (BIND)? [no]: 
Enter the fully qualified domain name of the computer
on which you're setting up server software. Using the form
<hostname>.<domainname>
Example: master.example.com.


Server host name [server.otus.local]: 
Warning: skipping DNS resolution of host server.otus.local
The domain name has been determined based on the host name.

Please confirm the domain name [otus.local]: 
The kerberos protocol requires a Realm name to be defined.
This is typically the domain name converted to uppercase.

Please provide a realm name [OTUS.LOCAL]: Certain directory server operations require an administrative user.
This user is referred to as the Directory Manager and has full access
to the Directory for system management tasks and will be added to the
instance of directory server created for IPA.
The password must be at least 8 characters long.

Directory Manager password: 
Password (confirm): 

The IPA server requires an administrative user, named 'admin'.
This user is a regular system account used for IPA server administration.

IPA admin password: 
Password (confirm): 

Checking DNS domain otus.local., please wait ...
Do you want to configure DNS forwarders? [yes]: Following DNS servers are configured in /etc/resolv.conf: 10.0.2.3
Do you want to configure these servers as DNS forwarders? [yes]: All DNS servers from /etc/resolv.conf were added. You can enter additional addresses now:
Enter an IP address for a DNS forwarder, or press Enter to skip: DNS forwarder 77.88.8.8 added. You may add another.
Enter an IP address for a DNS forwarder, or press Enter to skip: Checking DNS forwarders, please wait ...
Do you want to search for missing reverse zones? [yes]: Do you want to create reverse zone for IP 192.168.34.10 [yes]: Please specify the reverse zone name [34.168.192.in-addr.arpa.]: Do you want to create reverse zone for IP 10.0.2.15 [yes]: Please specify the reverse zone name [2.0.10.in-addr.arpa.]: Using reverse zone(s) 34.168.192.in-addr.arpa., 2.0.10.in-addr.arpa.

The IPA Master Server will be configured with:
Hostname:       server.otus.local
IP address(es): 192.168.34.10, 10.0.2.15
Domain name:    otus.local
Realm name:     OTUS.LOCAL

BIND DNS server will be configured to serve IPA domain with:
Forwarders:       10.0.2.3, 77.88.8.8
Forward policy:   only
Reverse zone(s):  34.168.192.in-addr.arpa., 2.0.10.in-addr.arpa.

Continue to configure the system with these values? [no]: 
The following operations may take some minutes to complete.
Please wait until the prompt is returned.

Adding [192.168.34.10 server.otus.local] to your /etc/hosts file
Adding [10.0.2.15 server.otus.local] to your /etc/hosts file
Configuring NTP daemon (ntpd)
  [1/4]: stopping ntpd
  [2/4]: writing configuration
  [3/4]: configuring ntpd to start on boot
  [4/4]: starting ntpd
Done configuring NTP daemon (ntpd).
Configuring directory server (dirsrv). Estimated time: 30 seconds
  [1/45]: creating directory server instance
  [2/45]: enabling ldapi
  [3/45]: configure autobind for root
  [4/45]: stopping directory server
  [5/45]: updating configuration in dse.ldif
  [6/45]: starting directory server
  [7/45]: adding default schema
  [8/45]: enabling memberof plugin
  [9/45]: enabling winsync plugin
  [10/45]: configure password logging
  [11/45]: configuring replication version plugin
  [12/45]: enabling IPA enrollment plugin
  [13/45]: configuring uniqueness plugin
  [14/45]: configuring uuid plugin
  [15/45]: configuring modrdn plugin
  [16/45]: configuring DNS plugin
  [17/45]: enabling entryUSN plugin
  [18/45]: configuring lockout plugin
  [19/45]: configuring topology plugin
  [20/45]: creating indices
  [21/45]: enabling referential integrity plugin
  [22/45]: configuring certmap.conf
  [23/45]: configure new location for managed entries
  [24/45]: configure dirsrv ccache
  [25/45]: enabling SASL mapping fallback
  [26/45]: restarting directory server
  [27/45]: adding sasl mappings to the directory
  [28/45]: adding default layout
  [29/45]: adding delegation layout
  [30/45]: creating container for managed entries
  [31/45]: configuring user private groups
  [32/45]: configuring netgroups from hostgroups
  [33/45]: creating default Sudo bind user
  [34/45]: creating default Auto Member layout
  [35/45]: adding range check plugin
  [36/45]: creating default HBAC rule allow_all
  [37/45]: adding entries for topology management
  [38/45]: initializing group membership
  [39/45]: adding master entry
  [40/45]: initializing domain level
  [41/45]: configuring Posix uid/gid generation
  [42/45]: adding replication acis
  [43/45]: activating sidgen plugin
  [44/45]: activating extdom plugin
  [45/45]: configuring directory to start on boot
Done configuring directory server (dirsrv).
Configuring Kerberos KDC (krb5kdc)
  [1/10]: adding kerberos container to the directory
  [2/10]: configuring KDC
  [3/10]: initialize kerberos container
  [4/10]: adding default ACIs
  [5/10]: creating a keytab for the directory
  [6/10]: creating a keytab for the machine
  [7/10]: adding the password extension to the directory
  [8/10]: creating anonymous principal
  [9/10]: starting the KDC
  [10/10]: configuring KDC to start on boot
Done configuring Kerberos KDC (krb5kdc).
Configuring kadmin
  [1/2]: starting kadmin 
  [2/2]: configuring kadmin to start on boot
Done configuring kadmin.
Configuring ipa-custodia
  [1/5]: Making sure custodia container exists
  [2/5]: Generating ipa-custodia config file
  [3/5]: Generating ipa-custodia keys
  [4/5]: starting ipa-custodia 
  [5/5]: configuring ipa-custodia to start on boot
Done configuring ipa-custodia.
Configuring certificate server (pki-tomcatd). Estimated time: 3 minutes
  [1/30]: configuring certificate server instance
  [2/30]: secure AJP connector
  [3/30]: reindex attributes
  [4/30]: exporting Dogtag certificate store pin
  [5/30]: stopping certificate server instance to update CS.cfg
  [6/30]: backing up CS.cfg
  [7/30]: disabling nonces
  [8/30]: set up CRL publishing
  [9/30]: enable PKIX certificate path discovery and validation
  [10/30]: starting certificate server instance
  [11/30]: configure certmonger for renewals
  [12/30]: requesting RA certificate from CA
  [13/30]: setting audit signing renewal to 2 years
  [14/30]: restarting certificate server
  [15/30]: publishing the CA certificate
  [16/30]: adding RA agent as a trusted user
  [17/30]: authorizing RA to modify profiles
  [18/30]: authorizing RA to manage lightweight CAs
  [19/30]: Ensure lightweight CAs container exists
  [20/30]: configure certificate renewals
  [21/30]: configure Server-Cert certificate renewal
  [22/30]: Configure HTTP to proxy connections
  [23/30]: restarting certificate server
  [24/30]: updating IPA configuration
  [25/30]: enabling CA instance
  [26/30]: migrating certificate profiles to LDAP
  [27/30]: importing IPA certificate profiles
  [28/30]: adding default CA ACL
  [29/30]: adding 'ipa' CA entry
  [30/30]: configuring certmonger renewal for lightweight CAs
Done configuring certificate server (pki-tomcatd).
Configuring directory server (dirsrv)
  [1/3]: configuring TLS for DS instance
  [2/3]: adding CA certificate entry
  [3/3]: restarting directory server
Done configuring directory server (dirsrv).
Configuring ipa-otpd
  [1/2]: starting ipa-otpd 
  [2/2]: configuring ipa-otpd to start on boot
Done configuring ipa-otpd.
Configuring the web interface (httpd)
  [1/22]: stopping httpd
  [2/22]: setting mod_nss port to 443
  [3/22]: setting mod_nss cipher suite
  [4/22]: setting mod_nss protocol list to TLSv1.2
  [5/22]: setting mod_nss password file
  [6/22]: enabling mod_nss renegotiate
  [7/22]: disabling mod_nss OCSP
  [8/22]: adding URL rewriting rules
  [9/22]: configuring httpd
  [10/22]: setting up httpd keytab
  [11/22]: configuring Gssproxy
  [12/22]: setting up ssl
  [13/22]: configure certmonger for renewals
  [14/22]: importing CA certificates from LDAP
  [15/22]: publish CA cert
  [16/22]: clean up any existing httpd ccaches
  [17/22]: configuring SELinux for httpd
  [18/22]: create KDC proxy config
  [19/22]: enable KDC proxy
  [20/22]: starting httpd
  [21/22]: configuring httpd to start on boot
  [22/22]: enabling oddjobd
Done configuring the web interface (httpd).
Configuring Kerberos KDC (krb5kdc)
  [1/1]: installing X509 Certificate for PKINIT
Done configuring Kerberos KDC (krb5kdc).
Applying LDAP updates
Upgrading IPA:. Estimated time: 1 minute 30 seconds
  [1/10]: stopping directory server
  [2/10]: saving configuration
  [3/10]: disabling listeners
  [4/10]: enabling DS global lock
  [5/10]: disabling Schema Compat
  [6/10]: starting directory server
  [7/10]: upgrading server
  [8/10]: stopping directory server
  [9/10]: restoring configuration
  [10/10]: starting directory server
Done.
Restarting the KDC
Configuring DNS (named)
  [1/12]: generating rndc key file
  [2/12]: adding DNS container
  [3/12]: setting up our zone
  [4/12]: setting up reverse zone
  [5/12]: setting up our own record
  [6/12]: setting up records for other masters
  [7/12]: adding NS record to the zones
  [8/12]: setting up kerberos principal
  [9/12]: setting up named.conf
  [10/12]: setting up server configuration
  [11/12]: configuring named to start on boot
  [12/12]: changing resolv.conf to point to ourselves
Done configuring DNS (named).
Restarting the web server to pick up resolv.conf changes
Configuring DNS key synchronization service (ipa-dnskeysyncd)
  [1/7]: checking status
  [2/7]: setting up bind-dyndb-ldap working directory
  [3/7]: setting up kerberos principal
  [4/7]: setting up SoftHSM
  [5/7]: adding DNSSEC containers
  [6/7]: creating replica keys
  [7/7]: configuring ipa-dnskeysyncd to start on boot
Done configuring DNS key synchronization service (ipa-dnskeysyncd).
Restarting ipa-dnskeysyncd
Restarting named
Updating DNS system records
Configuring client side components
Using existing certificate '/etc/ipa/ca.crt'.
Client hostname: server.otus.local
Realm: OTUS.LOCAL
DNS Domain: otus.local
IPA Server: server.otus.local
BaseDN: dc=otus,dc=local

Skipping synchronizing time with NTP server.
New SSSD config will be created
Configured sudoers in /etc/nsswitch.conf
Configured /etc/sssd/sssd.conf
trying https://server.otus.local/ipa/json
[try 1]: Forwarding 'schema' to json server 'https://server.otus.local/ipa/json'
trying https://server.otus.local/ipa/session/json
[try 1]: Forwarding 'ping' to json server 'https://server.otus.local/ipa/session/json'
[try 1]: Forwarding 'ca_is_enabled' to json server 'https://server.otus.local/ipa/session/json'
Systemwide CA database updated.
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
[try 1]: Forwarding 'host_mod' to json server 'https://server.otus.local/ipa/session/json'
SSSD enabled
Configured /etc/openldap/ldap.conf
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config
Configuring otus.local as NIS domain.
Client configuration complete.
The ipa-client-install command was successful

==============================================================================
Setup complete

Next steps:
	1. You must make sure these network ports are open:
		TCP Ports:
		  * 80, 443: HTTP/HTTPS
		  * 389, 636: LDAP/LDAPS
		  * 88, 464: kerberos
		  * 53: bind
		UDP Ports:
		  * 88, 464: kerberos
		  * 53: bind
		  * 123: ntp

	2. You can now obtain a kerberos ticket using the command: 'kinit admin'
	   This ticket will allow you to use the IPA tools (e.g., ipa user-add)
	   and the web user interface.

Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
files is the Directory Manager password
```

</details>


<details><summary>см. лог `kinit admin`</summary>

```text
Password for admin@OTUS.LOCAL: 
```

</details>


<details><summary>см. лог `klist`</summary>

```text
Ticket cache: KEYRING:persistent:0:0
Default principal: admin@OTUS.LOCAL

Valid starting       Expires              Service principal
06.07.2023 20:47:52  06.07.2023 20:47:52  krbtgt/OTUS.LOCAL@OTUS.LOCAL
```

</details>

### Настройка клиента

Настройка происходит автоматически с предварительно настроенным `resolv.conf`

```shell
ansible-playbook playbooks/client.yml --tags deploy > ../files/client.yml.txt
```


<details><summary>см. лог `playbooks/client.yml --tags deploy`</summary>

```text

PLAY [Playbook of client initialization] ***************************************

TASK [Gathering Facts] *********************************************************
ok: [client]

TASK [../roles/client : copy resolv.conf] **************************************
changed: [client]

TASK [../roles/client : network manager war | chattr +i /etc/resolv.conf] ******
changed: [client]

TASK [../roles/client : Check firewall-cmd state] ******************************
changed: [client]

TASK [../roles/client : Store to file firewall-cmd state] **********************
changed: [client -> localhost]

TASK [../roles/client : Set time zone] *****************************************
changed: [client]

TASK [../roles/client : Synchronize datetime | Install chrony] *****************
ok: [client]

TASK [../roles/client : Synchronize datetime | Turn on chronyd] ****************
changed: [client]

TASK [../roles/client : Set hostname] ******************************************
changed: [client]

TASK [../roles/client : Set hostname | In order to apply the new hostname] *****
changed: [client]

TASK [../roles/client : Install python-pip for pexpect promt answering] ********
changed: [client]

TASK [../roles/client : Pip install pexpect] ***********************************
changed: [client]

TASK [../roles/client : Configure iptables] ************************************
changed: [client]

TASK [../roles/client : Update network] ****************************************
changed: [client]

TASK [../roles/client : Install freeipa-client] ********************************
changed: [client]

TASK [../roles/client : Reboot VM] *********************************************
changed: [client]

TASK [../roles/client : Configure ipa-client] **********************************
changed: [client]

TASK [../roles/client : Save client_configure_log] *****************************
changed: [client -> localhost]

TASK [../roles/client : Check kinit admin] *************************************
changed: [client]

TASK [../roles/client : Store kinit admin log to file] *************************
changed: [client -> localhost]

TASK [../roles/client : Check klist] *******************************************
changed: [client]

TASK [../roles/client : Store klist log to file] *******************************
changed: [client -> localhost]

RUNNING HANDLER [../roles/client : systemctl daemon reload] ********************
ok: [client]

RUNNING HANDLER [../roles/client : Pip uninstall pexpect] **********************
changed: [client]

RUNNING HANDLER [../roles/client : Uninstall python-pip] ***********************
changed: [client]

PLAY RECAP *********************************************************************
client                     : ok=25   changed=22   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```

</details>


<details><summary>см. client-лог `firewall_cmd --state`</summary>

```text
running
```

</details>


<details><summary>см. процесс автоматизированного конфигурирования IPA-клиента посредством Ansible </summary>

```text
WARNING: ntpd time&date synchronization service will not be configured as
conflicting service (chronyd) is enabled
Use --force-ntpd option to disable it and force configuration of ntpd

Discovery was successful!
Client hostname: server.otus.local
Realm: OTUS.LOCAL
DNS Domain: otus.local
IPA Server: server.otus.local
BaseDN: dc=otus,dc=local

Continue to configure the system with these values? [no]: Skipping synchronizing time with NTP server.
User authorized to enroll computers: Password for admin@OTUS.LOCAL: 
Successfully retrieved CA cert
    Subject:     CN=Certificate Authority,O=OTUS.LOCAL
    Issuer:      CN=Certificate Authority,O=OTUS.LOCAL
    Valid From:  2023-06-07 17:17:33
    Valid Until: 2043-06-07 17:17:33

Enrolled in IPA realm OTUS.LOCAL
Created /etc/ipa/default.conf
New SSSD config will be created
Configured sudoers in /etc/nsswitch.conf
Configured /etc/sssd/sssd.conf
Configured /etc/krb5.conf for IPA realm OTUS.LOCAL
trying https://server.otus.local/ipa/json
[try 1]: Forwarding 'schema' to json server 'https://server.otus.local/ipa/json'
trying https://server.otus.local/ipa/session/json
[try 1]: Forwarding 'ping' to json server 'https://server.otus.local/ipa/session/json'
[try 1]: Forwarding 'ca_is_enabled' to json server 'https://server.otus.local/ipa/session/json'
Systemwide CA database updated.
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
[try 1]: Forwarding 'host_mod' to json server 'https://server.otus.local/ipa/session/json'
SSSD enabled
Configured /etc/openldap/ldap.conf
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config
Configuring otus.local as NIS domain.
Client configuration complete.
The ipa-client-install command was successful
```

</details>


<details><summary>см. лог `kinit admin`</summary>

```text
Password for admin@OTUS.LOCAL: 
```

</details>


<details><summary>см. лог `klist`</summary>

```text
Ticket cache: KEYRING:persistent:0:0
Default principal: admin@OTUS.LOCAL

Valid starting       Expires              Service principal
06.07.2023 20:56:11  06.07.2023 20:56:10  krbtgt/OTUS.LOCAL@OTUS.LOCAL
```

</details>

## Особенности в рамках ДЗ

Задействовал [ansible.builtin.expect](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/expect_module.html) для автоматизации ответов на промт-вопросы в терминале, ранее в ДЗ по VPN c `easy-rsa` подобное не прошло (подробнее там).

```properties
...
- name: Reconfigure ipa
  expect:
    command: ipa-server-install --uninstall
    responses:
      '(.*)Are you sure you want to continue with the uninstall procedure(.*)': 'yes'
    timeout: 600
  tags:
    - reconfigure-ipa-server

- name: Configure ipa
  expect:
    command: ipa-server-install
    responses:
      # Using last defined value only
      '(.*)Do you want to configure integrated DNS(.*)': 'yes'
      '(.*)Server host name(.*)': 'ipa-server.otus.local'
      '(.*)Please confirm the domain name(.*)': 'otus.local'
      '(.*)Please provide a realm name(.*)': 'OTUS.LOCAL'
      '(.*)Directory Manager password(.*)': 'P@ssw0rd'
      '(.*)Password \(confirm\)(.*)': 'P@ssw0rd'
      '(.*)IPA admin password(.*)': 'P@ssw0rd'
      '(.*)Do you want to configure DNS forwarders(.*)': 'yes'
      '(.*)Do you want to configure these servers as DNS forwarders(.*)': 'yes'
      '(.*)Do you want to search for missing reverse zones(.*)': 'yes'
      '(.*)You can enter additional addresses now(.*)Enter an IP address for a DNS forwarder, or press Enter to skip(.*)': '77.88.8.8'
      '(.*)You may add another(.*)Enter an IP address for a DNS forwarder, or press Enter to skip(.*)': ''
      '(.*)Do you want to create reverse zone for IP(.*)': 'yes'
      '(.*)Please specify the reverse zone name(.*)': ''
      '(.*)Continue to configure the system with these values(.*)': 'yes'
      'restarting directory server': 'yes'
    timeout: 36000
  register: ipa_server_configure_log
  tags:
    - configure-ipa-server
...
```

Но для этого необходим пакет на гостевой 
```shell
sudo yum install -y python-pip
pip install pexpect
```

Возможно ввиду использования локальной виртуализации playbook работает долго, время прeимущественно тратится на задачу `configure-ipa-server` 

Возникла ошибка при конфигурировании сервера
```text
# ERROR    IPv6 stack is enabled in the kernel but there is no interface that has ::1 address assigned. Add ::1 address resolution to 'lo' interface. You might need to enable IPv6 on the interface 'lo' in sysctl.conf.
#ipapython.admintool: ERROR    The ipa-server-install command failed. See /var/log/ipaserver-install.log for more information
```

Что требует "net.ipv6.conf.lo.disable_ipv6 = 0"  в /etc/sysctl.d/99-sysctl.conf и правки GRUB без IPv6.

## Для себя

```shell
cd ../../
cd ./034/vm/
vagrant destroy -f
vagrant up
python3 v2a.py -o ../ansible/inventories/hosts # Это уже как кредо
cd ../../
cd ./034/ansible/
ansible-playbook playbooks/server.yml --tags deploy > ../files/server.yml.txt
ansible-playbook playbooks/client.yml --tags deploy > ../files/client.yml.txt
cd ../../
./details.py 034.details.md 0

```

```shell
ipa-server-install << END
yes



P@ssw0rd
P@ssw0rd
P@ssw0rd
P@ssw0rd
yes
yes

yes
yes
8.8.8.8



yes
END
# или
ipa-server-install << 'EOF'
...
EOF
# или 
(sleep 10; echo "yes"; sleep 10; echo ""; sleep 10; echo ""; sleep 10; echo ""; sleep 10; echo "P@ssw0rd"; sleep 10;  echo "P@ssw0rd"; sleep 10; echo "P@ssw0rd"; sleep 10; echo "P@ssw0rd"; sleep 10; echo "yes"; sleep 10; echo "yes"; sleep 10; echo "8.8.8.8"; sleep 10; echo ""; sleep 10; echo "yes"; sleep 10;  echo "yes"; sleep 10;  echo ""; sleep 10;  echo "yes"; sleep 10;  echo ""; sleep 10;  echo "yes"; sleep 10; ) | ipa-server-install
```
