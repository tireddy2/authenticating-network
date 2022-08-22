---
title: "Authenticating a Network Connection"
abbrev: "Authenticating a Network Connection"
category: info

docname: draft-wing-authenticating-network-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - network
 - security
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "danwing/authenticating-network"
  latest: "https://danwing.github.io/authenticating-network/draft-wing-authenticating-network.html"

author:
 -
    ins: D. Wing
    fullname: Dan Wing
    organization: Citrix
    email: "dwing-ietf@fuggles.com"
 -
    ins: T. Reddy
    fullname: Tirumaleswar Reddy
    organization: Nokia
    email: "kondtir@gmail.com"



normative:
  DNR:    I-D.ietf-add-dnr
  DDR:    I-D.ietf-add-ddr
  AKA:    I-D.ietf-emu-aka-pfs

informative:
  Evil-Twin:
    title: "Evil twin (wireless networks)"
    date: 2022-06
    target: "https://en.wikipedia.org/wiki/Evil_twin_(wireless_networks)"
    author:
      -
        name: Wikipedia

  IEEE802.11:
    title: "IEEE802.11"
    date: 2022-08
    target: "https://en.wikipedia.org/wiki/IEEE_802.11"
    author:
      -
        name: Wikipedia

  RFC8792: RFC8792
  RFC8110: RFC8110


--- abstract

When connecting to a wireless network, users assume the same network exists each time.  This
document describes how a network client can verify it is, in fact, connecting to the same network,
without relying on layer 1 or layer 2 security.

--- middle


# Introduction

When a client connects to a network -- wireless -- the user
or their device want to be sure the connection is to the expected
network, as different networks provides different services --
performance, security, access to split-horizon DNS servers, and so on.
Although 802.1X provides layer 2 security for both Ethernet and WiFi
networks achieving the goals of this document, 802.1X is not widely
deployed and unavailable on residential or small office/home office
and Wi-Fi hotspots (e.g., coffeeshop, hotel, airport network).

On WiFi networks a malicious actor can operate a rogue access point
with the same SSID and WPA-PSK as the victim network [Evil-Twin].
In many deployments (for example, coffee shops and bars) offer free Wi-Fi as an incentive to customers
to enter and remain in the premises.  Many customers will use the availability of free Wi-Fi
as a deciding factor in which business to patronize. Since these businesses are not Internet
service providers, they are often unwilling and/or unqualified to perform complex
configuration on their network.  In addition, customers are generally unwilling to do
complicated provisioning on their devices just to obtain free Wi-Fi.
This leads to a popular deployment technique -- a network protected
using a shared and public Pre-Shared Key (PSK) that is printed on a
sandwich board at the entrance, on a chalkboard on the wall, or on a
menu.  The PSK is used in a cryptographic handshake, defined in
[IEEE802.11], called the "4-way handshake" to prove knowledge of the
PSK and derive traffic encryption keys for bulk wireless data. The same
deployement technique is typically used in residential or small office/home office
networks. If the Pre-Shared Key (PSK) for wireless authentication
is the same for all clients that connect to the same WLAN, the shared key
will be available to all nodes, including attackers, so it is possible to
mount an active on-path attack.

This document describes how a wireless client can utilize
network-advertised encrypted DNS servers to verify initial connection
to the intended network and automatically verify re-connection to the
same intended network. The proposed mechanism is also useful in deployments
using Opportunistic Wireless Encryption [RFC8110] and in 5G networks where the long-term
key in the SIM card on the UE is compromised (Section 1 of [AKA]).

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Procedure

Client connects to network and obtains network information via DHCPv4, DHCPv6, RA, or 3GPP mechanism.
The network indicates its encrypted DNS server using either [DNR] or [DDR].  The client connects
to that encrypted DNS server, completes the TLS handshake and performs public key validation of
the presented certificate.

The client device's imprecise location (if known), the WiFi network
name (SSID) and Basic Service Set Identifier (BSSID) can be associated with the encrypted DNS server's
Authentication Domain Names (ADNs) that were learned via [DNR] or the encrypted DNS server's IP addresses
that were learned via the mechanism dicsussed in Section 4 of [DDR]. If DDR is used for discovery,
the client would have to perform verified discovery explained in Section 4.2 of [DDR]. If DNR is used,
the encrypted DNS server's identity will be the ADNs; if DDR is used, the resolver identity will be the
encrypted DNS server's IP addresses.

If this is the first time connecting to that encrypted DNS server's identity,
an action can be performed such as prompting the user for verification,
verifying the encrypted DNS server's certificate with the fingerprint provided
in an extended WiFi QR code ({{qr}}), consulting a crowd-sourced database,
reputation system, or -- perhaps best -- using a matching SSID and SubjectAltName described
in {{avoid-tofu}}.  After this step, the relationship of location, WiFi network
name, BSSID, encrypted resolver disovery mechanism and encrypted DNS server's identity
are stored on the client.

An example is provided below for illustrative purpose.

~~~
SSID:"Example"; BSSID:"d8:c7:c8:44:32:40":
Discovery:"DNR"; Identity:"resolver1.example.com".
SSID:"Example2"; BSSID:"d8:c7:c8:44:32:42":
Discovery:"DDR"; Identity:"8.8.8.8".
~~~

If this is not the first time connecting to this same SSID,
the client location, WiFi network name, BSSID, encrypted resolver disovery mechanism and encrypted DNS server's
identity should all match for this re-connection.  If the encrypted DNS server's identity differs, this indicates
a different network than expected -- either a different network or a Evil Twin.

> todo: if a network advertises 8.8.8.8 via DNR or DDR, we can't
    detect an evil twin.  How do we identify 8.8.8.8 as a public DNS
    server?  Could we say the network-advertise encrypted DNS server
    has to be on the same network (RFC1918 space or same /64) as the
    client obtained??  But that seems somewhat constraining.  Another
    idea is "if you've seen this same certificate via another SSID",
    but that doesn't work well, either:  for example, I have a single
    DNS server in my house for all of my various SSIDs (guest, IoT,
    home network, work network).  Hmm.  Need more ideas.


# Avoiding Trust on First Use {#avoid-tofu}

Trust on First Use can be avoided if the SSID name and DNS server's
Subject Alt Name match.  Unfortunately such a constraint disallows
vanity SSID names.  Also, social engineering attacks gain additional
information if the network's physical address
(123-Main-Street.example.net) or name (John-Jones.example.net) is
included as part of the SSID.  Thus the only safe SSID name provides
no information to assist social engineering attacks such as a
customer number (customer-123.example.net), assuming the customer
number can safely be disclosed to neighbors. However, the above
attack may not be applicable for public Wi-Fi hotspots (e.g, 123-Main-Street.cofeeshop.com).

# Common WiFi Names

(( probably want to delete this section ))

Some WiFi names are pretty common such as "Airport WiFi" or "Hotel WiFi"
or "Guest" but are distinct networks with different WPA-PSK or are not
using security at all ("open" networks).

By using device location, client devices can avoid spurious



# Security Considerations

The network authentication mechanism relies on an attacker's inability
to obtain a signed certificate for the victim's domain name.


# IANA Considerations

This document has no IANA actions.


--- back

# Extending WiFi QR Code {#qr}

This section is non-normative and merely explains how extending the WiFi QR code could work.  QR codes come with their
own security risks, most signficant that an attacker can place their own QR code over a legitimate QR code.

Several major smartphone operating systems support a QR code with the following format for the SSID "example" with WPA-PSK "password",

~~~
WIFI:T:WPA;S:example;P:password;;
~~~

This could be extended to add a field containing the fingerprint of the encrypted DNS server
certificate.  As several DNS servers can be included in the QR code with "D:", each DNS server with
its own certificate using [RFC8792] line folding,

~~~
WIFI:T:WPA;S:example;P:password; \
D:df81dfa6b61eafdffffe1a250240db5d2e6cee25, \
D:28b236db27ff688f919b171e59e2fab81f9e4f2e;;
~~~



# Acknowledgments
{:numbered="false"}

This document was inspired by IETF review comments from Paul Wouters and Wes Eddy.

