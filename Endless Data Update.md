# Uptane Implementation Test Plan
# 10. Endless Data Update
**Test Case** : Attempt to send an endless data update to the Uptane Primary.   
**Test Steps** :  
1. Setup Test Computer :   
 Setup test computer to be able to communicate with both the Primary and the Uptane servers. Ensure this test computer has Python 3.5 or later installed.  

2.Monitor a Valid Update :  
Follow the Uptane tutorial on how to perform an update while monitoring the communication via Wireshark. Copy the responses sent from the servers to the Primary when the Primary performs an update_cycle().
Afterward, remove the connectivity of the servers and route traffic on the router destined for the servers to the attacking machine (e.g., add the rule manually on the router, ARP spoofing, etc.).
Ensure the attacking machine is listening on the same port as the repositories (i.e., port 30301 for the image repository, port 30401 for the director repository, and port 30601 for the timeserver) and is capable of handling the previously noted XMLRPC requests (i.e., get_signed_time, GET /metadata/timestamp.json, GET /111/metadata/timestamp.json, GET /targets/Secondary_update.img, submit_vehicle_manifest).    
3. Craft Endless Data Attack   
Create a file structure on the attacking machine from the attacking directory (e.g., ~/uptane/endless_data/) that mimics the valid Uptane server (i.e., ~/uptane/endless_data/metadata/, ~/uptane/endless_data/targets/, etc.). Create a large update (e.g., 1GB) into the expected update filename (i.e., update.txt) by running the following command:   
- dd if=/dev/zero bs=1024 count=1000000 > ~/uptane/endless_data/update.txt   

Afterward, navigate to the attacking directory (i.e., ~/uptane/endless_data/) and append the expected update data to the newly created 1GB update via the following command.
- echo ‘expected data’ | cat – update.txt >> temp.txt && mv temp.txt update.txt   

Lastly, copy the rogue update into the two expected directories via the following commands:  
cp update.txt targets/
cp update.txt [VIN]/targets/  

4. Run Endless Data Attack   
Run the attack by running the following commands in separate terminal windows:
-python3 endless_data_attack.py
-python3 -m http.server 30401
-python3 -m http.server 30301   

Afterward, run the following command on the Primary:  
- dp.update_cycle  

5. Monitor Response   
Monitor the response from the Primary. Verify the Primary output looks similar to the following.   
---
[...TRUNCATED...]
[Primary.py:Primary_update_cycle():563]
Metadata for the following Targets has been validated by both the Director and
the Image repository. They will now be downloaded:['/Secondary_update.txt']
Downloading: 'http://192.168.1.100:30301/targets/Secondary_update.txt'
Downloaded 18 bytes out of the expected 18 bytes.
Not decompressing http://192.168.1.100:30301/targets/Secondary_update.txt
Update failed from http://192.168.1.100:30301/targets/Secondary_update.txt.
BadHashError
Failed to update /Secondary_update.txt from all mirrors:
{'http://192.168.1.100:30301/targets/Secondary_update.txt':
BadHashError('651bdb7fa636052949a6220202c5faa7b9258a5dcb31ad01632b49c338c28b27'
, 'e116d4ef5a2f2dbba9a61970a25cab3e6695418e3dbfa71071e4d07aebb1f083')}
Downloading: 'http://192.168.1.100:30401/111/targets/Secondary_update.txt'
Downloaded 18 bytes out of the expected 18 bytes.
Not decompressing http://192.168.1.100:30401/111/targets/Secondary_update.txt
The file's 'sha256' hash is correct:
'651bdb7fa636052949a6220202c5faa7b9258a5dcb31ad01632b49c338c28b27'
The file's 'sha512' hash is correct:
'994d865396d913f8754af181aeba16996a44a07de595dea2c3a7f96ce0a3910aa8b74905edbb30
94954aabffe20f14dd2b3f0ea82767960c9fb030886fbb56ef'
[2018.02.14 15:29:18UTC] [Primary] INFO [Primary.py:Primary_update_cycle():651]
Successfully downloaded trustworthy 'Secondary_update.txt' image.
Submitting the Primary's manifest to the Director.
[...TRUNCATED...]   
---

Examine the tuf.log file to determine if there is any output related to our malicious update.  
- ---
[...TRUNCATED...]
18-02-14 15:29:18,899 UTC] [tuf.download] [DEBUG]
[_check_content_length:547@download.py]
The server reported a length of 1024000019 bytes.
[...TRUNCATED...]
---

As seen in the output above, although the Primary recognizes the update from the server is 1GB, the Primary limits the data it downloads to the expected data length (previously determined from signed metadata from director and image repository). Therefore, to successfully launch an endless data attack, both repository keys would need to be compromised sign the extremely large update. Therefore, this test passed.  
**Test Scripts** : endless_data_attack.py   
---
import sys
from xmlrpc.server import SimpleXMLRPCServer
from xmlrpc.server import SimpleXMLRPCRequestHandler
class RequestHandler(SimpleXMLRPCRequestHandler):
  rpc_paths = ('/RPC2',)
timeserver = SimpleXMLRPCServer(("192.168.1.100", 30601),
requestHandler=RequestHandler, allow_none=True)
#Define a function and register the response
def get_signed_time(val1=False,val2=False,val3=False):
  response = {'signed': {'time': '2018-02-14T13:48:02Z', 'nonces':
[610636176,1077783583]}, 'signatures':
[{'keyid':'79c796d7e87389d1ebad04edce49faef611d139ee41ea9fb1931732afbfaac2e',
'sig':'2f1169c382bb67f811d33fa4bff7529606724b5639bb9e61484dde5b4a078a44a9c4a409
80bf83da3f2aaccf05b213fd1df3fc10c7243b13dbba30bfe0f56e06',
'method':'ed25519'}]}
  return response
timeserver.register_function(get_signed_time,
      'get_signed_time')
try:
  timeserver.serve_forever()
except KeyboardInterrupt:
  print("\nKeyboard interrupt.")
  sys.exit(0)
---



