# Uptane Implementation Test Plan
## 15. Rollback Update
**Test Cases** : Send an update with an older version number than what is currently installed on the Uptane client.

**Test Steps** : 
1. Setup Test Computer  
Setup test computer to be able to communicate with both the Primary and the Uptane servers. Ensure this test computer has Python 3.5 or later installed.
2. Monitor a Valid Update  
Follow the Uptane tutorial on how to perform an update (v1) while monitoring the communication via Wireshark. Copy the responses sent from the servers to the Primary when the Primary performs an update_cycle(). Then perform a second update (v2) and ensure the Primary downloads the update successfully.
Afterward, remove the connectivity of the servers and route traffic on the router destined for the servers to the attacking machine (e.g., add the rule manually on the router, ARP spoofing, etc.).
Ensure the attacking machine is listening on the same port as the repositories (i.e., port 30301 for the image repository, port 30401 for the director repository, and port 30601 for the timeserver) and is capable of handling the previously noted XMLRPC requests (i.e., get_signed_time, GET /metadata/timestamp.json, GET /111/metadata/timestamp.json, GET /targets/Secondary_update.img, submit_vehicle_manifest).  
3. Craft and Run Rollback Update Attack
Ensure the attacking machine is configured like a server and call update_cycle() on the Primary. Craft a response with the original update captured (i.e., v1) for when the Primary calls update_cycle().  
4. Run Rollback Update Attack 
Run the following commands to execute the attacks. 
>>>python3 rollback_update.py
python3 -m http.server 30301
python3 -m http.server 30401  

5. Monitor Response
Monitor the Primary’s response, and verify the output looks similar to the following: 
>>> dp.update_cycle()
[...TRUNCATED...]
Failed to update timestamp.json from all mirrors: {'http://192.168.1.100:30401/111/metadata/timestamp.json': ReplayedMetadataError('timestamp', 2, 3)}
[...TRUNCATED...]
The Director has instructed us to download a Timestamp that is older than the
currently trusted version. This instruction has been rejected.
Submitting the Primary's manifest to the Director.  


Monitor the logs the Primary outputs, and verify the output looks similar to the following:  
>>>[...TRUNCATED...]   
[2018-02-15 19:56:46,848 UTC] [tuf.client.updater] [ERROR]
   [_get_metadata_file:1779@updater.py]
   Update failed from http://192.168.1.100:30401/111/metadata/timestamp.json.
   Traceback (most recent call last):
    File "/home/pi/workspace/uptane/src/tuf/tuf/client/updater.py", line 1410, in
   refresh
     self._update_metadata('timestamp', DEFAULT_TIMESTAMP_UPPERLENGTH)
    File "/home/pi/workspace/uptane/src/tuf/tuf/client/updater.py", line 2072, in
   _update_metadata
     compression_algorithm)
    File "/home/pi/workspace/uptane/src/tuf/tuf/client/updater.py", line 1792, in
   _get_metadata_file
     raise tuf.NoWorkingMirrorError(file_mirror_errors)
   tuf.NoWorkingMirrorError: No working mirror was found:
    '192.168.1.100:30401': ReplayedMetadataError('timestamp', 2, 3)
   [...TRUNCATED...]

**Test Scripts**:   
rollback_update_attack.py   

>>>from xmlrpc.server import SimpleXMLRPCServer
from xmlrpc.server import SimpleXMLRPCRequestHandler
class RequestHandler(SimpleXMLRPCRequestHandler):
  rpc_paths = ('/RPC2',)
timeserver = SimpleXMLRPCServer(("192.168.1.100", 30601),
requestHandler=RequestHandler, allow_none=True)
#Define a function and register the response
def get_signed_time(val1=False,val2=False,val3=False):
  response = {'signed': {'time': '2018-02-15T07:10:12Z', 'nonces':
[1108554777]}, 'signatures':
[{'keyid':'79c796d7e87389d1ebad04edce49faef611d139ee41ea9fb1931732afbfaac2e',
'sig':'1349dfb7052de2dfb0460e6018ddae489aa00cb3d2ed578776126376893d6c93ccd97ed3
83a46f2afe2ef2a3fcaafb4a04ce91ce987c67aa454b72b01a22fc0b',
'method':'ed25519'}]}
  #response = {'signed': {'time': '2018-02-14T13:48:02Z', 'nonces':
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


