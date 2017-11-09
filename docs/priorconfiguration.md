<h1>What's Been Done For You</h1>

**Note:** This section is included for reference, and no action is needed on your part.

## Read Only Domain Controller

User-ID Agent and User-ID Agent Credential service have already been installed on the RODC.  The 
following screen capture shows that the *"Import from User-ID Credential Agent"* is selected under 
the Credentials tab in the User-ID Agent GUI.  This enables the User-ID agent to import the bloom
filter that the User-ID credential agent creates to represent users and the corresponding password 
hashes.

Users that should receive credential submission enforcement have been added to the "Allowed RODC 
Password Replication Group".

## GoPhish On Kali Linux

The [GoPhish](https://getgophish.com) tool has been set up on the Kali Linux instance to simulate 
a phishing campaign.  Our campaign will use a fake Palo Alto Networks SSO landing page at 
**paloaito.sso.com** to capture domain credentials from our test users.	