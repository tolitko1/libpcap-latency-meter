This tool includes the following software:

* jNetPcap library developed by Sly Technologies and distributed under LGPL license and is free of charge and royalties.
  http://www.jnetpcap.com/

* HdrHistogram library by Gil Tene (distributed under Creative Commons License)
  http://hdrhistogram.github.io/HdrHistogram/

LBPCAP Latency Meter itself is distributed under LGPL license free of charge and royalties.

andymalakov/libpcap-latency-meter
•	
•	
•	
•	
•	
•	
Overview
andymalakov edited this page on Dec 4, 2014 • 18 revisions
 Pages 6
•	Home
•	Limitations
•	Linux
•	Overview
•	QuickStart
•	Windows
Clone this wiki locally
 
 Clone in Desktop
What is it?
LIBPCAP Latency Meter is a small tool that measures time difference between inbound and outbound TCP packets messages.
How does it work?
The tool intercepts inbound and outbound TCP packets containing your network messages. All captured packets carry time stamps. This tool matches outbound packets with corresponding inbound packets using some kind of correlation key (For example, value of specific FIX tags).
 
For example, this tool can be used measure latency of trading signals. FX market data feed usually contain a field QuoteEntryID(299), while FX orders may contain a fieldQuoteID(117). Tool can correlate inbound and outbound messages based on the value provided in these tags.
 
Simple statistics (min/max/avg) are printed out during run time and latency log of each signal is recorded into CSV file.
This tool can monitor latency of live network traffic or process previously captured traffic in pcap format (for example produced by Wireshark).
This tool is based on jNetPcap library, which is in turn is a wrapper around libpcap.
Results
Tool prints brief stats during its run and records latency of every signal into binary file. Binary file can be later converted to CSV format.
Additional utility produces percentile histograms from collected latencies (using HdrHistogram):

How precise is it?
Time stamps of each packet come from libpcap (WinPcap) library, which in turn gets them from the operating system kernel. WinPcap has microsecond precision, while libpcap (Unix origin) has nanosecond precision.
How to use it?
See Quick Start wiki page.
Limitations
•	Current version does not handle TCP re-transmissions and duplicate packets. It is possible to implement protection against these exceptions using JNetPcap API, but currently this is out-of-scope.
•	Current version doesn't re-assemble messages split between several TCP packets. jNetPcap has examples of doing that.
•	Current version relies on strict order of inbound and outbound signals.
•	Any outbound signal must have corresponding inbound signal. Timestamps of inbound signals are stored in ring buffer. Outbound packet processor consumes ring buffer as it searches for matching signal.
Disclaimer
This tool is provided "as is" without warranty of any kind and is to be used at your own risk.
License
This tool is distributed free of charge and royalties under LGPL license
Tool includes the following open source libraries:
•	jNetPcap software developed by Sly Technologies.
•	HdrHistogram from Gil Tene.
•	jUnit.
________________________________________
This tool has been tested with jNetPcap 1.4.r1425 with:
•	Windows platform: Windows Server 2008 SR2, Windows 7 SP1 Pro (WinPcap 4.1.3)
•	Linux platform: Centos 7.0 (LIBPCAP 1.5.3) and Fedora 19 (LIBPCAP 1.4.0)
andymalakov/libpcap-latency-meter
•	
•	
•	
•	
•	
•	
Limitations
andymalakov edited this page on Dec 4, 2014 • 6 revisions
 Pages 6
•	Home
•	Limitations
•	Linux
•	Overview
•	QuickStart
•	Windows
Clone this wiki locally
 
 Clone in Desktop
Known limitations
•	This tool cannot handle large messages. Tool does not re-assemble messages that were split between multiple TCP packets. We assume that your network's layer maximum packet size (MTU) is large enough to fit messages that you are capturing (Typically 1500 bytes). This should be true for most FIX traffic, with exception of SecurityDefinition(d) and MarketData-Snapshot/FullRefresh(W) messages containing many repeating groups.
•	This tool cannot process SSL-encrypted traffic. If your broker/market data provider requires SSL connection you can use STUNNEL gateway to unwrap SSL encryption.
Linux
andymalakov edited this page on Dec 4, 2014 • 11 revisions
 Pages 6
•	Home
•	Limitations
•	Linux
•	Overview
•	QuickStart
•	Windows
Clone this wiki locally
 
 Clone in Desktop
Linux walk-through
The following steps describe how to use this tool on Linux. We used Centos 7.0 (x64).
The following steps may require root privileges.
Step 0: Get a copy
git clone https://github.com/andymalakov/libpcap-latency-meter.git
Steps below assume this library resides in libpcap-latency-meter folder.
Step 1: LIBPCAP library
Make sure libpcap is installed
yum install libpcap
Our "Minimal" installation of CentOS came with libpcap 1.5.3 pre-installed. Let's find out where it is (may be needed later):
#find / -name '*libpcap*'

/usr/lib64/libpcap.so.1
/usr/lib64/libpcap.so.1.5.3
/usr/share/doc/libpcap-1.5.3
Step 2: jNetPcap (Java wrapper for LIBPCAP)
For CentOS we took RHEL5 x64 version of jNetPcap library from [http://jnetpcap.com/download]. Extract GZ and copy the following files into libpcap-latency-meter/bin/:
jnetpcap.jar libjnetpcap-pcap100.so libjnetpcap.so
Let's check
#ldd bin/libjnetpcap.so
If you see libpcap.so.0.9.4 => not found, you may need to define symbolic link:
#ln -s /usr/lib64/libpcap.so.1  /usr/lib/libpcap.so.0.9.4
jNetPcap 1.4 is upward compatible with Libpcap 1.X.
Step 3: Build it
ant build
Step 4: Verify setup
Let's capture some network traffic. Build script includes 'hello' target that displays network interfaces discovered by LIBPCAP and captures first few packets from interface #0.
ant hello
hello:
 [java] Network devices found:
 [java] #0: lo [No description available]
 [java] #1: any [Pseudo-device that captures on all interfaces]
 [java] #2: enp3s0 [No description available]
 [java] #3: enp2s0 [No description available]
 [java] #4: usbmon1 [USB bus number 1]
 [java] #5: nfqueue [Linux netfilter queue (NFQUEUE) interface]
 [java] #6: nflog [Linux netfilter log (NFLOG) interface]
 [java]
 [java] Choosing 'lo' on your behalf:
 [java] Received packet at Thu Dec 04 15:49:12 EST 2014 caplen=296  len=296 ...
In our case interface #0 is local loopback. Interfaces enp2s0 and enp3sp are network cards:
#nmcli connection show

NAME    UUID                                  TYPE            DEVICE
enp2s0  8f7b42ed-e26a-458e-b072-7b24129da9c7  802-3-ethernet  --
enp3s0  33d09ea0-0c71-4bd6-a21f-16d30ba74d7c  802-3-ethernet  enp3s0
Troubleshooting
Most errors are caused by missing LIBPCAP, try adding /usr/lib to library search path:
export LD_LIBRARY_PATH=/usr/lib
If this doesn't help, we suggest looking into jNetPcap release notes.
Overview
andymalakov edited this page on Dec 4, 2014 • 18 revisions
 Pages 6
•	Home
•	Limitations
•	Linux
•	Overview
•	QuickStart
•	Windows
Clone this wiki locally
 
 Clone in Desktop
What is it?
LIBPCAP Latency Meter is a small tool that measures time difference between inbound and outbound TCP packets messages.
How does it work?
The tool intercepts inbound and outbound TCP packets containing your network messages. All captured packets carry time stamps. This tool matches outbound packets with corresponding inbound packets using some kind of correlation key (For example, value of specific FIX tags).
 
For example, this tool can be used measure latency of trading signals. FX market data feed usually contain a field QuoteEntryID(299), while FX orders may contain a fieldQuoteID(117). Tool can correlate inbound and outbound messages based on the value provided in these tags.
 
Simple statistics (min/max/avg) are printed out during run time and latency log of each signal is recorded into CSV file.
This tool can monitor latency of live network traffic or process previously captured traffic in pcap format (for example produced by Wireshark).
This tool is based on jNetPcap library, which is in turn is a wrapper around libpcap.
Results
Tool prints brief stats during its run and records latency of every signal into binary file. Binary file can be later converted to CSV format.
Additional utility produces percentile histograms from collected latencies (using HdrHistogram):

How precise is it?
Time stamps of each packet come from libpcap (WinPcap) library, which in turn gets them from the operating system kernel. WinPcap has microsecond precision, while libpcap (Unix origin) has nanosecond precision.
How to use it?
See Quick Start wiki page.
Limitations
•	Current version does not handle TCP re-transmissions and duplicate packets. It is possible to implement protection against these exceptions using JNetPcap API, but currently this is out-of-scope.
•	Current version doesn't re-assemble messages split between several TCP packets. jNetPcap has examples of doing that.
•	Current version relies on strict order of inbound and outbound signals.
•	Any outbound signal must have corresponding inbound signal. Timestamps of inbound signals are stored in ring buffer. Outbound packet processor consumes ring buffer as it searches for matching signal.
Disclaimer
This tool is provided "as is" without warranty of any kind and is to be used at your own risk.
License
This tool is distributed free of charge and royalties under LGPL license
Tool includes the following open source libraries:
•	jNetPcap software developed by Sly Technologies.
•	HdrHistogram from Gil Tene.
•	jUnit.
________________________________________
This tool has been tested with jNetPcap 1.4.r1425 with:
•	Windows platform: Windows Server 2008 SR2, Windows 7 SP1 Pro (WinPcap 4.1.3)
•	Linux platform: Centos 7.0 (LIBPCAP 1.5.3) and Fedora 19 (LIBPCAP 1.4.0)
QuickStart
Andy Malakov edited this page on Mar 22 • 36 revisions
 Pages 6
•	Home
•	Limitations
•	Linux
•	Overview
•	QuickStart
•	Windows
Clone this wiki locally
 
 Clone in Desktop
General description of this tool can be found here.
Setup
•	Windows setup walk-through
•	Linux setup walk-through
Run
Live traffic monitoring
If your computer has multiple network adapters or VPN software, you need to obtain find out ID of network interface that you want to monitor.
NOTE: Due to specifics of Windows TCP/IP implementation WINPCAP cannot monitor loopback traffic (localhost).
Run bin/live-traffic script without any arguments.
live-traffic
Script will display command line arguments help, followed by list of interfaces detected by LIBPCAP.
Network devices found:
#0: \Device\NPF_{0093D9BE-190D-4B59-BF8F-D9CE04004DBE} [Marvell Yukon Ethernet Controller.]
#1: \Device\NPF_{BC81C4FC-242F-4F1C-9DAD-EA9523CC992D} [Intel(R) PRO/100 VE Network Connection]
Let's assume we want to capture network packets on Yukon network card from the above list (index #0). This interface can be selected using command line argument -interface:0.
Example 1
Imagine we are measuring latency of FIX server that receives orders and immediately acknowledges them. Server has FIX Acceptor on port 7777. Inbound messages (NewOrderSingle) are identified using FIX tag 11. Outbound messages (ExecutionReport) use the same tag to identify order they acknowledge.
live-traffic.cmd -interface:0 -in:fix:11 -out:fix:11  "-filter:(tcp port 777)"
Utility is going to correlate inbound and outbound messages based on order ID (tag 11). By default utility assumes that direction of each packet can be determined relative to local IP address. We can also be more excplicit and specify this using -dir argument (direction relative to host 'sauron'):
live-traffic.cmd -interface:0 -in:fix:11 -out:fix:11  -dir:sauron "-filter:(tcp port 777)"
Example 2
The following example shows how to capture FIX conversation with Integral FX-Inside FIX Server. Here we assume that the server offers two FIX connections: 1) Market Data Session is running on port 2508; 2) Trading Session is running on port 2509. We are going to parse inbound FIX market data and extract tag QuoteEntryID(299) from bids and asks of market data prices. For outbound traffic we are going parse outbound FIX messages and extract tag QuoteID(117) from trade orders. Utility is going to correlate inbound and outbound messages based on the quote IDs.
live-traffic.cmd -interface:0 -in:fix:299 -out:fix:117 -dir:2509:2508 "-filter:(tcp src port 2509) or (tcp dst port 2508)"
Previously captured traffic processing
This tool can also process previously captured network traffic. For example you can setup your own capture using Wireshark or TCPDump and later process it via this tool.
Command line syntax is similar
filed-traffic -pcap:./data/timebase.pcap  -in:fix:299 -out:fix:117 -dir:2509:2508
Windows
andymalakov edited this page on Dec 4, 2014 • 3 revisions
 Pages 6
•	Home
•	Limitations
•	Linux
•	Overview
•	QuickStart
•	Windows
Clone this wiki locally
 
 Clone in Desktop
Windows walk-through
The following steps describe how to use this tool on Linux. We used Centos 7.0 (x64).
The following steps may require root privileges.
Step 0: Get a copy
git clone https://github.com/andymalakov/libpcap-latency-meter.git
The following notes assume that you placed them into LATENCY_METER directory.
Step 1: WINPCAP library
Download and install winpcap. We tested with version 4.1.3.
Step 2: jNetPcap (Java wrapper for LIBPCAP)
•	Download jNetPcap. We tested with version 1.4.r1425.
•	Copy jnetpcap.dll and jnetpcap-pcap100.dll into LATENCY_METER/bin.
•	Copy jnetpcap.jar into LATENCY_METER/lib.
Step 3: Build it
ant build
Step 4: Verify setup
Let's capture some network traffic. Build script includes 'hello' target that displays network interfaces discovered by WINPCAP and captures first few packets from interface #0.
ant hello
hello:
 [java] Network devices found:
 [java] #0: \Device\NPF_{0093D9BE-190D-4B59-BF8F-D9CE04004DBE} [Marvell Yukon Ethernet Controller.]
 [java] #1: \Device\NPF_{BC81C4FC-242F-4F1C-9DAD-EA9523CC992D} [Intel(R) PRO/100 VE Network Connection]
 [java]
 [java] Choosing 'Marvell Yukon Ethernet Controller.' on your behalf:
 [java] Received packet at Thu Dec 04 16:15:36 EST 2014 caplen=107  len=107  ....




