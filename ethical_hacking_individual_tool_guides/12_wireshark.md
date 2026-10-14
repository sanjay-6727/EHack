# Wireshark — Packet Capture and Protocol Analysis

## 1. Purpose
Wireshark captures and decodes network packets so you can inspect protocols, conversations, timing, and errors.

**Memory hook:** Wireshark = **slow-motion replay for network traffic**.

## 2. Inputs
- A network interface to capture from.
- Permission to capture.
- Optional capture filter (limits packets captured) and display filter (limits what is shown afterward).
- A test activity, such as pinging your own lab target.

## 3. GUI workflow
1. Open Wireshark.
2. Select the correct lab interface by checking its packet activity/IP.
3. Start capture.
4. Generate a small, authorized test, e.g. from Kali: `ping -c 4 TARGET_IP`.
5. Stop capture.
6. Use a display filter such as:
```text
icmp
dns
tcp
ip.addr == TARGET_IP
```
Capture-filter syntax differs from display-filter syntax; do not interchange them blindly.

## 4. What output means
- Packet list: time, source, destination, protocol, length, summary.
- Packet details: decoded fields for the selected packet.
- Packet bytes: raw packet data.
- ICMP echo request/reply pairs: evidence of a ping exchange.
- DNS query/response: name lookup exchange.
- TCP SYN/SYN-ACK/ACK: common steps in establishing a TCP connection.

## 5. Explain the output
“I captured packets on the lab interface and applied the `icmp` display filter. The request/reply pairs show the observed ping exchange. This proves those packets were visible on this interface, not that every network path is healthy.”

## 6. Troubleshooting
- No packets: wrong interface, capture permission, or no traffic.
- Filter shows nothing: syntax may be wrong or that protocol was not captured.
- Encrypted application traffic: packet metadata may be visible while payload content remains encrypted.
- Promiscuous mode does not magically reveal all switched network traffic.

## 7. Viva
**Q: Capture filter vs display filter?** Capture filter decides what to record; display filter decides what to show from the capture.
**Q: What is a packet?** A unit of network data at a protocol layer.
**Q: Why can't Wireshark read HTTPS contents normally?** TLS encrypts application data.
**Q: What does TCP SYN mean?** A request to start a TCP connection.
