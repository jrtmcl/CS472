# Part 1

| Problem (from 1974 paper OR modern problem) | Solution Proposed by Paper OR Why Not Addressed | How We See This Today |
|---|---|---|
| How do you prevent others from reading your data as it crosses multiple networks? | This was not addressed in the paper because it didn't apply to the problem being solved. The foundation of the internet and network communication that they were discussing was not something where privacy was a concern, as they were more interested in the communication of the networks. | In the present day this is resolved with network encryption. As an extra layer of security, the encryption helps hide the information that is being sent between networks. Although, there is still some exposure, so most places and people use a VPN to add extra layers of encryption to the data. |
| How do you prevent fast senders from overwhelming slow receivers across network boundaries? | This is addressed in the flow control portion of the text, where they state that a receiver can reduce the window (even when packets are still coming) to limit the sender and discard incoming packets. In turn, the receiver will then reiterate the window size to the send. Or, the receiver can choose to open their window to accommodate the sender. | The flow control portion of the text covers the same idea of how we see this today. We still have a receiver window in the TCP that requires fast senders to acknowledge and adhere to a certain amount of buffer space the receiver can handle. The receiver can chose to reduce or increase the window on their own terms, but the sender must adhere. |
| How do you efficiently deliver the same popular content (like a viral video) to millions of people? | The paper does not address this as they were not thinking about scaling the communication between networks. At a time now, we can think about how this is done, but their objective was to provide a foundation of the internet with networks communicating packets between each other | We see this today with applications like Instagram hosting content that can handle a large influx of traffic. They have their own data centers/servers to cache and mitigate the content from being too overwhelming, allowing it to reach thousands of people around the same time |
| How does data find its way from Network A through Network B to reach Network C? | The paper addresses this in 'The Gateway Notion' section where it states that data can find its way from Network A through Network B to Network C with two gateways, M and N. M connects Network A to B, and N connects Network B to C. The data from Network A will go through Gateway M, conforming to B's requirements, then after traversing B, it takes gateway N, conforming to C's requirements, and finally reaching Network C. | We can see this today through a lot of devices. Thinking of something like an at home door sensor, network A, that must send data to a router, network B, and then send it to an external cloud platform from the company they purchased it from, so they can then notify you from their application. |
| When you visit amazon.com, how do you know you're really connected to Amazon and not an imposter site? | The paper does not address the idea of security because it likely wasn't a concern that people would run into. Even still, I imagine it is up to the user's due diligence and not a networking problem that felt like it needed to be addressed. | We see this today all the time with random phishing attempts. While there is some regulation, it is up to the user to check the web address and ensure that they are in the correct place. An O replaced with a 0 might be something that people miss, would be something to pay attention to.  |
| How do you prevent bad actors from flooding the network with unwanted traffic? | Since they were addressing the most important parts of networking and the internet, I don't think this was something they had thought about (or really bad actors at all). While they understand the stress that these packet switches and communication between networks can cause, they are more interested in addressing how to avoid flooding networks (such as with the gateway fragmentation) as opposed to thinking about it where networks would need protection from people trying to flood a network. | We see this today with extra layers of security on networks. While someone's at home network may not be prepared to handle an attack, large companies invest a lot into security to ensure that their networks are not affected by bad actors. |
| What happens when Network A supports 1000-byte packets but Network B only supports 500-byte packets? | In the 'Process Level Communication' section of the paper, it is stated that we allow the transmission control program (TCP) of the network to break up messages due to size restrictions or limitations, typically done in the gateway between the two networks. These broken up pieces would have a sequence number that pointing to the next packet for reassembly. | This is something that is still seen today, but typically blocked. IPv6 doesn't allow for fragmentation, while IPv4 defaults to not fragmenting. If this is the case, it now will drop the packet and let the sender know, but if it allows for fragmentation it will fragment the packet and follow a similar protocol as the one described in the paper.  |
| How do you create addresses that work across different network types? | In the paper, the solution proposed is on having the TCP handle the communication with transmit and receive control blocks. Additionally, they propose a uniform labeling for source and destination addresses with the full net/HOST/TCP/port address of the transmitter and receiver. | In the modern day, we still use the TCP concept introduced in the paper, but we also now have an additional IP address to connect everything between networks. So in the modern day we have IPv4 and IPv6 to define how we move these packets and communicate between networks.  |

# Part 2

## A. Investigation Overview

Problem chosen from table: Row 7, What happens when Network A supports 1000-byte packets but Network B only supports 500-byte packets?

Specific scenario: The case that I am going to be investigating is the difference between my VPN loading slowly when loading sites, while it loads perfectly fine without it. My understanding is that it could be sending extra packets with the VPN call, so I want to learn more about what is happening. 


## B. Key Questions You Asked

Why do some websites hang forever when I'm on a VPN, while others load fine? How does this connect to packets being too big for part of the path and how does this relate to the the concepts introduced in the 1974 paper 'A Protocol for Packet Network Intercommunication'

What is the purpose of the extra bytes for the VPN? Does it have to do with the fact that it is hiding information from the receiver?

If "Most of the extra bytes exist to get the packet delivered and to let the VPN server check it", then why do I get stuck on infinite loading occasionally? If the website is rejecting the request from the DF bit, wouldn't the VPN know and stop the request?

With the inclusion of the DF bit, is the communication still be done in a similar way to the concepts introduced in the Cerf and Kahn design, or does this change the way their original ideas operated?


## C. Connection to 1974 Paper



## D. Surprise Finding


## E. AI Conversation Documentation

# Conversation: VPNs, MTU, and Cerf & Kahn (1974)

*September 29, 2026*

---

## User

Why do some websites hang forever when I'm on a VPN, while others load fine? How does this connect to packets being too big for part of the path and how does this relate to the the concepts introduced in the 1974 paper 'A Protocol for Packet Network Intercommunication'

## Claude

### Why some sites hang on a VPN

A VPN wraps each of your packets inside another packet: a new outer IP header, the tunnel protocol's own header, and often encryption padding and an authentication tag. That overhead is typically 50–80 bytes, depending on WireGuard, IPsec or OpenVPN. Most Ethernet and home-broadband links carry at most 1500 bytes per packet, their **MTU** (maximum transmission unit). If your computer sends a full 1500-byte packet into the tunnel, the wrapped version becomes roughly 1560 bytes and no longer fits somewhere along the path.

The symptom is oddly specific, and that is the clue:

1. **The connection opens fine.** The TCP handshake (SYN, SYN-ACK, ACK) uses tiny packets, so the site appears to be responding.
2. **Then it stalls.** The first large packets are usually the server's TLS certificate chain or the first chunk of the page. These are full-size and get dropped at the too-small link.
3. **Nothing tells anyone.** TCP retransmits the same oversized packet, it gets dropped again, and the browser sits spinning.

### Why the sender never learns: Path MTU Discovery

Modern TCP stacks set the "Don't Fragment" (DF) bit on outgoing packets. When a router gets a DF packet that's too big for the next link, it's supposed to drop it and send back an ICMP message: "Fragmentation Needed, next-hop MTU is X" in IPv4, or "Packet Too Big" in IPv6. The sender then shrinks its packets. This is **Path MTU Discovery** (RFC 1191 for IPv4, RFC 8201 for IPv6).

It breaks when a firewall somewhere blocks all ICMP, often because an administrator thinks of ICMP as "just ping" and a security risk. The oversized packet vanishes silently. This is called a **PMTUD black hole**.

### Why other sites load fine

Several things can make a site immune:

- **Small responses.** If everything fits in smaller packets, the limit never matters.
- **MSS clamping.** Many VPN gateways and servers rewrite the TCP "maximum segment size" advertised during the handshake. Both sides then agree up front to send smaller segments, and the problem never arises. Well-run VPNs do this, and when it's missing on one path but present on another, you see exactly this "some sites" pattern.
- **Conservative servers.** Big CDNs often use a slightly lower MSS or probe for the right size themselves (Packetization Layer PMTUD, RFC 4821), so they don't depend on ICMP.
- **QUIC/HTTP/3.** It deliberately starts with packets around 1200 bytes, which fit through almost any tunnel.
- **Different routes.** The site's traffic might simply not cross the filtering firewall.

If you control the setup, the usual fixes are:
- Lower the MTU on the VPN interface (for example, 1420 for WireGuard, lower for some IPsec setups).
- Enable MSS clamping on the VPN gateway.
- Allow ICMP "fragmentation needed" and "packet too big" messages through firewalls.

### The connection to Cerf and Kahn (1974)

Vint Cerf and Bob Kahn's "A Protocol for Packet Network Intercommunication" (IEEE Transactions on Communications, May 1974) was about joining networks that didn't match. One of the explicit mismatches was **maximum packet size**. A packet valid on one network might be too big for the next.

Their answer had a few parts that still shape the problem:

- **Gateways fragment, destinations reassemble.** A gateway that received a packet too large for the next network could split it. Reassembly happened only at the destination host, never at intermediate gateways, which kept gateways simple.
- **Sequence numbers count bytes, not packets.** Because each byte had its own sequence position, data could be split and still be put back in order and acknowledged correctly at the far end. This byte-oriented numbering survives in TCP today.
- **Endpoints own reliability.** The network made a best effort, and hosts handled retransmission and ordering. This early form of what became the end-to-end principle is why your TCP stack, not the router, retries the lost packet.

Their single protocol was later split in two. IP (RFC 791, 1981) took over fragmentation, with the identification, fragment-offset and "more fragments" fields, and TCP handled reliability.

In-network fragmentation turned out to have real costs. Losing one fragment meant losing the whole packet, reassembly consumed receiver resources, and fragments caused trouble for firewalls. Kent and Mogul's 1987 paper "Fragmentation Considered Harmful" made the case. The response was to set DF and let endpoints discover the path limit through ICMP feedback. IPv6 went further and removed router fragmentation entirely.

So the VPN hang is a consequence of that historical shift. Cerf and Kahn's design had the network absorb size mismatches through gateway fragmentation. Today's internet makes endpoints responsible for avoiding them, which is consistent with their end-to-end philosophy. But that approach depends on a feedback signal that intermediate boxes are free to drop. When a VPN adds a new "network" inside the path (the tunnel, with its smaller effective MTU) and a firewall eats the ICMP, you get exactly the heterogeneous-network size problem they set out to solve, with neither the old fix (fragmentation) nor the new one (discovery) working.

---

## User

What is the purpose of the extra bytes for the VPN? Does it have to do with the fact that it is hiding information from the receiver?

## Claude

Partly, but the "hiding" is mostly from everyone *between* you and the VPN server, not from the site you're visiting. Most of the extra bytes exist to get the packet delivered and to let the VPN server check it, not to conceal anything.

### What the packet looks like

Your original packet is addressed from your computer to, say, a website. The VPN encrypts that whole packet, header included, and puts it inside a new packet addressed from your computer to the VPN server. Routers on the way can only read the outer envelope, so they see traffic going to the VPN server and nothing else.

WireGuard is a clean example, with 60 bytes of overhead over IPv4:

- **Outer IP header (20 bytes; 40 over IPv6):** New source and destination addresses so the internet can route the packet to the VPN server. This is the biggest piece, and it's purely about delivery.
- **UDP header (8 bytes):** Ports, so the packet reaches the right program on the VPN server and gets through home routers and NAT.
- **WireGuard header (16 bytes):** A message type, an index telling the server which session (and therefore which keys) this packet belongs to, and a counter. The counter serves as the encryption nonce and lets the server reject replayed copies of old packets.
- **Authentication tag (16 bytes):** A cryptographic checksum proving the packet came from someone holding the key and wasn't altered in transit. Without it, an attacker could flip bits in the encrypted data undetected.

IPsec and OpenVPN have similar pieces, plus sometimes padding, because their block-mode ciphers need data in fixed-size chunks.

The encryption itself usually adds almost nothing. Modern ciphers like ChaCha20 and AES-GCM produce ciphertext the same length as the input. The overhead is the envelope and the metadata needed to decrypt and verify it.

### Who is hidden from whom

- **Your ISP, the café Wi-Fi, and anyone on that path** see only encrypted packets going to the VPN server. They can't see which sites you visit or what you send. This is the privacy the encryption buys.
- **The VPN server** decrypts everything, strips the envelope, and forwards your original packet to the website, so it sees everything the ISP used to see. You're shifting trust to the VPN provider, not eliminating it.
- **The website** receives a normal, unencrypted-by-the-VPN packet (still protected by HTTPS if the site uses it). It sees the VPN server's IP address instead of yours. That's the only sense in which the receiver is kept in the dark: it doesn't learn where you really are. None of the extra bytes accomplish that; it happens because the VPN server re-sends the traffic under its own address.

### Tie-back to the MTU problem

The outer header and tag are the bytes that push a 1500-byte packet over the limit. Because the original packet, header included, is encrypted inside the tunnel, routers along the path can't see or adjust its TCP settings. That's why the fix has to happen at the tunnel's ends: the VPN endpoint clamps the MSS or lowers the interface MTU, since it's the only place that can read and adjust the inner packet.

---

## User

If "Most of the extra bytes exist to get the packet delivered and to let the VPN server check it", then why do I get stuck on infinite loading occasionally? If the website is rejecting the request from the DF bit, wouldn't the VPN know and stop the request?

## Claude

The key correction is that nobody *rejects* the request, and the website is usually not where the problem happens. A router somewhere in the middle quietly throws away a packet that is too big. Neither the website nor your VPN is necessarily in a position to see that happen, and nothing on the network is obliged to tell them.

### The website isn't refusing anything

The DF bit is not a request the website evaluates. It's an instruction to routers: "don't split this packet; if it doesn't fit, drop it." The drop happens at whichever router has the too-small outgoing link, and the only signal is that ICMP "too big" message sent back toward whoever sent the packet. If that message is blocked, lost, or goes to the wrong party, the drop is completely silent.

Also, the packet that gets stuck is usually heading *toward* you, not away from you. Your request is small and gets through. The website's reply (certificate, page data) comes in full 1500-byte packets, and those are the ones that don't fit.

### Why the VPN doesn't know

There are two common places the drop happens.

**1. At the VPN server itself.** The website sends a 1500-byte packet toward you. It reaches the VPN server, which needs to wrap it and finds the result is too big to send on.
- The VPN server is supposed to drop it and send the ICMP message back to the *website*, since the website is the sender that needs to shrink its packets.
- If the website's firewall or load balancer blocks incoming ICMP, as many do, the website never learns. It keeps resending the same oversized packet, and the VPN server keeps dropping it.
- In this case the VPN *does* know, but it can only report the problem, not fix the website's behavior, unless it clamps the MSS in advance.

**2. Somewhere after wrapping.** The VPN server sends the encrypted packet, and some link further along (a DSL line with PPPoE, a mobile network) has a smaller limit than expected.
- That router sees only the outer envelope. It sends its ICMP message to the envelope's sender, the VPN server, not the website.
- The VPN server then has to connect that message to the original inner packet and pass the lesson back to the website. Some VPN implementations do this well, some poorly, and the ICMP may be filtered before it even reaches the VPN.

In the phrase you quoted, "let the VPN server check it" meant verifying packets that arrive. A packet dropped on the way never arrives, so there's nothing to check and no error to raise.

### Why it hangs instead of failing

This goes back to the Cerf and Kahn design: the network makes a best effort, and loss is normal and unannounced. From TCP's point of view, a silently dropped packet looks exactly like ordinary congestion loss, so it does what it's designed to do:

1. Wait, then retransmit the same packet.
2. When that is dropped too, wait roughly twice as long and try again.
3. Repeat. On Linux, the default is to keep trying for about 15 minutes before giving up.

Every retry is the same oversized packet, so every retry fails. The connection is technically alive, just stuck, which the browser shows as endless loading. Browsers do have their own timeouts, but they're long for exactly this reason: an unannounced drop and a slow network look the same.

### Why only occasionally

It depends on whether a full-size packet is ever sent on that path:

- **Response size.** Small replies fit and never trigger the problem.
- **MSS clamping.** If the VPN or the site's network clamps the segment size during the handshake, every packet is small from the start.
- **Protocol.** HTTP/3 uses small packets by default, so a site using it may load while a site on older HTTP hangs.
- **Firewall policy.** Sites that let ICMP through get the message, shrink their packets after one lost round, and load with just a brief delay.
- **Route.** Different sites are reached through different routers and links, some with the small limit or ICMP filter and some without.

So "occasionally" is really "whenever a large packet meets a too-small link and the warning message can't get back to the sender."

---

## User

With the inclusion of the DF bit, is the communication still be done in a similar way to the concepts introduced in the Cerf and Kahn design, or does this change the way their original ideas operated

## Claude

Both. The DF bit keeps Cerf and Kahn's overall model intact but changes one specific part of it: **who deals with a packet that's too big**. Arguably the change pushes their philosophy further than they took it themselves, at the cost of a dependency their design didn't have.

### What stays the same

Almost everything fundamental survives:

- **Best-effort datagrams.** Routers still forward each packet independently and may drop it without notice. That's exactly the gateway role they described.
- **Simple gateways.** Routers still don't track connections or guarantee delivery.
- **Endpoints own reliability.** TCP on the hosts still detects loss, retransmits and reorders. That's why the hang we discussed looks like ordinary loss to TCP.
- **Byte-based sequence numbers.** Still how TCP tracks data, which is what lets it resend the same bytes in a smaller packet once it learns the right size.

### What changes

In the 1974 design, a size mismatch was handled **inside the network**. A gateway facing a smaller network split the packet, and the destination put it back together. The sender never had to know the path had a smaller limit. Data got through regardless, just in pieces.

With DF set, the network **refuses** to split. The gateway drops the packet and, ideally, reports back. The sender must learn the path's limit and send packets that fit from the start. Fragmentation moves from a network service to a sender responsibility.

A bit of history: DF arrived with IP in 1981 (RFC 791), originally for hosts that couldn't reassemble fragments. Path MTU Discovery repurposed it in 1990 as a probing tool, turning "don't split this" into "tell me if this is too big." IPv6 then made the shift permanent: routers never fragment, and only the original sender may.

### Is that a departure or a continuation?

It can be read either way.

**It's a continuation** in that Cerf and Kahn's instinct was to keep gateways simple and put intelligence in the hosts. Having routers split and track fragments was work in the middle of the network, and it performed badly in practice: one lost fragment wasted the whole packet, and receivers had to hold partial packets in memory. Moving the job to the endpoints is more faithful to the end-to-end idea than their own fragmentation scheme was.

**It's a departure** in two ways:

- **It depends on feedback.** The 1974 approach worked without any message getting back to the sender, since gateways just split and forwarded. The DF approach only works if the ICMP "too big" message makes it back. That's a new point of failure, and it's exactly the one VPN black holes expose. Their design was robust to lost signals; this one isn't.
- **It assumes a transparent network.** Cerf and Kahn pictured a network of cooperating gateways. Today's internet is full of firewalls, NAT boxes and tunnels that filter or rewrite traffic, and the DF scheme is fragile when those boxes don't cooperate.

### The newest step

Recent fixes move even further toward the endpoint. Packetization Layer PMTUD (RFC 4821, and RFC 8899 for datagram protocols like QUIC) has the sender probe with packets of increasing size and watch what gets acknowledged, not rely on ICMP at all. That removes the dependency on the network's cooperation. It uses only the tools Cerf and Kahn gave the endpoints: sequence numbers, acknowledgments and retransmission.

So the arc runs: the network splits packets (1974), then the network tells the sender (1990), then the sender works it out alone (2007 onward). Each step keeps their architecture while trusting the middle of the network less.

# Part 3



