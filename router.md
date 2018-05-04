Port Forwarding is required when running a Gladius Node. A machine running on a public pool must be accessible by the public internet so that end users can access content stored on their node. Port forwarding is essentially a form of NAT (network address translation) and requires a little configuration on en edge router or firewall to work. Below is a list of links to documents for a couple of popular router types. Please consult your router documentation for any changes that occur between versions and please always BACK UP YOUR CONFIGURATION :)


pfsense
https://forum.pfsense.org/index.php?topic=55676.0

CIsco ASA
https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/118996-config-asa-00.html

Cisco router
https://www.cisco.com/c/en/us/support/docs/long-reach-ethernet-lre-digital-subscriber-line-xdsl/asymmetric-digital-subscriber-line-adsl/12905-827spat.html

Juniper
https://www.juniper.net/documentation/en_US/junos/topics/task/configuration/nat-port-forwarding-static-destination-address-translation.html

VyOS
https://wiki.vyos.net/wiki/NAT#Destination_NAT_.28Incoming.29

Watchguard
https://www.watchguard.com/help/docs/fireware/12/en-US/Content/en-US/configuration_examples/snat_web_server_config_example.html

Palo Alto
https://www.paloaltonetworks.com/documentation/80/pan-os/pan-os/networking/nat/nat-configuration-examples/destination-nat-with-port-translation-example

ipTables
https://www.digitalocean.com/community/tutorials/how-to-forward-ports-through-a-linux-gateway-with-iptables



Hairpin NAT

Hairpin nat allows for someone inside a network (behind a NAT) to access information that is behind another NAT at the same edge. Gladius Nodes need hairpin NAT enabled so they can access content they themselves might be hosting. This would make access to CDN content extremely fast and efficient since the content is essentially stored on a machine on the same network.

Please watch this video on how Hairpin NAT works:

https://www.youtube.com/watch?v=Bdbn1pbe74o
