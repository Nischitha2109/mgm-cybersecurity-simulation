# mgm-cybersecurity-simulation
Cybersecurity Red Team and Blue Team simulation based on the September 2023 MGM Resorts cyberattack, integrated with Computer Networks.


## CN Demonstration Guide
NetTracer: Traffic Analysis and Secure Communication in a Virtual Enterprise Network

### Step 1  Start applications and capture

Start enterprise_server.py on the enterprise VM and employee_pc.py on the employee VM using your existing launch commands; leave already running scripts alone.

Keep Blue Team available for recovery; open Wireshark on the employee VM with the command below.

`wireshark`

Double-click tailscale0 to capture and leave the display filter empty initially; keep capturing throughout the demonstration.

Only if dumpcap permission is denied, run these commands in order, then launch Wireshark from that same terminal:

`sudo usermod -aG wireshark 'groupname' `

`newgrp wireshark `

`id `

`wireshark`

Check that id includes wireshark; alternatively, log out of the desktop and back in after adding the group.

### Step 2  Verify IP connectivity

Run four pings from employee to enterprise:

`ping -c 4 100.74.186.22`

Apply this display filter and show Echo request and Echo reply packets:

`icmp && ip.addr == 100.74.186.22`


### Step 3  Record normal booking service communication

Before running the attack, apply this filter:

`ip.addr == 100.74.186.22 && tcp.port == 8090`

`curl --max-time 5 -i http://100.74.186.22:8090/`

Expected Output: HTTP/1.0 200 OK and TRAINING booking service available in the terminal.

Switch to Wireshark; SYN → SYN, ACK → ACK from the new connection, then expand Transmission Control Protocol to show the ports.

Select a packet from that connection → right-click → Follow → TCP Stream; show the GET request and booking-service response.

### Step 4  Record employee to enterprise API traffic

Apply this filter, which includes both traffic directions:

`ip.addr == 100.88.132.127 && ip.addr == 100.74.186.22 && tcp.port == 8082`

On Kali, run your existing attacker simulation command once.
Show the resulting 8082 packets and the Source, Destination, Protocol and Info columns; keep HTTP headers and payloads collapsed to avoid displaying lab tokens.



### Step 5  Record service disruption

After simulated impact and before recovery, apply:

`ip.addr == 100.74.186.22 && tcp.port == 8090`

Repeat the booking request:

`curl --max-time 5 -i http://100.74.186.22:8090/`

Show the terminal failure, then the newest SYN and RST, ACK packets in Wireshark.

Click RST, ACK and expand Transmission Control Protocol → Flags; do not use Follow TCP Stream because no HTTP data was exchanged.

### Step 6  Record service restoration
On Blue Team, click Contain and Restore and wait for the response procedure to finish; do not restart the enterprise script.
Keep the same 8090 filter; then run:

`curl --max-time 5 -i http://100.74.186.22:8090/`

Shows 200 OK, then select a packet from the newest successful connection → Follow → TCP Stream and show the request and response.


### Step 7  Record Kali to employee traffic

Use the existing capture; do not run the attack again. Apply:

`ip.addr == 100.93.38.13 && ip.addr == 100.88.132.127 && tcp.port == 8081`

### Step 8  Record the network setup

Run:

`tailscale status`

Show the four lab VMs

`ip -4 addr show tailscale0`

### All Blue Team project traffic

(tcp.port == 8081 || tcp.port == 8082 || tcp.port == 8090) && ip.addr == 100.116.229.43


Containment requests

`http.request.uri == "/manage/contain"`


Recovery requests

`http.request.uri == "/manage/restore"`


Booking service traffic

`tcp.port == 8090`


Connection resets

`tcp.flags.reset == 1 && tcp.port == 8090`


Failed attack retry

`http.response.code == 409`


Successful HTTP responses

`http.response.code == 200`

### All Red Team project traffic

`ip.addr == 100.93.38.13`

`ip.addr == 100.93.38.13 && ip.addr == 100.88.132.127 && tcp.port == 8081`

`tcp.port == 8081 || tcp.port == 8082`

To show **Red Team’s HTTP requests to the employee**, use:

```wireshark
ip.src == 100.93.38.13 && tcp.port == 8081 && http.request
```

For individual endpoints:

**Directory lookup**
```wireshark
http.request.uri == "/directory"
```

**Helpdesk reset**
```wireshark
http.request.uri == "/helpdesk/reset"
```

**Login**
```wireshark
http.request.uri == "/auth/login"
```

**Employee connecting to enterprise**
```wireshark
http.request.uri == "/employee/connect"
```

**Requesting simulated impact**
```wireshark
http.request.uri == "/employee/impact"
```

For the **forwarded employee → enterprise requests**:

```wireshark
http.request.uri == "/enterprise/accept" || http.request.uri == "/enterprise/impact"
```
