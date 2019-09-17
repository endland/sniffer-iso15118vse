Wireshark version
------------
wireshark_3.0.1

General Information
-------------------

Wireshark is a network traffic analyzer, or "sniffer", for Unix and
Unix-like operating systems.  It uses Qt, a graphical user interface
library, and libpcap, a packet capture and filtering library.

The Wireshark distribution also comes with TShark, which is a
line-oriented sniffer (similar to Sun's snoop or tcpdump) that uses the
same dissection, capture-file reading and writing, and packet filtering
code as Wireshark, and with editcap, which is a program to read capture
files and write the packets from that capture file, possibly in a
different capture file format, and with some packets possibly removed
from the capture.

The official home of Wireshark is https://www.wireshark.org.

The latest distribution can be found in the subdirectory https://www.wireshark.org/download

Wireshark + ISO15118VSE
------------

### Whats different from wireshark?

1. Add OUI to dissect ISO15118 VSE message

    * /wireshark_vse_version/epan/dissectors/oui.h

      * line 85 - 87 : add OUI (0x70b3d5)


2. Call function when OUI == 0x70b3d5

    * /wireshark_vse_version/epan/dissectors/packet-ieee80211.c

      * line 19822 - 19832 :  if OUI == 0x70b3d5, goto line 13284 and call (dissect_vendor_ie_v2g) function


3. Add variables for dissecting

    * /wireshark_vse_version/epan/dissectors/packet-ieee80211.c

      * line 4794 - 4806 :  initialised to -1 that records our protocol
      * line 5840 - 5842 :  initialised to -1 when added a child node to the protocol tree which is where we will do our detail dissection.
                            The expansion of this node is controlled by the ett_v2g_flags_tree variable.
      * line 36716 - 36718 :  add ett_v2g_flags_tree

4. Dissect_vendor_ie_v2g function

    * /wireshark_vse_version/epan/dissectors/packet-ieee80211.c

      * line 13284 - 13338 :  do the actual dissecting

Example messages
------------

### Beacon Frame, Probe Response :

~~~
If message shown as below :

  dd 25 70 b3 d5 31 90 01 03 44 45 58 59 5a 01 23 45 67 88 41 43 3a 43 3d 31 7c 57 50 54 3a 5a 3d 32 3a 50 3d 31 2c 32

  dd -> tag number                                                                              1bytes
  25 -> VSE message total length                                                                1bytes
  70 b3 d5 -> OUI (70:b3:d5) in wireshark                  (this dissects in 19821 line)        3bytes

  static int dissect_vendor_ie_v2g function dissect as below:
  31 90 -> IAB (ISO TC22/SC31 15118)                                                            2bytes
  01 -> Type Type (SECC)                                                                        1bytes
  03 -> ETT                                                                                     1bytes
  44 45 -> country code                                                                         2bytes
  58 59 5a -> operator ID                                                                       3bytes
  01 23 45 67 88 -> charging site id                                                            5bytes
  41 43 3a 43 3d 31 7c 57 50 54 3a 5a 3d 32 3a 50 3d 31 2c 32 -> additional information         remain bytes
~~~

### Probe Request:

~~~
If message shown as below :

  dd 0c 70 b3 d5 31 90 02 05 01 23 45 67 89

  dd -> tag number
  0c -> VSE MSG total length                                                                    1bytes
  70 b3 d5 -> OUI (70:b3:d5) in wireshark                 (this dissects in 19821 line)         3bytes

  static int dissect_vendor_ie_v2g function dissect as below:
  31 90 -> IAB (ISO TC22/SC31 15118)                                                            2bytes
  02 -> Type (EVCC)                                                                             1bytes
  05 -> ETT                                                                                     1bytes
  01 23 45 67 89 -> additional information                                                      remain bytes
~~~
