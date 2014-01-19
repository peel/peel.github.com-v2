---
layout: post-no-feature
title: "Use iPad like a hacker"
category: articles 
comments: true
date: '2012-09-08 17:44:45'
---

Taking [notes](http://itunes.apple.com/au/app/day-one-journal/id421706526?mt=8) and [reading](http://itunes.apple.com/au/app/ibooks/id364709193?mt=8) [awesome dev books](http://pragprog.com/book/vsscala/programming-scala) on iPad is heaps fun. However making software come to life on iPad is so gravy it should come with a bowl of rice. Unless you are a IDE-dependent GUIist, the following might be helpful and give you some insight on how I successfuly migrated from desktop to a mobile workspace. With a desktop machine I still use whenever I'm around.

<ul class="contentList plus">
    <li>DDNS</li>
    <li>Port Forwarding</li>
    <li>VPN</li>
    <li>iOS config: VPN, SSH</li>
    <li>OS X setup</li>
</ul>

<!-- more start -->
### DDNS
Having a static external IP is a luxury that comes at a price and is available only to lucky few. Dynamic dns is a free solution to the issue. In the past I've been using dyndns.org but as for now they only offer paid service. My turboawesome [70€](http://www.amazon.com/Sapido-GR-1736-Wireless-N-Dual-band-Gigabit/dp/B008QWO59G) [Sapido GR-1736](http://www.sapidotech.com/products.php?act=view&no=118) suggested me a few alternatives including [changeIP](http://changeip.com) that made it easy to get a nifty & free of charge dyndns domain. Setting it all up in the admin console is like shooting fish in a barrel. 

---

### Port Forwarding
As you already are in the router console, forward the ports needed for the VPN connection. 

{% highlight bash linenos %}
    PPTP: TCP 1723   <- Used by PPTP control path
          GRE (value 47)   <- Used by PPTP data path
        
    L2TP: UDP 500    <- Used by IKEv1 (IPSec control path)
          UDP 4500   <- Used by IKEv1 (IPSec control path)
          ESP (value 50)   <- Used by IPSec data path
{%endhighlight%}
---
 
### VPN
Unless you are using Darwin-based OS you probably need some sort of VPN app. I am not quite sure what are options here. Feel free to drop me a comment.  
As my OS of choice is OSX and at times I make use of ArchLinux it is simple enough to utilize features available out-of-the-box and avoid unnecessary window clutter

#### **OSX VPN**
Since Snow Leopard Apple does not expose an easy way for handling VPN server magic in non-server edition. However simply typing `which vpnd` will tell you where to start from.  
Thus, we can do it the hacker or the easy way whereas the hacker way is the free way. Either way you'll need a few things to start with: ordinary username, password amd shared secret. The latter one's the code that's stored in the OSX Key Chain and exchanged at the handshake.
#### Easy way
No real magic here. EasyVPN is a $5 alternative to terminal witchcraft. Just download, fill in as depicted below and you're good to go.   

![easyvpn](/assets/post/1209/easyvpn.png)  

Not many alternatives here, huh? Fortunately this is perfectly enough.
I am no expert in VPNs but do know L2TP/PPTP are perfectly enough for my personal taste.
PS. the app comes in a free bersion that won't be able to start up w/ the OS but aside from that has all the features you'll need>
#### Hacker way
And this is where the fun begins.  

######Secret key
Let's start with storing the secret in the keychain:

{% highlight bash linenos %}
$ sudo security add-generic-password -a com.apple.ppp.l2tp 
    \ -s com.apple.net.racoon 
    \ -T /usr/sbin/racoon 
    \ -p "secret key" /Library/Keychains/System.Keychain
{% endhighlight %}
    
Hint: From what I read on blogs /relating Leopard version/ vpnd is the server itself and racoon does the handshake.

Now go to **Utilities > Keychain** to see the bells and whistles and you'll see the com.apple.net.racoon item listed as a 'application password'. Doubleclick, in Access Control select **Allow all applications to access this item** Pretty spiffy.

######vpnd service
The config for the VPN server is stored in `/Library/Preferences/com.apple.RemoteManagement.plist`.  
Contents of the file should be about the same as below.  

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>ActiveServers</key>
        <array>
                <string>com.apple.ppp.l2tp</string>
        </array>
        <key>Servers</key>
        <dict>
                <key>com.apple.ppp.l2tp</key>
                <dict>
                        <key>DNS</key>
                        <dict>
                                <key>OfferedSearchDomains</key>
                                <array/>
                                <key>OfferedServerAddresses</key>
                                <array>
                                        <string>8.8.8.8</string>
                                        <string>8.8.4.4</string>
                                </array>
                        </dict>
                        <key>IPv4</key>
                        <dict>
                                <key>ConfigMethod</key>
                                <string>Manual</string>
                                <key>DestAddressRanges</key>
                                <array>
                                        <string>192.168.1.150</string>
                                        <string>192.168.1.200</string>
                                </array>
                                <key>OfferedRouteAddresses</key>
                                <array/>
                                <key>OfferedRouteMasks</key>
                                <array/>
                                <key>OfferedRouteTypes</key>
                                <array/>
                        </dict>
                        <key>Interface</key>
                        <dict>
                                <key>SubType</key>
                                <string>L2TP</string>
                                <key>Type</key>
                                <string>PPP</string>
                        </dict>
                        <key>L2TP</key>
                        <dict>
                                <key>IPSecSharedSecret</key>
                                <string>com.apple.ppp.l2tp</string>
                                <key>IPSecSharedSecretEncryption</key>
                                <string>Keychain</string>
                                <key>Transport</key>
                                <string>IPSec</string>
                        </dict>
                        <key>PPP</key>
                        <dict>
                                <key>AuthenticatorProtocol</key>
                                <array>
                                        <string>MSCHAP2</string>
                                </array>
                                <key>LCPEchoEnabled</key>
                                <integer>1</integer>
                                <key>LCPEchoFailure</key>
                                <integer>5</integer>
                                <key>LCPEchoInterval</key>
                                <integer>60</integer>
                                <key>Logfile</key>
                                <string>/var/log/ppp/vpnd.log</string>
                                <key>VerboseLogging</key>
                                <integer>1</integer>
                        </dict>
                        <key>Server</key>
                        <dict>
                                <key>Logfile</key>
                                <string>/var/log/ppp/vpnd.log</string>
                                <key>MaximumSessions</key>
                                <integer>128</integer>
                                <key>VerboseLogging</key>
                                <integer>1</integer>
                        </dict>
                </dict>
        </dict>
</dict>
</plist>
{% endhighlight %}

The important parts:  
- OfferedServerAddresses = DNS Servers  
- DestAddressRanges = IP range

Make sure the rights to file are correctly set:
{% highlight bash %}$ chown root:wheel /Library/Preferences/SystemConfiguration/com.apple.RemoteAccessServers.plist{% endhighlight %}

######create log file
{% highlight bash %}$ touch /var/log/ppp/vpnd.log{% endhighlight %}

######setup users/secrets
For the secret authentication using chap, you'll need to setup your credentials in `/etc/ppp/chap-secrets` like that:


{% highlight bash linenos %}
    #client server secret ip_address
    #for 3g connection you should set it like this:
    username2 * secret2 *
{% endhighlight %}

######fire up the service on startup
Yup, pretty important one, isn't it?  
{% highlight bash %}$ vi -p /System/Library/LaunchDaemons/com.apple.ppp.l2tp.plist /System/Library/LaunchDaemons/com.apple.ppp.pptp.plist {% endhighlight %}
This time contents should look like:

{% highlight xml %}<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>Label</key>
        <string>com.apple.ppp.l2tp</string>
        <key>OnDemand</key>
        <false/>
        <key>Program</key>
        <string>/usr/sbin/vpnd</string>
        <key>ProgramArguments</key>
        <array>
                <string>vpnd</string>
                <string>-x</string>
                <string>-i</string>
                <string>com.apple.ppp.l2tp</string>
        </array>
        <key>ServiceIPC</key>
        <false/>
</dict>
</plist>
{% endhighlight %}

######check if loads correctly:  
{% highlight bash %}launchctl load /System/Library/LaunchDaemons/com.apple.ppp.l2tp.plist{% endhighlight %}

######and you're done. 

#### OSX Server Way
Fire up VPN server utility and set up the same way as for EasyVPN:

![osxserver](/assets/post/1209/osxservervpn.png)

Price tag for OSX Server is double the standard version. That's your call but the backend for VPN server features is pretty much the same as in standard div>

---

### iOS setup
Head to **Settings > General > Network > VPN** and fill in the form.  

![ipad-vpn](/assets/post/1209/ipad-vpn.png)

Again, no dark arts here. Piece of cake.
Aside from that I use [iSSH](http://itunes.apple.com/us/app/issh-ssh-vnc-console/id287765826?mt=8) for connecting via SSH/VNC and [Splashtop Mobile](http://itunes.apple.com/us/app/splashtop-2-remote-desktop/id382509315?mt=8) at times. The latter one needs a server app running on desktop but provides a golden ratio between performance and quality.

---

### OSX setup
To fully prepare a real development env I use:
tmux, zsh, vim w/ a bunch of totally ripper plugins and almighty homebrew to manage packages. But hey, you've got SSH and VNC, what else should I say?
Also I try to keep the most current setup in a [git repository](http://github.com/peel/dotfiles). 
<!-- more end -->
