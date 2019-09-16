# Uptane Implementation Test Plan
## 22. Partial Bundle
**Test Case** : Attackers perform a MITM, such that, they drop a subset of images intended for the Primary (i.e., out of 3 images for the Primary, only 2 are sent). Observe how the Primary reacts to the missing update.  

**Test Steps** :  
1. Setup Test Computer  
Setup test computer to be able to communicate with both the Primary and the Uptane servers. Ensure this test computer has Python 3.5 or later installed.
2. Monitor a Valid Update  
Follow the Uptane tutorial on how to perform an update while monitoring the communication via Wireshark. Afterward, remove the connectivity of the servers and route traffic on the router destined for the servers to the attacking machine (e.g., add the rule manually on the router, ARP spoofing, etc.).
Ensure the attacking machine is listening on the same port as the repositories (i.e., port 30301 for the image repository, port 30401 for the director repository, and port 30601 for the timeserver) and is capable of handling the previously noted XMLRPC requests (i.e., get_signed_time, GET /metadata/timestamp.json, GET /111/metadata/timestamp.json, GET /targets/Secondary_update.img, submit_vehicle_manifest).   
3. Craft and Run Partial Bundle Attack  
Ensure the attacking machine is configured like the valid server except it is missing one of the three valid update images intended for the Primary. Run the following commands to execute the attack:   
>>> python3 rollback_update.py
python3 -m http.server 30301
python3 -m http.server 30401  


4. Monitor Response  
Afterward, run the following command on the Primary:  
-dp.update_cycle()  
Monitor the Primary’s response, and verify the output looks similar to the following:  
>>> dp.update_cycle()
   Submitting a request for a signed time to the Timeserver.
   Time attestation validated. New time registered.
    Now updating top-level metadata from the Director and Image Repositories
     (timestamp, snapshot, root, targets)
   [TRUNCATED...]
   A correctly signed statement from the Director indicates that this vehicle has
   updates to install:['/v3-update.txt', '/v1-update.txt', '/v2-update.txt']
   Metadata for the following Targets has been validated by both the Director and
   the Image repository. They will now be downloaded:['/v3-update.txt', '/v1-
   update.txt', '/v2-update.txt']
   Downloading: 'http://192.168.1.100:30301/targets/v3-update.txt'
   Could not download URL: 'http://192.168.1.100:30301/targets/v3-update.txt'
   HTTPError
   Update failed from http://192.168.1.100:30301/targets/v3-update.txt.
HTTPError
   Failed to update /v3-update.txt from all mirrors:
   {'http://192.168.1.100:30301/targets/v3-update.txt': <HTTPError 404: 'File not
   found'>}
   Downloading: 'http://192.168.1.100:30401/111/targets/v3-update.txt'
   Could not download URL: 'http://192.168.1.100:30401/111/targets/v3-update.txt'
   HTTPError
   Update failed from http://192.168.1.100:30401/111/targets/v3-update.txt.
   HTTPError
   Failed to update /v3-update.txt from all mirrors:
   {'http://192.168.1.100:30401/111/targets/v3-update.txt': <HTTPError 404: 'File
   not found'>}
   [2018.02.20 20:22:59UTC] [Primary] INFO [Primary.py:Primary_update_cycle():625]
   In downloading target 'v3-update.txt', am unable to find a mirror providing a
   trustworthy file. Checking the mirrors resulted in these errors: HTTPError from
   http://192.168.1.100:30401/111/targets/v3-update.txt; HTTPError from
   http://192.168.1.100:30301/targets/v3-update.txt;
   [...TRUNCATED...]
   Submitting the Primary's manifest to the Director.
   Submission of Vehicle Manifest complete.
   
**Test Scripts** :  partial_bundle_attack.py 

>>>import sys
from xmlrpc.server import SimpleXMLRPCServer
from xmlrpc.server import SimpleXMLRPCRequestHandler
class RequestHandler(SimpleXMLRPCRequestHandler):
  rpc_paths = ('/RPC2',)
timeserver = SimpleXMLRPCServer(("192.168.1.100", 30601),
requestHandler=RequestHandler, allow_none=True)
#Define a function and register the response
def get_signed_time(val1=False,val2=False,val3=False):
  response = {'signed': {'time': '2018-02-21T18:40:53Z', 'nonces':
[1253808851]}, 'signatures':
[{'keyid':'79c796d7e87389d1ebad04edce49faef611d139ee41ea9fb1931732afbfaac2e',
'sig':'0be89e5fcb10494b96b05c9018371ae3d817dad3a73d833bef60e07ed4021f224bf31cb2
a3cbf8c8ccf049823b57933d7b3ca33cc45b60fd2753dfae59055b0c',
'method':'ed25519'}]}
  return response
timeserver.register_function(get_signed_time,
      'get_signed_time')
try:
  timeserver.serve_forever()
except KeyboardInterrupt:
  print("\nKeyboard interrupt.")
  sys.exit(0)    
  
  partial_bundle_attack_director.py   
  
  
  >>>import sys
from xmlrpc.server import SimpleXMLRPCServer
from xmlrpc.server import SimpleXMLRPCRequestHandler
class RequestHandler(SimpleXMLRPCRequestHandler):
  rpc_paths = ('/RPC2',)
timeserver = SimpleXMLRPCServer(("192.168.1.100", 30501),
requestHandler=RequestHandler, allow_none=True)
#Define a function and register the response
def submit_vehicle_manifest(val1=False,val2=False,val3=False):
response = ''
  return response
timeserver.register_function(submit_vehicle_manifest,
      'submit_vehicle_manifest')
try:
  timeserver.serve_forever()
except KeyboardInterrupt:
  print("\nKeyboard interrupt.")
  sys.exit(0)


  