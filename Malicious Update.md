# Uptane Implementation Test Plan
## 12. Malicious Update
**Test Case** : Modify a valid update and send it to the Uptane client to determine if it detects a malicious update.  

**Test Steps** :  
1. Setup Test Computer : 
 Setup a test computer to be able to communicate with both the Primary and the Uptane servers. Ensure this test computer has Python 3.5 or later installed.  
2. Perform an Update and Launch Attack  
Follow the tutorial on how to perform an update on a Secondary while monitoring the communication via Wireshark. Monitor the responses sent from the Primary to the Secondary when the Secondary performs an update_cycle().
Afterward, add a new version of the update to the repository. Push the update to the Primary by running the following command on the Primary:
- dp.update_cycle()  

Next, remove the connectivity of the Primary and route traffic on the router destined for the Primary to the attacking machine (either add the rule manually on the router or ARP spoofing). Additionally, copy the director/ and imagerepo/ directories onto the attacking machine and zip them into one file via the following command: 
- zip -r malicious_metadata_archive.zip director/ imagerepo/  

Ensure the attacking machine is listening on the same port as the Primary (port 30701) and is capable of handling the previously noted XMLRPC requests (i.e., get_time_attestation_for_ecu, get_metadata, update_exists_for_ecu, get_image, submit_ecu_manifest). Run the malicious Primary by running the following command:  
- python malicious_update.py  

Perform the following command on the Secondary.  
- ds.update_cycle()  
Observe the output of the Secondary.

3. Monitor Secondary Response  
Verify the response from the Secondary looks similar to the following when providing an update with the wrong file name.  

>>> ds.update_cycle()
   Timeserver attestation from Primary does not check out: This Secondary's nonce
   was not found. Not updating this Secondary's time this cycle.
   Verifying 'timestamp'. Requesting version: None
   Downloading:
   [...TRUNCATED...]
     dp.update_cycle()
        zip -r malicious_metadata_archive.zip director/ imagerepo/
        python malicious_update.py
        ds.update_cycle()
                     Requested and received image from Primary, but this Secondary has not validated any target info that matches the given filename. Expected: 'Secondary.txt'; received: 'Secondary_update.txt'; aborting "install".   
                     >>>
Verify the response from the Secondary looks similar to the following when providing an update with the wrong file length.   
>>> ds.update_cycle()
Timeserver attestation from Primary does not check out: This Secondary's nonce
was not found. Not updating this Secondary's time this cycle.
Verifying 'timestamp'. Requesting version: None
Downloading:
[...TRUNCATED...]
Image from Primary failed to validate: length mismatch. Image: 'Secondary.txt'    

Verify the response from the Secondary looks similar to the following when providing an update with the wrong file hash (i.e., a malicious or modified file).   

>>> ds.update_cycle()
Timeserver attestation from Primary does not check out: This Secondary's nonce
was not found. Not updating this Secondary's time this cycle.
Verifying 'timestamp'. Requesting version: None
Downloading:
[...TRUNCATED...]
Image from Primary failed to validate: hash mismatch. Image: 'Secondary.txt'   


As seen above, the Secondary realizes the timeserver attestation does not contain the Secondary’s nonce, so it does not update it’s time. However, the Secondary continues to verify the metadata, which implies, its update process was not hindered by a non-valid timeserver attestation response.
Ultimately, the Secondary recognizes that the malicious update we tried to send does not match the expected filename, length, or hash of the update as detailed in both the director’s and image repository’s targets.json file. Thus, the Secondary does not attempt to install our malicious update, therefore, this test passed.   

**Test Scripts** :  malicious_update.py  

import sys
   from xmlrpc.server import SimpleXMLRPCServer
   from xmlrpc.server import SimpleXMLRPCRequestHandler
   class RequestHandler(SimpleXMLRPCRequestHandler):
     rpc_paths = ('/RPC2',)
   #Act as the Primary
   server = SimpleXMLRPCServer(("192.168.1.81", 30701),
   requestHandler=RequestHandler, allow_none=True)
   #Define a function and register the response
   def get_time_attestation_for_ecu(val1=False,val2=False,val3=False):
                             response = {'signed': {'time': '2018-02-20T17:13:30Z', 'nonces':
   [1574771411]}, 'signatures':
   [{'keyid':'79c796d7e87389d1ebad04edce49faef611d139ee41ea9fb1931732afbfaac2e',
   'sig':'d60642c791ac15bc8f5546bd596831a0fd1802d8e4a818228da87c942f6ff3e5a8346597
   01f59231c6d8872333210b5c6253a0af79217639b166275ce99da90f',
   'method':'ed25519'}]}
     return response
   server.register_function(get_time_attestation_for_ecu,
         'get_time_attestation_for_ecu')
   #Define a function and register the response
   def get_metadata(val1=False,val2=False,val3=False):
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
     response=['Secondary.txt', b'v7 SECONDARY_ECU_1']
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



 