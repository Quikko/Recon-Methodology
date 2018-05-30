# Recon Methodology

# Little Intro

I'm Quinten Van Ingh an application security specialist and in my spare time I love to hunt for bugs. I just started with bug bounty (4 weeks ago) on HackerOne and like most of you guys, I want to share my resources and other things. I consider myself to be in the beginner phase of the bug bounty sector but I try to learn every day. So if you have any suggestions, advice, tips, tricks, tools let me know !

This document is based on my own research but mostly on the talk of [Jhaddix](https://twitter.com/Jhaddix) - __The Bug Hunters Methodology v3__.

Why am I sharing this? Everything I've learned is from guys like __Jhaddix__. These people shares all the knowledge they have, to give other hackers to opportunity to grow.

Keep Posted because I'll update this page !


# Sub-domain enumeration

We can split this up in two different categories:
* Horizontal sub-domain enumeration
* Vertical sub-domain enumeration

Horizontal sub-domain enumeration examples: www.google.com, dev.google.com, maps.google.com

Vertical sub-domain enumeration are sites which are also used by the main domain. For example snapchat.com, snap.com, spectacles.com.

## Horizontal subdomain enumeration

The tools that can be used to perform horizontal sub-domain enumeration can also be split into two categories.

1. Sub-domain brute-forcing
2. Looking for sub-domains via logging, search engines, ...

### Sub-domain bruteforcing tools

Tools to use for these:
* [gobuster](https://github.com/OJ/gobuster)
```
gobuster -m dns -u $TARGET.com -t 100 -w all.txt
```
* [massdns](https://github.com/blechschmidt/massdns)
```
./subbrute.py /root/work/bin/all.txt $TARGET.com | ./bin/massdns -r resolvers.txt -t A -a -o -w massdns_output.txt
```

```
./scripts/ct.py example.com | ./bin/massdns -r lists/resolvers.txt -t A -o S -w results.txt
```

The all.txt is a collection of all the different wordlist used by all the different sub-domain bruteforcing tools. You can find it over here [Jason Haddix' subdomain compilation](https://gist.githubusercontent.com/jhaddix/86a06c5dc309d08580a018c66354a056/raw/f58e82c9abfa46a932eb92edbe6b18214141439b/all.txt)

Below the [massdns](https://github.com/blechschmidt/massdns) github page, you can see that the subbrute.py and ct.py are already included in the massdns project itself. The default subbrute.py list is a good one but is also included in all.txt


Prevent of typing all these commands over and over again every time you have a new project. Make a simple bash script like I did, for example

__subbrute.sh__

```
#/bin/bash
./massdns/scripts/subbrute.py ./massdns/lists/names.txt $1 | ./massdns/bin/massdns -r ./massdns/lists/resolvers.txt -t A -o S -w results/$1.sub.txt

```

__subbrute-big.sh__ : note the all.txt

```
#/bin/bash
./massdns/scripts/subbrute.py ./all.txt $1 | ./massdns/bin/massdns -r ./massdns/lists/resolvers.txt -t A -o S -w results/$1.sub.txt
```

__ct.sh__

```
#/bin/bash
./massdns/scripts/ct.py $1 | ./massdns/bin/massdns -r ./massdns/lists/resolvers.txt -t A -o S -w results/$1.ct.txt
```

Another possibility is to add certain function/command to your __.bash_profile__

The $1 is where the domain itself. So for example to execute __subbrute.sh__:

```
./subbrute.sh google.com
```

This will write all the possible subdomains in a directory results named google.com.txt.

__Massdns__ will default provide all the A records and CNAME records it finds for the sub domains in the results (which is awesome). Later more on this.

__Note__, there are way more tools which has the option to perform bruteforce attacks against a certain domain. For example:
* [amass](https://github.com/caffix/amass)
* [subfinder](https://github.com/Ice3man543/subfinder)
*  ...

### Looking for sub-domains via logging, search engines, ...

Tools which can be used:
* [amass](https://github.com/caffix/amass)
* [subfinder](https://github.com/Ice3man543/subfinder)
* [Sublist3r](https://github.com/aboul3la/Sublist3r)
* [aquatone](https://github.com/michenriksen/aquatone)
* [domlink](https://github.com/vysec/DomLink)


Next to tools, there are also sites which can be used to find some sub-domains.

* https://www.virustotal.com
* https://censys.io/
* https://dnsdumpster.com/
* https://securitytrails.com/dns-trails
* https://www.shodan.io/ :
  * org: "Tesla Motors"
* Google dorks:
  * examples: http://zeroday-security.com/find-bug-get-bounty-using-google-dorks/


Now I know that most of the sites above are included in the tools. Just provide them your API key and play with it.

## Vertical sub-domain enumeration

Look for reserved IP blocks and look to the IP's. This way you can see other domains used by an organazations as well sub-domains. Below you'll find some useful links:

* https://bgp.he.net/
* https://whois.arin.net/ui/query.do
* https://apps.db.ripe.net/db-web-ui/#/fulltextsearch
* https://viewdns.info : DNS and WHOIS.
* https://reverse.report
* Google dork:
  * ip:

__Note__ this can easily be script for example: https://reverse.report/search?q=test.com where you replace "test.com" by a parameter you provide to your script. Also look if the above sites have an API available. So you can easily lookup for domains and automate this.

## Acquisitions

Next to vertical and horizontal subdomain enumeration, Jhaddix also mentioned "acquisitions" in his talk:

* www.crunchbase.com/

Enter the the company or a person into the search bar at the top of the page and look at the acquisitions.


## Linked Discovery

For this you'll need the tool [BurpSuite](https://portswigger.net/burp). I highly recommend you to go and watch how this works on [Jhaddix his twitter account](https://twitter.com/twitter/statuses/972926512595746816). I'll try to explain the steps to do it below:
- Turn off passive scanning (Scanner tab -> turn off passive scanning)
- Set forms auto to submit (Spider tab --> options -->  Application login : Handle as ordinary forms. )BE CAREFUL FOR EMAIL FORMS !!!
- Set scope to advanced control and use string of target name - -> target tab --> enable : use advanced scope control  --> match it to a keyword (not a normal FQDN) e.g. Tesla in the host field
- Show only in scope items.
- Walk+Browse through the website
- Then spider all hosts recursively.

With this you'll also find some extra sub-domains.

## CSP-Header

Always go through the CSP-Header. In here you can also find domains/subdomains.


# What now?

So you have a list of sub-domains and form some tools also the IP's. From here we can perform several steps.

## Portscanner

Gather from all the (sub)domains the IP addresses and throw them in a masscan or any other portscanner:

* [Masscan](https://github.com/robertdavidgraham/masscan)
* Nmap
* [Sparta](https://github.com/SECFORCE/sparta)

I prefer using Masscan because it's really fast. The command provided by Jhaddix:

```
masscan -p1-65535 -iL $TARGET_LIST --max-rate 100000 -oG $TARGET_OUTPUT
```

To gather the IP's of the (sub) domains. There is a script available in the talk from Jhaddix. I'll add this later.

Look through the ports and see if there are old versions or services which should not be open to the public. To perform a bruteforce attack Jhaddix gave a nice tool:

* [Brutespray](https://github.com/x90skysn3k/brutespray)

__Note__ the output file of masscan needs to be in a .gnamp or .xml file. (nmap -oG)

Example of brutespray:

```
 python brutespray.py --file nmap.gnmap -U /usr/share/wordlist/user.txt -P /usr/share/wordlist/pass.txt --threads 5 --hosts 5
```

## Screenshots

Get all the domains in one file, make sure all domains are listed only once (uniq command in Linux). Provide the list to a tool which will make a screen shot of every domain:

* [EyeWitness](https://github.com/ChrisTruncer/EyeWitness)

An example of usage:

```
sudo ./EyeWitness.py -f test.txt  --headless --prepend-https
```

The __--prepend-https__ options make sure it will take a screen shot of port 80 (HTTP) and 443(HTTPS)

* [WebScreenshot](https://github.com/maaaaz/webscreenshot.git)

* [wmap](https://chrome.google.com/webstore/detail/wmap/pflahkdjlekaeehbenhpkpipgkbbdbbo) : Chrome extension.

When you've used [aquatone](https://github.com/michenriksen/aquatone)-discover to gather sub-domains, you can use the __aquatone-gather__ to make screenshots of the sub-domains. Make sure you first run __aquatone-scan__.

## 401/403 response

After all the screenshots are made, go through the list and search for the ones which gave you a 401/403 response. Copy and paste these and throw them in a list. Use the waybackmachine and check if you can find some directories. Maybe the organization forgot to put the right permissions on certain directories or files. Tools u can use for this:

* [WayBackUrls](https://github.com/tomnomnom/waybackurls)
* [WayBackUnifier](https://github.com/mhmdiaa/waybackunifier)
* [ReconCat](https://github.com/daudmalik06/ReconCat)

Another tip is to run a tool which perform directory brute forcing on it. There are several tools for this:

* [dirbuster](https://sourceforge.net/projects/dirbuster/)
* [dirb](http://dirb.sourceforge.net/)
* [gobuster](https://github.com/OJ/gobuster)
* [RobotsDisallowed](https://github.com/danielmiessler/RobotsDisallowed)
* [snallygaster](https://github.com/hannob/snallygaster.git)

There are many wordlists you can use here:

* [Jhaddix Content_discovery_all.txt](https://gist.github.com/jhaddix/b80ea67d85c13206125806f0828f4d10)
* The one(s) provided with the Tools
* [Seclist](https://github.com/danielmiessler/SecLists) : Which contains several wordlists with several purposes (DNS, passwords, usernames, payloads,...)
* ...

## Identify

So you got your screenshots from all the sub-domains but don't know where to start? Look for the ones which are custom made (ASP.NET,...). Tools which can help you to identify these are:

* [Wappalyzer](https://www.wappalyzer.com/) : Extension for chrome/firefox.
* [Builtwith](https://builtwith.com/)  
* [Vulners Web Scanner](https://chrome.google.com/webstore/detail/vulners-web-scanner/dgdelbjijbkahooafjfnonijppnffhmd)

Keep these domains in a list and start looking at them when your whole recon phase is done.

## Linkfinder

Another tip provided by Jhaddix and several other infosec guys is, always check the JavaScript code. It could be possible that you'll find new endpoints, hardcoded credentials, hardcoded JWT signing key, ... Especially when they are making use of a CMS based on JavaScript.

Tools to make your life easier:

* [LinkFinder](https://github.com/GerbenJavado/LinkFinder)
* [JSParser](https://github.com/nahamsec/JSParser)
* [relative-url-extractor](https://github.com/jobertabma/relative-url-extractor)

Do you need to manually put all the used JavaScript files into the tool ? Not when using BurpSuite:

* In BurpSuite, Go to Target tab.
* Right click on the subdomain (where you want to analyze the JS from)
* Click on Engagements tools
* Select Find scripts
* CTRL + A to select all the scripts Burp has found.
* Right click and select "Copy Selected URLS"
* Paste them into a file and run a command like uniq. This to remove the duplicates.
* Paste all  the urls in LinkFinder/JSParser.
* Enjoy ;)

Via these tools, you can easily find new endpoints.

# Subdomain takeover

When you have a main list of all the subdomains, you can start looking for subdomain takeovers. You can provide your list of subdomains to a tool like [SubOver](https://github.com/Ice3man543/SubOver). I higly recommended resource is https://github.com/EdOverflow/can-i-take-over-xyz. In here you'll find the default messages provided by several services which can lead to a subdomain takeover.

When you are using __aquatone__, you can use  __aquatone-takeover__.

# AWS - Buckets

To find buckets of an organization:

* [slurp](https://github.com/bbb31/slurp)
* [S3scanner](https://github.com/sa7mon/S3Scanner)
* [teh_s3_bucketeer](https://github.com/tomdev/teh_s3_bucketeers)

Another tip it's not because a bucket is not publicly readable that the permissions to write or delete a file into/from the bucket are correctly configured. Always test writing into the bucket.

# GitHub

Github is great for searching things (credentials, keys, endpoints, services, APK's/IPA's, ..) of an organization. You can find these by using sort of github dorks. I highly recommend resource/tool :

* https://github.com/techgaun/github-dorks

I highly suggest to check the commits of a certain repository.

To be honest, I need to do more resources for tools.

# WAF

To check which WAF is used on a certain subdomain, I make use of [wafwoof](https://github.com/EnableSecurity/wafw00f).

To bypass a WAF, you'll need to have the original IP of the webserver. This can be obtained via several ways:

* https://censys.io/
* https://dnsdumpster.com/
* https://securitytrails.com/dns-trails
* http://viewdns.info/iphistory/?domain=tesla.com

A good friend/colleague of mine wrote a tool for this which I only can recommend:
* [bypass-firewalls-by-dns-history](https://github.com/vincentcox/bypass-firewalls-by-DNS-history)

Another technique that can be used to obtain the original IP address:

When the website has a certain "subscribe" functionality or a build in functionality which send you an email, check the headers of the mail and look for the IP.

Once you got several IP's, you can test these with a simple curl command

```
curl --silent --fail -H "Host: www.test.com" http://$IP_YOU_HAVE_FOUND
```

When the html of the curl command is the same as the one you can see on www.test.com, you got a WAF bypass.

You can also just use the IP as an URL or change your host file (or the one in burp.)

## Thank you

First of all I want to thank bug bounty platforms like [BugCrowd](https://twitter.com/bugcrowd) and [HackerOne](https://twitter.com/Hacker0x01) to organize events and give people like me the opportunity to learn and enter the bug bounty community. Things like the levelupx02 by BugCrowd really helps for people like me.

Secondly I want to thank all the security researchers who share their knowledge and tools they have written. Also thanks to all the speakers at BugCrowd event: __levelupx02__
