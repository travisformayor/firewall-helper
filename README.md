# qvm-firewall wrapper script

Tool for mass adding accepted ips/hosts, and applying other rules and managment for vpn support in the QubesOS firewall. This is simply a wrapper for automating certain vm-firewall commands. The -f [file] flag takes a file with a different ip/host on each line, and mass adds them as accepted. The -a [actions] flag handles other tasks.

### Syntax
```
> ./fhelper -h
Usage: ./fhelper -flag []
  -h Help. Displays usage info.
  -q [vm name] Pick the qube to work on.
  -f [filename] File with list of ips on newlines.
  -c [comment] Firewall rule comment text for imported ips.
  -v Verbose. Lists all the ip's as they are added.
  -a [action] dns | icmp | drop | clear | fix
    dns: Adds 'accept dns' rule to bottom.
    icmp: Adds 'drop icmp' rule to bottom.
    drop: Adds 'drop everything else' rule to bottom.
    reset: Deletes all added rules, resets to default 'accept all' rule.

    ### Important Note ###
    Rules should be ordered: accepted ips -> accept dns -> drop icmp -> drop all else
    Check your rules with 'qvm-firewall <qube> list'
```
### Example
Only allow connection to a set of VPN servers.In this example you have a text file 'vpn.txt' which contains a bunch of ip addresses for vpn servers you want to allow connection to, each one on a newline.

```
fhelper -q vm-name -f vpn.txt -c "VPN Servers"
```

#### Extra note: Add DNS hijacking script
( The following is from https://mullvad.net/en/guides/qubes-os-4-and-mullvad-vpn/ )

Add the follwing to the file /rw/config/qubes-firewall-user-script 
Be sure the change the values in [brackets]. To find out your vif* ip address issue:  ip a | grep -i vif in a terminal (make sure you have the APP VM assigned before you do this, otherwise it will not show up)

```
#!/bin/bash
virtualif=[Put the ip of your vif* interface here]
vpndns1=[Put ip of dns server here]
vpndns2=[Put ip of dns server here]
iptables -F OUTPUT
iptables -I FORWARD -o eth0 -j DROP
iptables -I FORWARD -i eth0 -j DROP
iptables -F PR-QBS -t nat
iptables -A PR-QBS -t nat -d $virtualif -p udp --dport 53 -j DNAT --to $vpndns1
iptables -A PR-QBS -t nat -d $virtualif -p tcp --dport 53 -j DNAT --to $vpndns1
iptables -A PR-QBS -t nat -d $virtualif -p udp --dport 53 -j DNAT --to $vpndns2
iptables -A PR-QBS -t nat -d $virtualif -p tcp --dport 53 -j DNAT --to $vpndns2
```

This is to redirect DNS requests to your specified DNS server for all AppVMs that use the vpnVM.
