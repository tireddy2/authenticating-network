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
    fullname: Tiru Reddy
    organization: Nokia
    email: "kondtir@gmail.com"



normative:
  DNR:    I-D.ietf-add-dnr
  DDR:    I-D.ietf-add-ddr

informative:
  Evil-Twin:
    title: "Evil twin (wireless networks)"
    date: 2022-06
    target: "https://en.wikipedia.org/wiki/Evil_twin_(wireless_networks)"
    author:
      -
        name: Wikipedia

  RFC8792: RFC8792


--- abstract

When connecting to a wired or wireless network, users assume the same network exists each time.  This
document describes how a network client can verify it is, in fact, connecting to the same network,
without relying on layer 1 or layer 2 security.

--- middle


# Introduction

When a client connects to a network -- wired or wireless -- the user
or their device want to be sure the connection is to the expected
network, as different networks provides different services --
performance, security, access to split-horizon DNS servers, and so on.
Although 802.1X provides layer 2 security for both Ethernet and WiFi
networks achieving the goals of this document, 802.1X is not widely
deployed and unavailable on LTE and 5G networks.

On wired networks a malicious actor or innocent cabling or VLAN
configuration mistakes can cause the same physical Ethernet jack to
connect to a different network.  On WiFi networks a malicious actor
can operate a rogue access point with the same SSID and WPA-PSK as the
victim network [Evil-Twin].

This document describes how a wired or wireless client can utilize
network-advertised encrypted DNS servers to verify initial connection
to the intended network and automatically verify re-connection to the
same intended network.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Procedure

Client connects to network and obtains network information via DHCPv4, DHCPv6, RA, or 3GPP mechanism.
The network indicates its encrypted DNS server using either [DNR] or [DDR].  The client connects
to that encrypted DNS server, completes the TLS handshake and performs public key validation of
the presented certificate.

> todo: need name of, and citation to, the 3gpp mechanism.

As a wired network has no 'name', the network name can be considered blank. This
isn't ideal, but it still allows the client to distinguish between the first
connection to a network versus re-connecting to a network.

The client device's imprecise location (if known) and the WiFi network
name (SSID) can be associated with the encrypted DNS server's
SubjectAltName that was learned via [DNR] or [DDR].

If this is the first time connecting to that encrypted DNS server's identity,
an action can be performed such as prompting the user for verification,
verifying the encrypted DNS server's certificate with the fingerprint provided
in an extended WiFi QR code ({{qr}}), consulting a crowd-sourced database,
or -- perhaps best -- using a matching SSID and SubjectAltName described
in {{avoid-tofu}}.  After this step, the relationship of location, WiFi network
name, and SubjectAltName are stored on the client.

If this is not the first time connecting to this same SSID,
the client location, WiFi network name, and SubjectAltName should all
match for this re-connection.  If the SubjectAltName differs, this indicates
a different network than expected -- either a different network

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
number can safely be disclosed to neighbors.

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

