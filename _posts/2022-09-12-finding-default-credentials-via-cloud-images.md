---
layout: post
title: Finding Default Credentials via Cloud Images
---

PLACEHOLDER

-----

During an engagement on a clients network, I came across a device running [Haivision Kraken Transcoder](https://www.haivision.com/products/kraken-series/). Searching Google for "Haivision Kraken default password" shows some uploaded user's guides that contain the default credentials of `haiadmin:manager` for the web interface. While the default ssh account username of `hvroot` was discoverable in the User's guide, I could not find the default password to go with.

My next step was to try and get access to a trial to view source for the application. After browsing their website I didn't see an easy way to get access to trail. However, they did offer a cloud instance in [Azure Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/haivisionsystemsinc1580780591922.haivision-kraken) and [AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-irfl3yfrpjcqo?sr=0-5&ref_=beagle&applicationId=AWSMPContessa) that you could spin up.

[[https://www.christianweiler.com/assets/kraken_azure_marketplace.png]]

After spinning up an image in Azure cloud, I search the file system for references to the ssh account username `hvroot`

```bash
grep -ir -C3 "hvroot" .

/opt/haivision/usr/bin/hai-cui-install:CONSOLE_USER=hvroot
/opt/haivision/usr/bin/hai-cui-install-CONSOLE_GROUP=hvcui
/opt/haivision/usr/bin/hai-cui-install-CONSOLE_PASSWORD=hairoot

...


/opt/haivision/etc/vfprofile.default:SUPPORT_USER='hvroot'
/opt/haivision/etc/vfprofile.default-SUPPORT_PASSWORD='$1$I9Iud94T$WJEDNFTVqI8APw/ZPhE4t.'
/opt/haivision/etc/vfprofile.default-SSH_GROUP='haissh'
/opt/haivision/etc/vfprofile.default-SUDO_GROUP='haisudo'

```

Here we find references to a probably plaintext password from an install script and a password hash. If we crack the password hash, it matches the plaintext credentials found of `hvroot:hairoot`

```batch
.\hashcat -a 0 -m 500 -r .\rules\OneRuleToRuleThemAll.rule.txt ..\hvroot.txt .\rockyou.txt
$1$I9Iud94T$WJEDNFTVqI8APw/ZPhE4t.:hairoot
```