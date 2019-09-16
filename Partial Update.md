# Uptane Implementation Test Plan
## 13. Partial Update
**Test Case** : Interrupt the updating process to determine how the Uptane client responds.

**Test Steps** : 
1. Setup Devices  
Ensure the Uptane servers, Primary, and Secondary are up and running. Afterwards, push an update intended for the Secondary from the server to the Primary.  
2. Interrupt Update for Secondary  
Perform the following command from the Secondary to pull the update and applicable metadata from the Primary:  
-ds.update_cycle() 
Interrupt the traffic during transmission (e.g., removing Secondary’s connection) and monitor the output from the Secondary.
Verify the Secondary was not able to complete the download and presented an error to the screen similar to the following:  
>>> ds.update_cycle()
Verifying 'timestamp'.  Requesting version: None
Downloading:
[TRUNCATED...]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/pi/workspace/uptane/demo/demo_Secondary.py", line 373, in
update_cycle
    if not pserver.update_exists_for_ecu(Secondary_ecu.ecu_serial):
  File "/usr/lib/python3.5/xmlrpc/client.py", line 1092, in __call__
    return self.__send(self.__name, args)
  File "/usr/lib/python3.5/xmlrpc/client.py", line 1432, in __request
    verbose=self.__verbose
  File "/usr/lib/python3.5/xmlrpc/client.py", line 1134, in request
    return self.single_request(host, handler, request_body, verbose)
  File "/usr/lib/python3.5/xmlrpc/client.py", line 1146, in single_request
    http_conn = self.send_request(host, handler, request_body, verbose)
  File "/usr/lib/python3.5/xmlrpc/client.py", line 1259, in send_request
    self.send_content(connection, request_body)
  File "/usr/lib/python3.5/xmlrpc/client.py", line 1289, in send_content
    connection.endheaders(request_body)
  File "/usr/lib/python3.5/http/client.py", line 1103, in endheaders
    self._send_output(message_body)
  File "/usr/lib/python3.5/http/client.py", line 934, in _send_output
    self.send(msg)
  File "/usr/lib/python3.5/http/client.py", line 877, in send
    self.connect()
  File "/usr/lib/python3.5/http/client.py", line 849, in connect
    (self.host,self.port), self.timeout, self.source_address)
  File "/usr/lib/python3.5/socket.py", line 712, in create_connection
    raise err
  File "/usr/lib/python3.5/socket.py", line 703, in create_connection
    sock.connect(sa)
OSError: [Errno 113] No route to host
                                                                         
Afterwards, reconnect the Secondary to the network to determine if it would automatically attempt to reconnect and finish the update. Lastly, perform the following command to ensure the Secondary can download the update and has not entered a compromised state.  

-ds.update_cycle()

The Secondary did not attempt to reconnect automatically after regaining its connection. However, it was able to successfully download and install the update after regaining its connection.
  
 3. Interrupt Update for Primary  
  Prepare an update on the servers. Perform the following command on the Primary to pull the update and applicable metadata from the servers:   
- dp.update_cycle() 

Interrupt the traffic during transmission (e.g., removing Primary’s connection) and monitor the output from the Primary.
Verify the Primary was not able to complete the download and presented an error to the screen similar to the following:  
>>> dp.update_cycle()
Submitting a request for a signed time to the Timeserver.
[TRUNCATED...]
Downloading: 'http://192.168.1.100:30301/metadata/timestamp.json'
Could not download URL: 'http://192.168.1.100:30301/metadata/timestamp.json' URLError
Update failed from http://192.168.1.100:30301/metadata/timestamp.json. URLError
Failed to update timestamp.json from all mirrors: {'http://192.168.1.100:30301/metadata/timestamp.json': URLError(timeout('timed out',),)}
Valid top-level metadata cannot be downloaded. Trying to update Root metadata in case keys have changed for other metadata roles.
Verifying 'root'. Requesting version: None
Downloading: 'http://192.168.1.100:30301/metadata/root.json'
Could not download URL: 'http://192.168.1.100:30301/metadata/root.json' URLError
Update failed from http://192.168.1.100:30301/metadata/root.json.
URLError
Failed to update root.json from all mirrors: {'http://192.168.1.100:30301/metadata/root.json': URLError(timeout('timed out',),)}
Submitting the Primary's manifest to the Director.
Submission of Vehicle Manifest complete.   

Afterwards, reconnect the Primary to the network to determine if it would automatically attempt to reconnect and finish the update. Lastly, perform the following command to ensure the Primary can download the update and has not entered a compromised state.  

- dp.update_cycle() 

Neither the Primary nor Secondary attempted to reconnect automatically upon regaining their connection. However, they both successfully downloaded and installed the update when prompted after regaining their connection. This implies neither client entered a compromised state when experiencing the loss of their connection, therefore, this test passed.  



