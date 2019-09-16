# Uptane Implementation Test Plan
# 9. Key Revocation
**Test Case** : Examine the key revocation process and attempt to exploit the process by sending an unauthorized key revocation command to the Uptane client.    

** Test Steps**:     
1. Examine Documents for Key Revocation :    
The Uptane framework has key revocation built in because of the underlying TUF framework. More specifically, Uptane Design Overview Section 8.1 states ‘The root role serves as the certificate authority: it distributes and revokes the public keys used to verify metadata produced by each of these four roles (including itself).’ Additionally, the Uptane Implementation Specification Section 2.1 states ‘The root role serves as the certificate authority. It distributes and revokes the public keys used to verify metadata produced by each of the four basic roles (including itself).   
2. Examine Code for Key Revocation Process and Perform Key Revocation     
Both demo_director.py and demo_image_repo.py have a revoke_compromised_keys() method. Perform a key revocation for all roles (except root) on the director repository’s by running the following commands on the machine running the Uptane servers.      

- dd.revoke_compromised_keys()     


   Afterward request the updated metadata on both the Primary and Secondary by running the following commands:    
- dp.update_cycle()
- ds.update_cycle()

3. Verify Response    
Examine the temporary file structure created by the Secondary. The hierarchy looks similar to the following:     
       
temp_Secondary12345
                    

> unverified_targets

            - update.txt
         > unverified
                   > imagerepo
                            > metadata
                                     - timestamp.json
                                     - targets.json
                                     - snapshot.json
                                     - root.json
                   > director
                            > metadata
                                     - timestamp.json
                                     - targets.json
                                     - snapshot.json
                                     - root.json
         > metadata
                   > imagerepo
                            > previous
                                     - timestamp.json
                                     - targets.json
                                     - snapshot.json
                                     - root.json
                            > current
                                     - timestamp.json
                                     - targets.json
                                     - snapshot.json
                                     - root.json
                   > director
                            > previous
                                     - timestamp.json
                                     - targets.json
                                     - snapshot.json
                                     - root.json
                            > current
                                     - timestamp.json
                                     - targets.json
                                     - snapshot.json
                                     - root.json
         metadata_archive.zip
         update.txt
                                                            

Verify all of the metadata files under the director > current directory have a newer (greater) version number than the metadata in the director > previous directory.    
4. Attempt Unauthorized Key Revocation     
Create 4 sets of keys by performing the following actions in a python3 terminal.   
root_key = tuf.keys.generate_ed25519_key()
snapshot_key = tuf.keys.generate_ed25519_key()
timestamp_key = tuf.keys.generate_ed25519_key()
targets_key = tuf.keys.generate_ed25519_key()

Afterward, create a metadata directory that mimics the directory of a valid Primary. Copy the current metadata from the Primary into the previous directory. Open the metadata files in the current directory and increment all of the version numbers.
Then perform the following steps to create the ‘new’ root.json file:     
signable[‘signed’] = { [metadata] }
signabled[‘signatures’] = (tuf.sig.sign_over_metadata(root_key,signable['signed'])) open(‘root.json’,’w’).write(repr(signable)) print(tuf.hash.digest_filename('key_revocation_testing/modified/metadata/direct or/current/root.json').hexdigest()) 􏰁 <-used in snapshot.json 


Perform the previous steps for the other three metadata files: snapshot.json, timestamp.json, and targets.json.
Then setup an attacking machine that is can communicate with the Secondary. Replace the Primary’s connection to the Secondary with the attacking machine. Copy the previously created files to your current working directory. Run the following commands to mimic a Primary listening on port 30701:
- python key_revocation.py     

Run the following command to request an update on the Secondary:    
-ds.update_cycle()
Monitor the response of the Secondary and the temporary directory’s metadata to determine if the malicious key revocation was successful. The output from the Secondary looked similar to the following:     
>>> ds.update_cycle()
Timeserver attestation from Primary does not check out: This Secondary's nonce
was not found. Not updating this Secondary's time this cycle.
Verifying 'timestamp'. Requesting version: None
Downloading:
'file:////home/pi/workspace/uptane/temp_SecondarylRcju/unverified/imagerepo/met
adata/timestamp.json'
[TRUNCATED]
Update failed from
file:////home/pi/workspace/uptane/temp_SecondarylRcju/unverified/director/metad
ata/timestamp.json.
BadSignatureError
Failed to update timestamp.json from all mirrors:
{'file:////home/pi/workspace/uptane/temp_SecondarylRcju/unverified/director/met
adata/timestamp.json': BadSignatureError('timestamp',)}
Valid top-level metadata cannot be downloaded. Trying to update Root metadata
in case keys have changed for other metadata roles.
Verifying 'root'. Requesting version: None
Downloading:
'file:////home/pi/workspace/uptane/temp_SecondarylRcju/unverified/director/meta
data/root.json'
Downloaded 2120 bytes out of an upper limit of 512000 bytes.
Not decompressing
file:////home/pi/workspace/uptane/temp_SecondarylRcju/unverified/director/metad
ata/root.json
metadata_role: 'root'
Update failed from
file:////home/pi/workspace/uptane/temp_SecondarylRcju/unverified/director/metad
ata/root.json.
BadSignatureError
Failed to update root.json from all mirrors:
{'file:////home/pi/workspace/uptane/temp_SecondarylRcju/unverified/director/met
adata/root.json': BadSignatureError('root',)}
[TRUNCATED]    

5. Attempt Valid Key Revocation     
Afterward, sign the root.json metadata with the valid director root key to verify the Secondary is capable of handling a valid key revocation situation. The root.json file changes are highlighted in bold below:     
{
    "signatures": [
    {
     "keyid": "fdba7eaa358fa5a8113a789f60c4a6ce29c4478d8d8eff3e27d1d77416696ab2",
     "method": "ed25519",
     "sig":
   "4f6ccda358737474e4999f02e8943636422249c38d3a8ac23c5701105480f54e7a5cb8eeb5ee59
   be07d9e1d0f1535583f130bf1e7b7d715a5e490b46b50ccf0b"
}, {
     "keyid": "be24a45ed164dae69221a0cdb2031117f3b0ccc0df4aa0670441f18bbe30004d",
     "method": "ed25519",   
 "sig":
"9987484857320632338983f91d156e7b53482902e7d3d98bd764bc63cb3c849f0414d092c36153
fde17e59100dcc32fe8cbf3ba6d0123c543758ca475144fa03"
 }
 ],
 "signed": {
 "_type": "Root",
 "compression_algorithms": [
"gz" ],
 "consistent_snapshot": false,
 "expires": "2019-03-02T23:09:15Z",
 "keys": {
  "1d08cabb04831c3482df4e20bb648841530d060946e385bc1558fbc0f382d9d7": {
  "keyid_hash_algorithms": [
"sha256",
"sha512" ],
  "keytype": "ed25519",
  "keyval": {
   "public": "bbf9b7a7eb1b4693e2b9ece71186bc56d6b1fcb4682935c0708e416de1d08b22"
  }
  },
  "a3dc9c8deebeb63cf4bbccf2ab81834c94de582566dae42ce611fcff04f98693": {
  "keyid_hash_algorithms": [
"sha256",
"sha512" ],
  "keytype": "ed25519",
  "keyval": {
   "public": "9a02df2b0c0be3d7af000f34be257823a6c8a540b4fab747d877d14ad7563b19"
  }
  },
  "01aebb890a6bb3157eecbc02ce1e086a0c998729f03b7349b6d680de2b251b57": {
  "keyid_hash_algorithms": [
"sha256",
"sha512" ],
  "keytype": "ed25519",
  "keyval": {
   "public": "3a7a20e154d1744a389ef2eedbcedbeef3763a53a9ec80c21746c4a83dd7bf6c"
  }
  },
  "fdba7eaa358fa5a8113a789f60c4a6ce29c4478d8d8eff3e27d1d77416696ab2": {
  "keyid_hash_algorithms": [
"sha256",
"sha512" ],
  "keytype": "ed25519",
  "keyval": {
   "public": "f3b4c231520580eca92e17ae1581a708f606f72d43cc200af493afeec22a5e79"
  }
  },
  "be24a45ed164dae69221a0cdb2031117f3b0ccc0df4aa0670441f18bbe30004d": {
  "keyid_hash_algorithms": [
    "sha256",
    "sha512"
    ],
    "keytype": "ed25519",
    "keyval": {
   "public":
   "0a38cee58dcc3ab0a097bb36ab0da148639d985b50fae20ce7cbd69b3103bf81"
} }
}, "roles": {
     "root": {
     "keyids": [
      "fdba7eaa358fa5a8113a789f60c4a6ce29c4478d8d8eff3e27d1d77416696ab2",
      "be24a45ed164dae69221a0cdb2031117f3b0ccc0df4aa0670441f18bbe30004d"
     ],
     "threshold": 1
     },
     "snapshot": {
     "keyids": [
      "a3dc9c8deebeb63cf4bbccf2ab81834c94de582566dae42ce611fcff04f98693"
     ],
     "threshold": 1
     },
     "targets": {
     "keyids": [
      "1d08cabb04831c3482df4e20bb648841530d060946e385bc1558fbc0f382d9d7"
     ],
     "threshold": 1
     },
     "timestamp": {
     "keyids": [
      "01aebb890a6bb3157eecbc02ce1e086a0c998729f03b7349b6d680de2b251b57"
     ],
     "threshold": 1
} },
"version": 3
} }      
Run the following command to request updated metadata on the Secondary:   
- ds.update_cycle()   

Verify the Secondary has updated its metadata appropriately.
Since the Secondary only updated its metadata during a valid key revocation command, this test passed.

**Test Scripts**:key_revocation.py    
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
      # Define a function and register the response
def get_metadata(val1=False,val2=False,val3=False):
  with open('full_metadata_archive.zip','rb') as f:
    return f.read()
server.register_function(get_metadata,
      'get_metadata')     
      # Define a function and register the response
def update_exists_for_ecu(val1=False,val2=False,val3=False):
  return False
server.register_function(update_exists_for_ecu,
      'update_exists_for_ecu')      
      # Define a function and register the response
def get_image(val1=False,val2=False,val3=False):
  response=['Secondary.txt', b'v7 SECONDARY_ECU_1']
  return response
server.register_function(get_image,
      'get_image')       
      # Define a function and register the response
def submit_ecu_manifest(val1=False,val2=False,val3=False,val4=False):
  return ''
server.register_function(submit_ecu_manifest,
      'submit_ecu_manifest')
try:
  server.serve_forever()
except KeyboardInterrupt:
  print("\nKeyboard interrupt.")
  sys.exit(0)

   
   



  




