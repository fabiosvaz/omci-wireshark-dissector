# omci-wireshark-dissector
Wireshark Dissector for OMCI protocol

### Install omci wireshark dissectors
Install omci.lua and BinDecHex.lua dissectors from this repo. 

The files are originally from [wiki.wireshark.org/Contrib](https://wiki.wireshark.org/Contrib#Protocol_Dissectors), and it already contains a fix [fix_reference](https://ask.wireshark.org/question/4557/bindechexlua-error-bad-argument-to-module-packageseeall/?answer=4573#post-id-4573) to BinDecHex.lua.

Ubuntu

    mkdir -p $(HOME)/.local/lib/wireshark/plugins
    cd $(HOME)/.local/lib/wireshark/plugins
    wget https://raw.githubusercontent.com/fabiosvaz/omci-wireshark-dissector/main/BinDecHex.lua
    wget https://raw.githubusercontent.com/fabiosvaz/omci-wireshark-dissector/main/omci.lua

### Install gen_hexdump utility
The gen_hexdump utility will be useful to convert the hex strings from captured omci packets to a wireshark hexdump:

```
mkdir gen_hexdump
cd gen_hexdump
git clone https://github.com/bo-yang/misc/blob/master/gen_hexdump.cc
g++ -Wall -std=c++0x -o gen_hexdump gen_hexdump.cc
```

### Dumping omci packet from logs
For my use case, I collect trace logs containing omci information from my running container. A 'grep' command is then used to extract only the omci packets based on a specific log output.

```
docker logs container_name  2>&1 | tee container_name.log
sudo cat container_name.log  | grep -Po 'OmciMsg: \K[^"]*' | tee omciPackets.log
```

This will provide a file containing:
```
00014f0a00020000000000000000000000000000000000000000000000000000000000000000000000000028
00012f0a0002000000000000000000000000000000000000000000000000000000000000000000000000000000000000
00024d0a00020000000000000000000000000000000000000000000000000000000000000000000000000028
00022d0a0002000001230000000000000000000000000000000000000000000000000000000000000000000000000000
00034e0a00020000000000000000000000000000000000000000000000000000000000000000000000000028
00032e0a0002000000020000800000000000000000000000000000000000000000000000000000000000000000000000
```

### Convert omci packet dumps to Wireshark-understandable hexdump

Convert the hex strings from omciPackets.log to a wireshark hexdump:
```
gen_hexdump -i omciPackets.log -o omciPackets.hex
```
### Load omci hexdump into wireshark

    File -> Import from Hex Dump
    Browse -> omciPackets.hex
    Encapsulation Type -> Ethernet
    Ethernet -> Ethertype (hex): 88b5
    
![alt text](https://github.com/fabiosvaz/omci-wireshark-dissector/blob/main/omciWiresharkDissector.JPG?raw=true)

### References
https://gist.github.com/shadansari/4159cf94dfc629d41215a844bd6ad1e7
https://wiki.wireshark.org/Contrib#Protocol_Dissectors
https://ask.wireshark.org/question/4557/bindechexlua-error-bad-argument-to-module-packageseeall/?answer=4573#post-id-4573
