Goal of the project
------------


Repository directories
------------

- wireshark_src : source code of modified wireshark, based on version 3.0.1

- wireshark_15118vse.diff : Diff output between this project and wireshark 3.0.1

- screenshots : Screenshots of the execution of wireshark 15118 VSE feature

- sample_pcap : Sample packet capture file that includes beacon, association request/response messages

How to install
------------

### Install in ubuntu 18.04

~~~
1. sudo apt-get update && sudo apt-get upgrade

2. sudo apt install qttools5-dev qttools5-dev-tools libqt5svg5-dev qtmultimedia5-dev build-essential automake autoconf libgtk2.0-dev libglib2.0-dev flex bison libpcap-dev libgcrypt20-dev cmake -y
~~~

### Download wireshark from github and compile source code

~~~
3. mkdir /tmp

4. cd /tmp

5. git clone https://github.com/appseclab/wireshark15118vse

6. mkdir /tmp/build

7. cd /tmp/build

8. cmake /tmp/wireshark15118vse/wireshark_src

9. make
~~~

### Install wireshark

~~~
10. sudo make install

11. wireshark
~~~

How to use
------------

1. run wireshark

2. click file at the left corner and open "sample.pcapng" in /wireshark15118vse/sample_pcap folder

3. filter beacon frame, probe request, probe response etc..

	* beacon frame : wlan.fc.type_subtype eq 8

	* probe request : wlan.fc.type_subtype eq 4

	* probe response : wlan.fc.type_subtype eq 5

	* association request : wlan.fc.type_subtype eq 0

	* association response : wlan.fc.type_subtype eq 1

4. click packet information(ssid == linux_ap)

	* click IEEE 802.11 wireless LAN

	* click Tagged Parameters (nbytes)

	* click Tag : Vendor Specific: Vehicle to Grid and see information of the Message

Screenshots
------------



How wireshark is modified
------------

1. Add OUI to dissect ISO15118 VSE message

    * /wireshark15118vse/wireshark_src/epan/dissectors/oui.h

      * line 85 - 87 : add OUI (0x70b3d5)

2. ADD OUI Name

    * /wireshark15118vse/wireshark_src/manuf

      * line 24725, 26569 :  modify OUI Name IEEE Regiestration Authority to Vehicle to Grid

3. Call function when OUI == 0x70b3d5

    * /wireshark15118vse/wireshark_src/epan/dissectors/packet-ieee80211.c

      * line 19822 - 19832 :  if OUI == 0x70b3d5, goto line 13284 and call (dissect_vendor_ie_v2g) function


4. Add variables for dissecting

    * /wireshark15118vse/wireshark_src/epan/dissectors/packet-ieee80211.c

      * line 4794 - 4806 :  initialized to -1 that records our protocol
      * line 5840 - 5842 :  initialized to -1 when added a child node to the protocol tree which is where we will do our detail dissection.
                            The expansion of this node is controlled by the ett_v2g_flags_tree variable.
      * line 36716 - 36718 :  add ett_v2g_flags_tree

5. Dissect_vendor_ie_v2g function

    * /wireshark15118vse/wireshark_src/epan/dissectors/packet-ieee80211.c

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
