# Uptane Implementation Test Plan
# 4. Certificate Checking
**Test Case** : Determine if the Uptane client is verifying the certificate of the server before communicating and sending sensitive information.

**Test Steps**     
1. Setup Attacking Machine     
Setup an attacking machine to be configured on the same network as the Uptane server and the Uptane Primary. Monitor the communication occurring between the Uptane server and Primary. Afterward, remove the communication channel of the Uptane server. Next, use the attacking machine to spoof the network and appear as if it is the Uptane server (this includes running http/XMLRPC servers on the appropriate ports).       
2. Attempt to Retrieve Sensitive Information from Primary     
Perform normal Primary functions such as:       
update_cycle()       
generate_signed_vehicle_manifest()         
submit_vehicle_manifest_to_director()
3. Monitor Output
Monitor if the Primary verifies the server before communicating sensitive information with it (e.g., vehicle manifests, registering ecu’s, etc.). The Primary does not verify the server’s identity, and instead, sends data to the hardcoded IP in its memory. Since an attacker can successfully man-in-the-middle between the Primary and the server, the Primary is vulnerable to sending sensitive information to an unauthorized party. Therefore, this test failed.





