
You can achieve Wireshark-grade packet filtering with TCPDump when you read and analyse PCAP files. *To read a saved capture file with default settings, use* `**tcpdump -r**` _filename_. For a *more verbose* output, the `**-v**`, `**-vv**` or `**-vvv**` options can be used, with each one increasing in verbosity. 

![[Pasted image 20240413130138.png]]

![[Pasted image 20240413015716.png]]
  
Controlling verbosity allows users to tailor the output of TCPDump to their specific needs. For quick network overviews, low verbosity may suffice, while high verbosity may be necessary for detailed troubleshooting and analysis. However, it's essential to strike a balance between verbosity and readability, as excessively verbose output can be challenging to interpret, especially in complex network environments.

Users can specify the verbosity level in TCPDump using command-line options. For example, the `-v` option enables medium verbosity, the `-vv` option enables high verbosity, and the `-vvv` option enables even higher verbosity levels, depending on the specific implementation of TCPDump.



In the first line of each TCP packet, IP header information is displayed, including the TTL, flags, Layer 4 protocol, length, and fragmentation offset. In the second line of each TCP packet, you can see the source IP address and port (may have been resolved to a name), the destination IP address and port, the TCP flag (**S** = SYN, **P** = PSH, **F** = FIN, **R** = RST, **U** = URG and **A** for = ACK), checksum, sequence and acknowledgement numbers, window size, any options and header length. 

TCP (Transmission Control Protocol) flags are used within TCP headers to control the connection and communication between devices over a network. TCPDump, being a packet analyzer, can interpret these flags to provide insights into network traffic. Here are the main TCP flags and their contributions to TCPDump:

1. **SYN (Synchronize):** This flag is used to initiate a connection between two devices. When a device wants to establish a connection with another device, it sends a TCP segment with the SYN flag set. In TCPDump, you can filter packets with the SYN flag to identify connection initiation attempts.

2. **ACK (Acknowledgment):** This flag is used to acknowledge received data. After the initial SYN packet, the receiving device responds with a packet containing the ACK flag set to acknowledge the receipt of the SYN packet. TCPDump can filter packets based on the ACK flag to identify acknowledgment packets.

3. **FIN (Finish):** This flag is used to terminate a connection. When a device wants to end a TCP connection, it sends a packet with the FIN flag set. TCPDump can capture these packets, allowing you to monitor connection terminations.

4. **RST (Reset):** This flag is used to reset a connection. It indicates that something unexpected happened, and the connection should be reset. TCPDump can capture packets with the RST flag set, helping you diagnose network issues or abnormal behavior.

5. **PSH (Push):** This flag is used to push data to the receiving application without waiting for more data to fill the TCP segment. TCPDump can capture packets with the PSH flag set, indicating data that should be immediately delivered to the application.

6. **URG (Urgent):** This flag indicates that the data in the TCP segment is urgent and should be prioritized. TCPDump can capture packets with the URG flag set, allowing you to identify urgent data within network traffic.

7. **CWR (Congestion Window Reduced):** This flag is used to indicate that the TCP sender has reduced its congestion window size in response to congestion in the network. TCPDump can capture packets with the CWR flag set, providing insights into congestion control mechanisms.

8. **ECE (Explicit Congestion Notification Echo):** This flag is used to indicate that the TCP sender received an Explicit Congestion Notification (ECN) message from the network. TCPDump can capture packets with the ECE flag set, helping you monitor congestion events in the network.

By examining these TCP flags in TCPDump output, network administrators and analysts can gain valuable insights into network behavior, troubleshoot issues, and monitor network performance.

You can also apply filters specifying particular fields and values in the packet header. Some of the most useful ones include:

 – **Specifying TCP flags**:  
To match all packets with a single specified TCP flag, add the **tcp[tcpflags] == tcp-ack**/**tcp-syn**/**tcp-fin**/**tcp-push**/**tcp-urg**/**tcp-rst** option. For example, **tcp[tcpflags] == tcp-push** displays packets with only the PSH flag set.

In order to match packets with multiple TCP flags, you must first calculate the decimal value of the flag section of the TCP header. 

The 8 flags are ordered as shown below.

![](https://d2y9h8w1ydnujs.cloudfront.net/uploads/content/images/44865c4cda2679047739c6b38873318a7725280d53ec383568cb1af73cd76bb332585273ec900832494e079c6614.png)

If you want to match packets with multiple flag bits on, convert the flags into a decimal value and use this number in `**tcp[tcpflags] == x**`. For example, for packets with the PSH and ACK flags set, the decimal value becomes 00011000 = 24, and the expression becomes `**tcp[tcpflags] == 24**`, as shown below.

  
![](https://d2y9h8w1ydnujs.cloudfront.net/uploads/content/images/76579d1b8bc82200e86097e500d8c42b4e453a0a27600c76c40de1ec9aa643bd07e27b959344ad122d9e1fe71c81.png)

– **Specifying packet header bytes**:  
You can specify any byte in the packet header by using its index. For example, let’s suppose that we want to display only ICMP Echo Request packets. We know that echo requests have an ICMP type of 8. Since the type field in an ICMP header is the first byte, byte 0 (index is counted from 0) of the ICMP header should have a value of 8 (or 00001000 in binary). From this, we can construct our expression as `**icmp[0] == 8**`. The same applies for other protocol headers and values, such as `**ip[8]**` for the TTL.

Note that you can sometimes replace the byte index with the name of the field that you are specifying. For example, `**icmp[0]**` could be replaced with `**icmp[icmptype]**`. 

There are many other useful features in TCPDump, such as:

- `**-A**` to display packet contents in ASCII
- `**-x**` to print the hex dump of the packet
- `**-X**` to print both of the above, and
- `**t**`, `**-tt**`, `**-ttt**`, `**-tttt**`, `**-ttttt**` to manage timestamps

You should explore more options as you get more and more familiar with TCPDump. 

As TCPDump is a command-line, text-based tool unlike Wireshark, it also opens up the possibility to pipe the output to other shell commands and scripts for additional filtering, statistics calculation etc.. How useful this ability is largely depends on your familiarity with certain shell commands. Below are some commands which can manipulate TCPDump output for additional filtering.

– **cut -d “**_delimiter_**” -f** _no._: **cut** is a useful tool that can extract certain fields within each line with the use of delimiters. Delimiters are single characters that separate each field, with each field being able to be specified with a number.

  
![](https://d2y9h8w1ydnujs.cloudfront.net/uploads/content/images/b086e121714f475a605eadc26642ab6c9160d40f9b7fac9bf663dd725ec3abe677b0b850ae43c4895f7d589c1e7c.png)

For example, in the above image, the initial TCPDump output has been printed, however, we only want to display the first 3 sections of each line (time, Layer 3 protocol, source address and port). We notice that a ‘greater than’ symbol separates the 3 sections we are looking for, and the rest of the packet header fields – therefore, we can use the ‘greater than’ symbol as our delimiter, and specify the 1st field to be displayed with `**cut -d “>” -f 1**`. 

You can specify more than one field to be displayed, such as `**cut -d “:” -f 1,2,3**` to print up to the destination address and port.

– **awk ‘{print $**_no._**}’**: **awk** is a very powerful tool when it comes to text manipulation and one use case is to print a specific field within each line. Each field is separated by spaces.

  
![](https://d2y9h8w1ydnujs.cloudfront.net/uploads/content/images/ced919b9da3093354cce6b3ba8b1f0327d77b41f205453dafe613b14d72b436e74b583eb8d8ed841c22b072ecccf.png)

In the above image, only the 7th field – the TCP flags – are printed (the comma and the end has been removed). Since TCPDump uses spaces frequently to separate header fields, `**awk**` can be very handy in extracting a single field on all Packets. You can also use `**$NF**` for the last field and `**$(NF-**_**x**_**)**` for the _(x+1)_th field from the end, such as `**awk ‘{print $(NF-2)}’**` for the third last field.

– **grep** _keyword_: Even if you are a Unix CLI beginner, you will most likely know this Command. `**grep**` is the go-to command for finding lines within files that match a given keyword. For example, in order to find all FTP packets that relate to a .zip file, you can use `**grep zip**`, as shown below.

![](https://d2y9h8w1ydnujs.cloudfront.net/uploads/content/images/64a88e8a20305bf85f48a6dd43a2970f2d217e622f031b09f8dcaad5dffb923af07e0246353c42d1fd4c5ca072ed.png)

`**grep**` also has numerous other options available, such as `**-v**` to match lines that _don’t_ contain the keyword and `**-i**` to perform a case-insensitive search. On a more advanced level, regex can be used to fine-tune the search – but this is out of the scope of this course.

There are far more commands and tools available, on both Windows and Unix, which can be used to manipulate text output. You can also build Bash scripts that can be used to do all sorts of things – calculate averages, add parameters up, extract pieces of data from packets, and the list runs indefinitely.