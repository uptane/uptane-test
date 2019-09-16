# Uptane Implementation Test Plan
## 27. Replace ECU
**Test Case** : Replace an ECU on the vehicle to see if the vehicle will fail to authenticate for an update since the vehicle version manifest is different than what is expected by the inventory database (old ECU not present and new ECU may not be associated with vehicle). Note if any dependency resolution issues occur due to the new ECU being newer/older than the previous/replaced ECU.   

**Test Steps** :  
1. Setup Test Environment 
While monitoring the communication via Wireshark create two Primaries (p1, p2). Afterward, create two identical Secondaries (s1, s2) but assign them to only one of the two Primaries (i.e., s1-p1, s2-p2).  
2. Replace Primary  
Simulate replacing a Primary by opening a new terminal and running the following command using a vin and an ecu_serial that are already registered with the Uptane servers as seen below:  
>>>import demo.demo_Primary as dp
dp.clean_slate(vin=’111’,ecu_serial=’REPLACEMENT_PRIMARY’)   

Monitor the request and response via Wireshark and verify it looks like the following:  
>>>POST /RPC2 HTTP/1.1
   Host: 192.168.1.100:30501
   Accept-Encoding: gzip
   Content-Type: text/xml
   User-Agent: Python-xmlrpc/3.5
   Content-Length: 930
   <?xml version='1.0'?>
   <methodCall>
   <methodName>register_ecu_serial</methodName>
   <params>
   <param>
   <value><string>2_PRIMARY</string></value>
   </param>
   <param>
   <value><struct>
   <member>
   <name>keytype</name>
   <value><string>ed25519</string></value>
   </member>
   <member>
   <name>keyid</name>
   <value><string>9a406d99e362e7c93e7acfe1e4d6585221315be817f350c026bbee84ada260da
   </string></value>
   </member>
   <member>
   <name>keyval</name>
   <value><struct>
   <member>
   <name>public</name>
   <value><string>a1293426fcf4ce6f38135eb72bf89fedfdcba1b732779683b951d71a0b9e89a2
   </string></value>
   </member>
   </struct></value>
   </member>
   <member>
   <name>keyid_hash_algorithms</name>
   <value><array><data>
   <value><string>sha256</string></value>
   <value><string>sha512</string></value>
   </data></array></value>
   </member>
   </struct></value>
   </param>
   <param>
   <value><string>112</string></value>
   </param>
   <param>
   <value><boolean>1</boolean></value>
   </param>
   </params>
   </methodCall>
   HTTP/1.0 200 OK
   Server: BaseHTTP/0.6 Python/3.5.3
   Date: Sat, 24 Feb 2018 15:52:52 GMT
   Content-type: text/xml
   Content-length: 350
SwRI Project 10.21713 Version 3 May 31, 2018
                                    <?xml version='1.0'?>
   <methodResponse>
   <fault>
   <value><struct>
   <member>
   <name>faultString</name>
   <value><string>&lt;class 'uptane.Spoofing'&gt;:The given VIN, '112', is already
   associated with a Primary ECU.</string></value>
   </member>
   <member>
   <name>faultCode</name>
   <value><int>1</int></value>
   </member>
   </struct></value>
   </fault>
   </methodResponse>    
   
   
   As seen above, the Uptane servers respond stating an error when registering a duplicate Primary for a specific vehicle. However, attempting to perform an update_cycle() request from the Primary afterward, shows that the Primary was able to successfully download metadata from the servers. This highlights an inconsistency between the servers detecting a spoofed ECU registration yet still allowing the rogue Primary to download metadata.
Analyzing the code reveals that if the Primary fails the register_self_with_director() then it assumes that the Primary is already registered, but does not prevent the Primary from any future commands or functionality.
This implies that there should not be any issues experienced when replacing a Primary ECU.   
3. Replace Secondary  
Afterward attempt to replace a Secondary with a similar one on a different Primary. This simulates the process of swapping ECU’s from decommissioned vehicles into a running vehicle. This is done by first killing the process of the Secondary being replaced, and running the following commands on the Secondary that will be the replacement:   
>>>ds._vin = 112
ds._Primary_port = 30702
ds.register_self_with_Primary()
ds.update_cycle()   

Monitor the output on the Secondary and verify it looks similar to the following:
>>> ds.update_cycle()
Timeserver attestation from Primary does not check out: This Secondary's nonce
was not found. Not updating this Secondary's time this cycle.
Verifying 'timestamp'. Requesting version: None
[TRUNCATED]
Delivered target file has been fully validated:
'/home/pi/workspace/uptane/temp_Secondary0InMp/unverified_targets/1_Secondary.t
xt'
Installed firmware received from Primary that was fully validated by the
Director and
                   Image Repo. Image: '1_Secondary.txt'
The contents of the newly-installed firmware with filename '/1_Secondary.txt'
are:
v1 1_SECONDARY.txt
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
 File "/home/pi/workspace/uptane/demo/demo_Secondary.py", line 519, in
update_cycle
  submit_ecu_manifest_to_Primary()
 File "/home/pi/workspace/uptane/demo/demo_Secondary.py", line 251, in
submit_ecu_manifest_to_Primary
  signed_ecu_manifest)
 File "/usr/lib/python3.5/xmlrpc/client.py", line 1092, in __call__
  return self.__send(self.__name, args)
 File "/usr/lib/python3.5/xmlrpc/client.py", line 1432, in __request
  verbose=self.__verbose
 File "/usr/lib/python3.5/xmlrpc/client.py", line 1134, in request
  return self.single_request(host, handler, request_body, verbose)
 File "/usr/lib/python3.5/xmlrpc/client.py", line 1150, in single_request
  return self.parse_response(resp)
 File "/usr/lib/python3.5/xmlrpc/client.py", line 1322, in parse_response
  return u.close()
 File "/usr/lib/python3.5/xmlrpc/client.py", line 655, in close
  raise Fault(**self._stack[0])
xmlrpc.client.Fault: <Fault 1: "<class 'uptane.UnknownVehicle'>:Received an ECU
Manifest supposedly hailing from a different vehicle....">
>>>   

Note, since an update was already pushed to the VIN for the associated Secondary, the newly registered Secondary successfully downloaded and installed the update, even though it received errors throughout the update process.
Although registering a duplicate Primary and Secondary produces error messages, it still allows for functionality (i.e., downloading of images and metadata) from the servers without an issue. Since the replacement of an ECU is possible through the reference implementation, this test passed.   


