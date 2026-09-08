# SBT-DF203 Lab 1 — HTTP Analysis Using Wireshark: Text Traffic

## Overview

This repository documents my practical work for **SBT-DF203: Basic Networking Skills for Digital Forensics — Lab 1: HTTP Analysis Using Wireshark: Text Traffic**.

The exercise involved creating a controlled local HTTP webpage, capturing the resulting plaintext traffic, reconstructing the TCP conversation, identifying the HTTP request and response, examining TCP sequence and acknowledgement behaviour, reviewing IPv4/TCP/HTTP encapsulation, documenting connection termination, and preserving the packet capture with SHA-256 integrity verification.

> **Author:** Athanasius Alekwe  
> **Registration Number:** 2025/FWSD/11230  
> **Course:** SBT-DF203 — Basic Networking Skills for Digital Forensics  
> **Lab:** Lab 1 — HTTP Analysis Using Wireshark: Text Traffic  
> **Instructor:** Aminu Idris  
> **Submission Date:** 08 September 2026  
> **Case/Lab Identifier:** `SBT-DF203-Lab1-Athanasius-Alekwe`

---

## Objectives

The practical was completed to demonstrate the following network-forensic skills:

- Create and serve an authorised local HTTP webpage.
- Capture a complete plaintext HTTP session.
- Identify the TCP three-way handshake: SYN, SYN-ACK and ACK.
- Extract source and destination IP addresses and TCP ports.
- Record TCP sequence and acknowledgement numbers.
- Identify packet timestamps.
- Examine HTTP request and response headers.
- Reconstruct the complete application conversation using **Follow TCP Stream**.
- Explain HTTP/TCP/IPv4 encapsulation.
- Identify the TCP connection-termination sequence.
- Explain why loopback traffic does not provide meaningful physical Ethernet MAC-address evidence.
- Preserve the original PCAPNG evidence.
- Create and verify a forensic working copy.
- Calculate SHA-256 hashes for integrity verification.
- Produce a concise chronological forensic timeline.

---

## Laboratory Environment

| Component | Use |
|---|---|
| VMware Workstation | Virtual-machine platform |
| Kali Linux | Main analysis environment |
| Apache2 | Local web server |
| Firefox | Browser verification of the training page |
| curl 8.21.0 | Controlled HTTP request generation |
| TShark | Packet capture and command-line extraction |
| Wireshark | Packet analysis and TCP-stream reconstruction |
| sha256sum | Evidence-integrity verification |
| Linux loopback interface (`lo`) | Capture interface |
| TCP port 80 | Apache HTTP service |

---

## Authorised Training Page

The local Apache webpage was created as:

```text
/var/www/html/basic.html
```

The page contained:

```html
<!DOCTYPE html>
<html>
<body>
  <h1>SBT-DF203 HTTP Evidence</h1>
  <p>Name: Athanasius Alekwe</p>
  <p>Reg No: 2025/FWSD/11230</p>
</body>
</html>
```

It was accessed locally at:

```text
http://127.0.0.1/basic.html
```

---

## Repository Structure

```text
SBT-DF203-Lab1/
├── README.md
├── evidence/
│   └── basic.pcapng
├── working/
│   └── basic_working.pcapng
├── reports/
│   ├── capture_hashes.txt
│   ├── curl_verbose.txt
│   ├── handshake.tsv
│   ├── http_requests.tsv
│   ├── http_responses.tsv
│   ├── pre_preservation_sha256.txt
│   └── final_submission_zip_sha256.txt
├── screenshots/
│   └── ...
├── exported/
│   └── ...
└── SBT-DF203-Lab1_2025-FWSD-11230_Athanasius-Alekwe.pdf
```

### Evidence Handling

- `evidence/basic.pcapng` is the preserved original capture.
- `working/basic_working.pcapng` is the analysis copy.
- Analysis was performed on the working copy rather than the original evidence.
- The original evidence was later made read-only to reduce accidental modification.

---

## Acquisition Method

### 1. Prepare the laboratory folders

```bash
mkdir -p ~/SBT-DF203-Lab1/{evidence,working,exported,reports,screenshots,scripts}
cd ~/SBT-DF203-Lab1
```

### 2. Install the required tools

```bash
sudo apt update
sudo apt install -y apache2 curl wireshark tshark
sudo systemctl enable --now apache2
```

### 3. Verify Apache and TCP port 80

```bash
sudo systemctl status apache2 --no-pager
sudo ss -lntp | grep ':80'
```

### 4. Create the training webpage

```bash
printf '<!DOCTYPE html>\n<html><body><h1>SBT-DF203 HTTP Evidence</h1><p>Name: Athanasius Alekwe</p><p>Reg No: 2025/FWSD/11230</p></body></html>\n' | sudo tee /var/www/html/basic.html
```

### 5. Verify the HTTP request/response

```bash
curl -v http://127.0.0.1/basic.html 2>&1 | tee reports/curl_verbose.txt
```

### 6. Capture the HTTP session

The first attempt to write the TShark capture directly into the evidence directory produced a permission error on the laboratory VM.

The successful capture was therefore written temporarily to `/tmp`:

```bash
sudo tshark -i lo -f 'tcp port 80' -w /tmp/basic.pcapng
```

While TShark was running, the controlled request was generated:

```bash
curl --no-keepalive -v http://127.0.0.1/basic.html
```

The capture was then stopped with `Ctrl+C`.

The completed session contained **10 packets**.

### 7. Preserve the evidence

```bash
sudo sha256sum /tmp/basic.pcapng
sudo mv /tmp/basic.pcapng ~/SBT-DF203-Lab1/evidence/basic.pcapng
sudo chown kali:kali ~/SBT-DF203-Lab1/evidence/basic.pcapng
sha256sum evidence/basic.pcapng
```

A timestamp-preserving working copy was created:

```bash
cp --preserve=timestamps evidence/basic.pcapng working/basic_working.pcapng
```

Both files were then hashed:

```bash
sha256sum evidence/basic.pcapng working/basic_working.pcapng | tee reports/capture_hashes.txt
```

---

## Evidence Integrity

### SHA-256

```text
5bf443700a4ec790002331428012c71193d6bbc83992e720f5b100a6bcef8c1f
```

The same SHA-256 value was recorded for:

```text
evidence/basic.pcapng
working/basic_working.pcapng
```

This confirmed that the forensic working copy was bit-for-bit identical to the preserved original at the time of verification.

---

## Key Forensic Findings

| Finding | Verified Value |
|---|---|
| Evidence file | `basic.pcapng` |
| Working copy | `basic_working.pcapng` |
| Packets captured | 10 |
| First packet timestamp | 8 Sep 2026 19:11:51.347894911 WAT |
| Last packet timestamp | 8 Sep 2026 19:11:51.353054816 WAT |
| Client IP | `127.0.0.1` |
| Client port | `57098` |
| Server IP | `127.0.0.1` |
| Server port | `80` |
| Client raw ISN | `2286643435` |
| Server raw ISN | `4118857147` |
| HTTP request frame | 4 |
| HTTP request time | 19:11:51.348032929 WAT |
| Method | `GET` |
| Requested URI | `/basic.html` |
| Host | `127.0.0.1` |
| User-Agent | `curl/8.21.0` |
| HTTP response frame | 6 |
| Response time | 19:11:51.352610501 WAT |
| HTTP status | `200 OK` |
| Server software | `Apache/2.4.68 (Debian)` |
| Content-Type | `text/html` |
| Content-Length | `135 bytes` |
| Termination | Graceful FIN/ACK closure |
| MAC observation | Loopback capture; no meaningful physical Ethernet MAC evidence |

---

## TCP Three-Way Handshake

| Frame | Time (WAT) | Source | Destination | Flags | Relative Seq | Relative Ack |
|---:|---|---|---|---|---:|---:|
| 1 | 19:11:51.347894911 | `127.0.0.1:57098` | `127.0.0.1:80` | SYN | 0 | 0 |
| 2 | 19:11:51.347913288 | `127.0.0.1:80` | `127.0.0.1:57098` | SYN, ACK | 0 | 1 |
| 3 | 19:11:51.347927529 | `127.0.0.1:57098` | `127.0.0.1:80` | ACK | 1 | 1 |

### Raw Initial Sequence Numbers

```text
Client raw ISN: 2286643435
Server raw ISN: 4118857147
```

The SYN-ACK acknowledged `2286643436`, which is one greater than the client ISN because a TCP SYN consumes one sequence number.

---

## HTTP Request Analysis

Frame 4 contained:

```http
GET /basic.html HTTP/1.1
Host: 127.0.0.1
User-Agent: curl/8.21.0
Accept: */*
```

| Field | Value |
|---|---|
| Frame | 4 |
| Timestamp | 8 Sep 2026 19:11:51.348032929 WAT |
| Source | `127.0.0.1:57098` |
| Destination | `127.0.0.1:80` |
| Method | GET |
| URI | `/basic.html` |
| Host | `127.0.0.1` |
| User-Agent | `curl/8.21.0` |

---

## HTTP Response Analysis

Frame 6 contained:

```http
HTTP/1.1 200 OK
Server: Apache/2.4.68 (Debian)
Content-Type: text/html
Content-Length: 135
```

| Field | Value |
|---|---|
| Frame | 6 |
| Timestamp | 8 Sep 2026 19:11:51.352610501 WAT |
| Status | `200 OK` |
| Server | `Apache/2.4.68 (Debian)` |
| Content-Type | `text/html` |
| Content-Length | `135 bytes` |

---

## TCP Stream Reconstruction

Wireshark's **Follow TCP Stream** function reconstructed the complete application-layer conversation.

The stream showed:

1. The client requesting `/basic.html`.
2. The Host header `127.0.0.1`.
3. The User-Agent `curl/8.21.0`.
4. The Apache `HTTP/1.1 200 OK` response.
5. `Content-Type: text/html`.
6. `Content-Length: 135`.
7. The returned HTML page containing the trainee identification.

Wireshark displayed the entire reconstructed conversation as **469 bytes**.

---

## TCP Data Progression

### Client request

```text
Frame 4:
Sequence = 1
TCP payload = 83 bytes
Next expected sequence = 84
```

Frame 5 acknowledged:

```text
Ack = 84
```

### Server response

```text
Frame 6:
Sequence = 1
TCP payload = 386 bytes
Next expected sequence = 387
```

Frame 7 acknowledged:

```text
Ack = 387
```

The HTTP `Content-Length` was 135 bytes because it refers only to the HTTP message body. The larger TCP payload also contained the HTTP response headers.

---

## TCP Connection Termination

| Frame | Direction | Flags | Relative Seq | Relative Ack | TCP Len |
|---:|---|---|---:|---:|---:|
| 8 | `127.0.0.1:57098 → 127.0.0.1:80` | FIN, ACK | 84 | 387 | 0 |
| 9 | `127.0.0.1:80 → 127.0.0.1:57098` | FIN, ACK | 387 | 85 | 0 |
| 10 | `127.0.0.1:57098 → 127.0.0.1:80` | ACK | 85 | 388 | 0 |

No TCP reset was observed.

---

## Encapsulation Analysis

| Layer / Field | Value |
|---|---:|
| Frame length | 149 bytes |
| Link-layer header shown by Wireshark | 14 bytes |
| IPv4 total length | 135 bytes |
| IPv4 header length | 20 bytes |
| TCP header length | 32 bytes |
| TCP payload length | 83 bytes |
| TCP source port | 57098 |
| TCP destination port | 80 |
| Relative TCP Seq | 1 |
| Relative TCP Ack | 1 |
| HTTP request | `GET /basic.html HTTP/1.1` |

Relationship:

```text
IPv4 Total Length
= IPv4 Header + TCP Header + TCP Payload

135
= 20 + 32 + 83
```

Captured frame:

```text
149 bytes
= 14-byte link-layer representation + 135-byte IPv4 packet
```

---

## Loopback and MAC-Address Limitation

The session was captured on the Linux loopback interface:

```text
lo
```

Both logical endpoints were `127.0.0.1`.

Wireshark displayed all-zero link-layer addresses in the loopback capture. These should **not** be interpreted as real physical Ethernet MAC addresses.

Because the traffic never traversed a conventional Ethernet network, meaningful source and destination MAC-address evidence was unavailable.

---

## Chronological Forensic Timeline

| Frame | Time (WAT) | Event | Interpretation |
|---:|---|---|---|
| 1 | 19:11:51.347894911 | SYN | Client port 57098 requests connection to TCP 80 |
| 2 | 19:11:51.347913288 | SYN, ACK | Apache acknowledges and supplies server ISN |
| 3 | 19:11:51.347927529 | ACK | TCP connection established |
| 4 | 19:11:51.348032929 | HTTP GET | Client requests `/basic.html` |
| 5 | 19:11:51.348076937 | ACK | Server acknowledges the 83-byte request payload |
| 6 | 19:11:51.352610501 | HTTP 200 OK | Apache sends response headers and HTML content |
| 7 | 19:11:51.352644844 | ACK | Client acknowledges server payload |
| 8 | 19:11:51.352955431 | FIN, ACK | Client initiates graceful closure |
| 9 | 19:11:51.353032500 | FIN, ACK | Server closes its side |
| 10 | 19:11:51.353054816 | ACK | Client completes connection termination |

---

## Useful TShark Commands

### Handshake

```bash
tshark -r working/basic_working.pcapng -Y 'tcp.stream eq 0 && tcp.flags.syn==1' -T fields -e frame.number -e frame.time -e ip.src -e tcp.srcport -e ip.dst -e tcp.dstport -e tcp.flags -e tcp.seq -e tcp.ack | tee reports/handshake.tsv
```

### HTTP request

```bash
tshark -r working/basic_working.pcapng -Y 'http.request' -T fields -e frame.number -e frame.time -e ip.src -e tcp.srcport -e ip.dst -e tcp.dstport -e http.request.method -e http.request.uri -e http.host -e http.user_agent | tee reports/http_requests.tsv
```

### HTTP response

```bash
tshark -r working/basic_working.pcapng -Y 'http.response' -T fields -e frame.number -e frame.time -e http.response.code -e http.response.phrase -e http.server -e http.content_type -e http.content_length | tee reports/http_responses.tsv
```

### Connection closure

```bash
tshark -r working/basic_working.pcapng -Y 'tcp.flags.fin==1 || tcp.flags.reset==1' -T fields -e frame.number -e frame.time -e ip.src -e tcp.srcport -e ip.dst -e tcp.dstport -e tcp.flags -e tcp.seq -e tcp.ack
```

### Integrity verification

```bash
sha256sum evidence/basic.pcapng working/basic_working.pcapng
```

---

## Wireshark Filters Used

```text
tcp.stream eq 0
http.request
http.response
tcp.stream eq 0 && frame.number <= 3
tcp.stream eq 0 && frame.number >= 8
frame.number == 4
frame.number == 2
```

---

## Screenshot Evidence

The full PDF report contains the practical screenshots. Key screenshots include:

1. Laboratory folder structure and start time.
2. Apache service verification.
3. TCP port 80 verification.
4. Local training webpage with trainee identification.
5. TShark capture running on `lo`.
6. TCP three-way handshake.
7. HTTP GET request.
8. HTTP `200 OK` response.
9. Follow TCP Stream reconstruction.
10. FIN/ACK connection termination.
11. Encapsulation analysis.
12. Raw TCP sequence-number verification.
13. SHA-256 evidence-hash verification.

Separate image files, when included, should be stored under:

```text
screenshots/
```

---

## Forensic Value of the Protocol Layers

### HTTP
HTTP exposed the request method, URI, Host, User-Agent, response status, server software, content type, content length and plaintext page content.

### TCP
TCP provided the source/destination ports, connection state, sequence numbers, acknowledgement numbers, payload progression and connection termination.

### IPv4
IPv4 identified the logical source and destination addresses of the communication.

### Link Layer
The link layer would normally provide local-network addressing information. In this loopback capture, however, meaningful physical Ethernet MAC addresses were not available.

---

## Limitations

- The traffic was generated entirely on the local host.
- Both logical endpoints used `127.0.0.1`.
- The capture was performed on the loopback interface.
- Meaningful physical Ethernet MAC addresses were unavailable.
- Only one short authorised HTTP session was examined.
- The exercise focused on plaintext HTTP rather than encrypted HTTPS.
- The findings apply only to this controlled training session.

---

## Conclusion

This exercise successfully demonstrated the acquisition and forensic examination of a complete plaintext HTTP session.

The packet capture contained the full TCP lifecycle: connection establishment, HTTP request, successful HTTP response, acknowledgement progression and graceful connection termination. The analysis also demonstrated how HTTP data is transported within TCP, encapsulated in IPv4, and represented at the capture link layer.

The original PCAPNG was preserved separately from the forensic working copy. Matching SHA-256 values confirmed that both files were identical at the time of verification.

---

## Legal, Ethical and Academic Use

This repository documents an **authorised laboratory exercise only**.

No third-party systems, public services, private communications, credentials or production network traffic were targeted or intercepted.

The repository is intended for educational and digital-forensics training purposes.

---

## Reference

- International Cybersecurity and Digital Forensics Academy (ICDFA)
- SBT-DF203 — Basic Networking Skills for Digital Forensics
- Lab 1 — HTTP Analysis Using Wireshark: Text Traffic
- Instructor: Aminu Idris

---

## Author

**Athanasius Alekwe**  
Registration Number: **2025/FWSD/11230**
