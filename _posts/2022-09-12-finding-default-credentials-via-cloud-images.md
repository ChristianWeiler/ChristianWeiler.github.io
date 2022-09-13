---
layout: post
title: Finding Default Credentials via Cloud Images
---

During an engagement on a client's network, I came across a device running [Haivision Kraken Transcoder](https://www.haivision.com/products/kraken-series/). 

[[https://www.christianweiler.com/assets/2022-09-12/kraken_web_login.png]]

Searching Google for "Haivision Kraken default password" shows some uploaded user's guides that contain the default credentials of `haiadmin:manager` for the web interface. While the default ssh account username of `hvroot` was discoverable in the user's guide, I could not find the default password that was associated with the account.

My next step was to try and get access to a trial to view source for the application. After browsing their website I didn't see an easy way to get access to trail. However, they did offer a cloud instance in [Azure Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/haivisionsystemsinc1580780591922.haivision-kraken) and [AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-irfl3yfrpjcqo?sr=0-5&ref_=beagle&applicationId=AWSMPContessa) that you could use to create your own instance.

[[https://www.christianweiler.com/assets/2022-09-12/kraken_azure_marketplace.png]]

After spinning up an image in Azure cloud, I attempted to ssh in with the credentials specified when creating the instance, but would get an access denied. For a quick and easy way to bypass this, I used the "Run command" feature in Azure to execute a bash reverse shell.

[[https://www.christianweiler.com/assets/2022-09-12/kraken_azure_run_cmd.png]]


After getting command line access to the host, I searched the file system for references to the ssh account username `hvroot`.

```bash
grep -ir -C3 "hvroot" .

...

/opt/haivision/usr/bin/hai-cui-install:CONSOLE_USER=hvroot
/opt/haivision/usr/bin/hai-cui-install-CONSOLE_GROUP=hvcui
/opt/haivision/usr/bin/hai-cui-install-CONSOLE_PASSWORD=hairoot

...


/opt/haivision/etc/vfprofile.default:SUPPORT_USER='hvroot'
/opt/haivision/etc/vfprofile.default-SUPPORT_PASSWORD='$1$I9Iud94T$WJEDNFTVqI8APw/ZPhE4t.'
/opt/haivision/etc/vfprofile.default-SSH_GROUP='haissh'
/opt/haivision/etc/vfprofile.default-SUDO_GROUP='haisudo'

...

```

Here we find references to what is likely the plaintext password from an install script and a references to a password hash for the account. If we crack the password hash, it matches the plaintext credentials found.

```batch
.\hashcat -a 0 -m 500 -r .\rules\OneRuleToRuleThemAll.rule.txt ..\hvroot.txt .\rockyou.txt
$1$I9Iud94T$WJEDNFTVqI8APw/ZPhE4t.:hairoot
```

The default ssh credentials for the Haivision Kraken device were `hvroot:hairoot`.