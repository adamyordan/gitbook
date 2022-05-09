# Analyzing HTTPS SSL/TLS traffic

![](https://1.bp.blogspot.com/-W1cciUHElT8/XO0ybRzvnOI/AAAAAAAAGYY/A_GCIDd1ItYCbcHba8fn55Ly49ceoCdfQCLcBGAs/s1600/Screenshot%2B2019-05-28%2Bat%2B8.43.41%2BPM.png)

Use wireshark to capture network packet. Use Curl to communicate to https://adamjordan.id \(172.217.194.121, port 443\)

```text
$ curl https://adamjordan.id
```

#### 1. Client -&gt; Server: Client Hello

Client connect to server and initiate the SSL Handshake negotiation. Client send "Client Hello" message. It contains client-supported cipher suites, compression methods, and extensions.

```text
No  Time        Source          Destination     Protocol Length Info
34 2.524442 10.2.210.183 172.217.194.121 TLSv1.2  290 Client Hello

0000   84 b8 02 66 87 5c 8c 85 90 32 c5 fc 08 00 45 00  ...f.\...2....E.
0010   01 14 00 00 40 00 40 06 ed d7 0a 02 d2 b7 ac d9  ....@.@.........
0020   c2 79 dc 9e 01 bb 86 84 7a af 8a 86 7a 40 80 18  .y......z...z@..
0030   08 04 c2 f1 00 00 01 01 08 0a 2d 5b 6f 8d 29 bb  ..........-[o.).
0040   65 53 16 03 01 00 db 01 00 00 d7 03 03 3d 62 84  eS...........=b.
0050   77 23 2f 13 f1 2a 46 59 d5 2c 5f 9d fa b9 f4 63  w#/..*FY.,_....c
0060   92 cb 44 4b 5d 3a 87 1c bc dc fd 07 c5 00 00 54  ..DK]:.........T
0070   cc a9 cc a8 cc aa c0 30 c0 2c c0 28 c0 24 c0 14  .......0.,.(.$..
0080   c0 0a 00 9f 00 6b 00 39 ff 85 00 c4 00 88 00 81  .....k.9........
0090   00 9d 00 3d 00 35 00 c0 00 84 c0 2f c0 2b c0 27  ...=.5...../.+.'
00a0   c0 23 c0 13 c0 09 00 9e 00 67 00 33 00 be 00 45  .#.......g.3...E
00b0   00 9c 00 3c 00 2f 00 ba 00 41 c0 12 c0 08 00 16  ...<./...A......
00c0   00 0a 00 ff 01 00 00 5a 00 00 00 12 00 10 00 00  .......Z........
00d0   0d 61 64 61 6d 6a 6f 72 64 61 6e 2e 69 64 00 0b  .adamjordan.id..
00e0   00 02 01 00 00 0a 00 08 00 06 00 1d 00 17 00 18  ................
00f0   00 0d 00 1c 00 1a 06 01 06 03 ef ef 05 01 05 03  ................
0100   04 01 04 03 ee ee ed ed 03 01 03 03 02 01 02 03  ................
0110   00 10 00 0e 00 0c 02 68 32 08 68 74 74 70 2f 31  .......h2.http/1
0120   2e 31                                            .1

Frame 34: 290 bytes on wire (2320 bits), 290 bytes captured (2320 bits) on interface 0
Ethernet II, Src: Apple_32:c5:fc (8c:85:90:32:c5:fc), Dst: Cisco_66:87:5c (84:b8:02:66:87:5c)
Internet Protocol Version 4, Src: 10.2.210.183, Dst: 172.217.194.121
Transmission Control Protocol, Src Port: 56478, Dst Port: 443, Seq: 1, Ack: 1, Len: 224
Secure Sockets Layer
    TLSv1.2 Record Layer: Handshake Protocol: Client Hello
        Content Type: Handshake (22)
        Version: TLS 1.0 (0x0301)
        Length: 219
        Handshake Protocol: Client Hello
            Handshake Type: Client Hello (1)
            Length: 215
            Version: TLS 1.2 (0x0303)
            Random: 3d628477232f13f12a4659d52c5f9dfab9f46392cb444b5d...
            Session ID Length: 0
            Cipher Suites Length: 84
            Cipher Suites (42 suites)
                Cipher Suite: TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca9)
                Cipher Suite: TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca8)
                Cipher Suite: TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xccaa)
                Cipher Suite: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)
                Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 (0xc02c)
                Cipher Suite: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 (0xc028)
                Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384 (0xc024)
                Cipher Suite: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (0xc014)
                Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA (0xc00a)
                Cipher Suite: TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 (0x009f)
                Cipher Suite: TLS_DHE_RSA_WITH_AES_256_CBC_SHA256 (0x006b)
            Compression Methods Length: 1
            Compression Methods (1 method)
            Extensions Length: 90
            Extension: server_name (len=18)
            Extension: ec_point_formats (len=2)
            Extension: supported_groups (len=8)
            Extension: signature_algorithms (len=28)
            Extension: application_layer_protocol_negotiation (len=14)
```

#### 2. Server -&gt; Client: Server Hello

As a response, Server send "Server Hello" message. It contains ONE \(1\) cipher suites, and 1 compression method. This cipher suites and compression method will be used along with this TLS session.

```text
No  Time        Source          Destination     Protocol Length Info
36 2.539060 172.217.194.121 10.2.210.183 TLSv1.2  1434 Server Hello

Frame 36: 1434 bytes on wire (11472 bits), 1434 bytes captured (11472 bits) on interface 0
Ethernet II, Src: Cisco_66:87:5c (84:b8:02:66:87:5c), Dst: Apple_32:c5:fc (8c:85:90:32:c5:fc)
Internet Protocol Version 4, Src: 172.217.194.121, Dst: 10.2.210.183
Transmission Control Protocol, Src Port: 443, Dst Port: 56478, Seq: 1, Ack: 225, Len: 1368
Secure Sockets Layer
    TLSv1.2 Record Layer: Handshake Protocol: Server Hello
        Content Type: Handshake (22)
        Version: TLS 1.2 (0x0303)
        Length: 96
        Handshake Protocol: Server Hello
            Handshake Type: Server Hello (2)
            Length: 92
            Version: TLS 1.2 (0x0303)
            Random: 5ced276d9d49a8aa1e5ed946f2db7aa3b9a47f92a6a279dd...
            Session ID Length: 32
            Session ID: 4d6ac64edbe1ae430bd6dd2a60e8bf7f2fb73da5719fe8af...
            Cipher Suite: TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca8)
            Compression Method: null (0)
            Extensions Length: 20
            Extension: renegotiation_info (len=1)
            Extension: ec_point_formats (len=2)
            Extension: application_layer_protocol_negotiation (len=5)
```

#### 3. Server -&gt; Client: Certificate

Server then send "Certificate" message. It contains server public key certificates chain.

```text
No  Time        Source          Destination     Protocol Length Info
37 2.539066 172.217.194.121 10.2.210.183 TLSv1.2  1434 Certificate [TCP segment of a reassembled PDU]

Frame 37: 1434 bytes on wire (11472 bits), 1434 bytes captured (11472 bits) on interface 0
Ethernet II, Src: Cisco_66:87:5c (84:b8:02:66:87:5c), Dst: Apple_32:c5:fc (8c:85:90:32:c5:fc)
Internet Protocol Version 4, Src: 172.217.194.121, Dst: 10.2.210.183
Transmission Control Protocol, Src Port: 443, Dst Port: 56478, Seq: 1369, Ack: 225, Len: 1368
[2 Reassembled TCP Segments (2558 bytes): #36(1267), #37(1291)]
Secure Sockets Layer
    TLSv1.2 Record Layer: Handshake Protocol: Certificate
        Content Type: Handshake (22)
        Version: TLS 1.2 (0x0303)
        Length: 2553
        Handshake Protocol: Certificate
            Handshake Type: Certificate (11)
            Length: 2549
            Certificates Length: 2546
            Certificates (2546 bytes)
                Certificate Length: 1366
                Certificate: 308205523082043aa003020102021203219097d39dd8bfb8... (id-at-commonName=adamjordan.id)
                    signedCertificate
                        version: v3 (2)
                        serialNumber: 0x03219097d39dd8bfb836615f73263593ea5f
                        signature (sha256WithRSAEncryption)
                        issuer: rdnSequence (0)
                            rdnSequence: 3 items (id-at-commonName=Let's Encrypt Authority X3,id-at-organizationName=Let's Encrypt,id-at-countryName=US)
                                RDNSequence item: 1 item (id-at-countryName=US)
                                RDNSequence item: 1 item (id-at-organizationName=Let's Encrypt)
                                RDNSequence item: 1 item (id-at-commonName=Let's Encrypt Authority X3)
                        validity
                            notBefore: utcTime (0)
                                utcTime: 19-04-22 21:39:43 (UTC)
                            notAfter: utcTime (0)
                                utcTime: 19-07-21 21:39:43 (UTC)
                        subject: rdnSequence (0)
                            rdnSequence: 1 item (id-at-commonName=adamjordan.id)
                        subjectPublicKeyInfo
                            algorithm (rsaEncryption)
                            subjectPublicKey: 3082010a0282010100cd71f5763a212f09e995c877caca7e...
                        extensions: 9 items
                    algorithmIdentifier (sha256WithRSAEncryption)
                    Padding: 0
                    encrypted: 5d04d4c03d8b322514d992cfa9021a7ebd695c5ff7d702f1...
                Certificate Length: 1174
                Certificate: 308204923082037aa00302010202100a0141420000015385... (id-at-commonName=Let's Encrypt Authority X3,id-at-organizationName=Let's Encrypt,id-at-countryName=US)
                    signedCertificate
                        version: v3 (2)
                        serialNumber: 0x0a0141420000015385736a0b85eca708
                        signature (sha256WithRSAEncryption)
                        issuer: rdnSequence (0)
                            rdnSequence: 2 items (id-at-commonName=DST Root CA X3,id-at-organizationName=Digital Signature Trust Co.)
                                RDNSequence item: 1 item (id-at-organizationName=Digital Signature Trust Co.)
                                RDNSequence item: 1 item (id-at-commonName=DST Root CA X3)
                        validity
                            notBefore: utcTime (0)
                                utcTime: 16-03-17 16:40:46 (UTC)
                            notAfter: utcTime (0)
                                utcTime: 21-03-17 16:40:46 (UTC)
                        subject: rdnSequence (0)
                            rdnSequence: 3 items (id-at-commonName=Let's Encrypt Authority X3,id-at-organizationName=Let's Encrypt,id-at-countryName=US)
                                RDNSequence item: 1 item (id-at-countryName=US)
                                RDNSequence item: 1 item (id-at-organizationName=Let's Encrypt)
                                RDNSequence item: 1 item (id-at-commonName=Let's Encrypt Authority X3)
                        subjectPublicKeyInfo
                            algorithm (rsaEncryption)
                            subjectPublicKey: 3082010a02820101009cd30cf05ae52e47b7725d3783b368...
                        extensions: 7 items
                    algorithmIdentifier (sha256WithRSAEncryption)
                    Padding: 0
                    encrypted: dd33d711f3635838dd1815fb0955be7656b97048a5694727...
```

#### 4. Server -&gt; Client: Server Key Exchange, Server Hello Done

Then server send "Server Key Exchange" message and "Server Hello Done" message. "Server Key Exchange" message contains the keys exchange algorithm for client and server to do symmetric encryption. "Server Hello Done" message marks that the server finishes its part of the handshake negotiation.

```text
No  Time        Source          Destination     Protocol Length Info
38 2.539068 172.217.194.121 10.2.210.183 TLSv1.2  303 Server Key Exchange, Server Hello Done

Frame 38: 303 bytes on wire (2424 bits), 303 bytes captured (2424 bits) on interface 0
Ethernet II, Src: Cisco_66:87:5c (84:b8:02:66:87:5c), Dst: Apple_32:c5:fc (8c:85:90:32:c5:fc)
Internet Protocol Version 4, Src: 172.217.194.121, Dst: 10.2.210.183
Transmission Control Protocol, Src Port: 443, Dst Port: 56478, Seq: 2737, Ack: 225, Len: 237
[2 Reassembled TCP Segments (305 bytes): #37(77), #38(228)]
Secure Sockets Layer
    TLSv1.2 Record Layer: Handshake Protocol: Server Key Exchange
        Content Type: Handshake (22)
        Version: TLS 1.2 (0x0303)
        Length: 300
        Handshake Protocol: Server Key Exchange
            Handshake Type: Server Key Exchange (12)
            Length: 296
            EC Diffie-Hellman Server Params
                Curve Type: named_curve (0x03)
                Named Curve: x25519 (0x001d)
                Pubkey Length: 32
                Pubkey: 81543ec207f0694fd8758e8e83b069b33c0564b91a1f1b58...
                Signature Algorithm: rsa_pkcs1_sha256 (0x0401)
                Signature Length: 256
                Signature: 9308d1566cf79524a14b5e59801bd04217a27e8df6f603fd...
Secure Sockets Layer
    TLSv1.2 Record Layer: Handshake Protocol: Server Hello Done
        Content Type: Handshake (22)
        Version: TLS 1.2 (0x0303)
        Length: 4
        Handshake Protocol: Server Hello Done
            Handshake Type: Server Hello Done (14)
            Length: 0
```

#### 5. Client -&gt; Server: Client Key Exchange, Change Cipher Spec, Finished

Client then responds by sending "Client Key Exchange", "Change Cipher Spec", and "Encrypted Handshake" message "Client Key Exchange" contains data for server to generate keys for the symmetric encryption. "Change Cipher Spec" message is sent in order to signal that the symmetric encryption has been activated. The "Encrypted Handshake Message" is an encrypted "Finished" message, which mark that handshake negotiation is finished for client part.

```text
No  Time        Source          Destination     Protocol Length Info
41 2.552896 10.2.210.183 172.217.194.121 TLSv1.2  151 Client Key Exchange, Change Cipher Spec, Encrypted Handshake Message

Frame 41: 151 bytes on wire (1208 bits), 151 bytes captured (1208 bits) on interface 0
Ethernet II, Src: Apple_32:c5:fc (8c:85:90:32:c5:fc), Dst: Cisco_66:87:5c (84:b8:02:66:87:5c)
Internet Protocol Version 4, Src: 10.2.210.183, Dst: 172.217.194.121
Transmission Control Protocol, Src Port: 56478, Dst Port: 443, Seq: 225, Ack: 2974, Len: 85
Secure Sockets Layer
    TLSv1.2 Record Layer: Handshake Protocol: Client Key Exchange
        Content Type: Handshake (22)
        Version: TLS 1.2 (0x0303)
        Length: 37
        Handshake Protocol: Client Key Exchange
            Handshake Type: Client Key Exchange (16)
            Length: 33
            EC Diffie-Hellman Client Params
                Pubkey Length: 32
                Pubkey: cf81666eddee37af4e8491dda30f5ab1f2c4b15410e7fd09...
    TLSv1.2 Record Layer: Change Cipher Spec Protocol: Change Cipher Spec
        Content Type: Change Cipher Spec (20)
        Version: TLS 1.2 (0x0303)
        Length: 1
        Change Cipher Spec Message
    TLSv1.2 Record Layer: Handshake Protocol: Encrypted Handshake Message
        Content Type: Handshake (22)
        Version: TLS 1.2 (0x0303)
        Length: 32
        Handshake Protocol: Encrypted Handshake Message
```

#### 6. Server -&gt; Client: Change Cipher Spec, Finished

Server then respond with similar message: "Change Cipher Spec" and encrypted "Finished" message. After this, the handshake process is considered successful.

```text
No  Time        Source          Destination     Protocol Length Info
42 2.561317 172.217.194.121 10.2.210.183 TLSv1.2  109 Change Cipher Spec, Encrypted Handshake Message

Frame 42: 109 bytes on wire (872 bits), 109 bytes captured (872 bits) on interface 0
Ethernet II, Src: Cisco_66:87:5c (84:b8:02:66:87:5c), Dst: Apple_32:c5:fc (8c:85:90:32:c5:fc)
Internet Protocol Version 4, Src: 172.217.194.121, Dst: 10.2.210.183
Transmission Control Protocol, Src Port: 443, Dst Port: 56478, Seq: 2974, Ack: 310, Len: 43
Secure Sockets Layer
    TLSv1.2 Record Layer: Change Cipher Spec Protocol: Change Cipher Spec
        Content Type: Change Cipher Spec (20)
        Version: TLS 1.2 (0x0303)
        Length: 1
        Change Cipher Spec Message
    TLSv1.2 Record Layer: Handshake Protocol: Encrypted Handshake Message
        Content Type: Handshake (22)
        Version: TLS 1.2 (0x0303)
        Length: 32
        Handshake Protocol: Encrypted Handshake Message
```

#### 7. Client &lt;-&gt; Server: Application Data

After TLS Handshake negotiation finished. Server and Client may communicate and send data in ecnrypted format.

```text
No  Time        Source          Destination     Protocol Length Info
44 2.562359 10.2.210.183 172.217.194.121 TLSv1.2  111 Application Data

0000   84 b8 02 66 87 5c 8c 85 90 32 c5 fc 08 00 45 00  ...f.\...2....E.
0010   00 61 00 00 40 00 40 06 ee 8a 0a 02 d2 b7 ac d9  .a..@.@.........
0020   c2 79 dc 9e 01 bb 86 84 7b e4 8a 86 86 08 80 18  .y......{.......
0030   08 00 c6 a2 00 00 01 01 08 0a 2d 5b 6f ae 29 bb  ..........-[o.).
0040   65 87 17 03 03 00 28 ab b4 ad 3e 27 36 e0 67 a1  e.....(...>'6.g.
0050   65 b8 b1 37 66 9c d3 96 10 05 77 88 83 57 bd 48  e..7f.....w..W.H
0060   b0 79 c1 96 c9 13 bc 96 29 e2 f2 cb 75 7b c7     .y......)...u{.


Frame 44: 111 bytes on wire (888 bits), 111 bytes captured (888 bits) on interface 0
Ethernet II, Src: Apple_32:c5:fc (8c:85:90:32:c5:fc), Dst: Cisco_66:87:5c (84:b8:02:66:87:5c)
Internet Protocol Version 4, Src: 10.2.210.183, Dst: 172.217.194.121
Transmission Control Protocol, Src Port: 56478, Dst Port: 443, Seq: 310, Ack: 3017, Len: 45
Secure Sockets Layer
    TLSv1.2 Record Layer: Application Data Protocol: http2
        Content Type: Application Data (23)
        Version: TLS 1.2 (0x0303)
        Length: 40
        Encrypted Application Data: abb4ad3e2736e067a165b8b137669cd396100577888357bd...
```

#### 8. Done

Done

