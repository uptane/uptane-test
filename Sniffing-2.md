# Uptane Implementation Test Plan
# 1. Sniffing
**Test Case** : Monitor all traffic coming to/from the Uptane client to the Uptane server during all points of communication (e.g , registration, download, etc.)
**Test Steps** 
*Step 1*: Setup Test Computer 
Ensure the 3 Raspberry Pi’s are setup to emulate the servers (i.e., Director, Image, Timeserver), the Primary, and the Secondary. Afterward, configure the test computer to be able to communicate with the devices (i.e., on the same network). Plug the test computer into the Throwing Star LAN Tap Pro to be able to sniff all traffic to/from the Primary
*Step 2* : Monitor the traffic
Monitor the traffic to/from the Primary using Wireshark on the test computer previously setup. Perform an update by following the procedures listed in the readme located at https://github.com/uptane/uptane-test.
*Step 3*: Examine Traffic
After examining the traffic, several RPC calls are made between the Primary and Secondary, and the Primary and the Servers. Additionally, all traffic was communicated in the clear. These results will be utilized for future testing.



