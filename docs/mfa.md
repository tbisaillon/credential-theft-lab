<h1>Multi-Factor Authentication</h1>

### Duo Setup

- Create account
- Register device
- Set up application

### Firewall Configuration

In the **Device** tab, add a Multi Factor Authentication server profile:

  - **Profile Name**: DUO_MFA
  - **Certificate Profile**: Duo-Cert-Profile (certificates have been imported for you already)
  - **MFA Vendor**: Duo v2
  - **API Host**, **Integration Key**, and **Secret Key** are specific to your Duo account.

![Multi Factor Server Profile](img/mfa_server_profile.png)

Add a new **Authentication Profile** by cloning the existing LDAP_Auth profile.  Call it 
**LDAP_MFA**, and add the **DUO_MFA** profile to it as an additional factor.

![Authentication Profile](img/auth_profile.png)

Captive Portal is used to deliver the MFA prompt, so that needs to be configured next.  There
should be a self-signed certificate for the portal already created (CP-MFA), as well as a SSL/TLS
profile (CP_MFA_SSL) for you to use.

Configure Captive Portal for the GP tunnel interface as shown:

![Captive Portal](img/captive_portal.png)

We want GlobalProtect to prompt us to authenticate for applications that aren't web based, so we
need to add those settings.  In the Agent config section of the GlobalProtect portal configuration,
click on the **App** tab and include the following settings:

  - **Enable Inbound Authentication Prompts from MFA Gateways**: Yes
  - **Network Port for Inbound Authentication Prompts (UDP)**: 4501
  - **Trusted MFA Gateways**: 172.16.10.1

![GP MFA](img/gp_mfa.png)

In **Objects > Authentication**, create a new authentication object using the authentication 
profile.

![Authentication](img/authentication.png)

In the **Authentication** policy, create a rule to trigger the challenge for traffic destined for
our web server.  Set the timeout to a smaller value (1 minute) to make testing easier.

![MFA Firewall 6](img/mfa_firewall_6.png)

  - **Policy Name:** Require MFA
  - **Source Zone:** GP
  - **Destination Zone:** TRUST
  - **Destination Address:** 10.0.1.12
  - **Service:** service-http, service-rdp
  - **Authentication Enforcement:** Duo-Authentication

Finally, create a **Security** policy allowing traffic to the web server.

![MFA Firewall 7](img/mfa_firewall_7.png)

  - **Policy Name:** Lab MFA Test
  - **Source Zone:** GP
  - **Destination Zone:** TRUST
  - **Destination Address:** 10.0.1.12
  - **Application:** web-browsing, ms-rdp
  - **Service:** application-default
  - **Action:** allow

Commit the configuration.

### Verification

On the menu of your GlobalProtect client, click **Rediscover Network** to make sure it has the
latest configuration from the Portal.

Open a browser and connect to [http://web1.credlab.local](http://web1.credlab.local).  When 
directed to the captive portal page, accept the self-signed certificate warning prompt.

Log in to the captive portal page using the username and password corresponding to the user that 
you enrolled in Duo.  The first factor authentication will be done against the LDAP server.

![MFA Test 1](img/mfa_test_1.png)

Once the first factor has succeeded, the user is redirected to the second factor page.  In this 
lab, a push notification is sent to the user's device for the second factor authentication.  
On your device, either allow or deny the access.

![MFA Test 2](img/mfa_test_2.png)

On the firewall under **Monitor > Logs > Authentication**, you can view the authentication logs.
Note the two authentications, one for LDAP and one for Duo.

![MFA Test 3](img/mfa_test_3.png)  

If you opted to approve the authentication using the Duo app, the authentication will succeed and 
you will be redirected to the restricted site, which in our case is just the IIS default page.

![MFA Test 4](img/mfa_test_4.png)

You can also deliver authentication challenges for applications that are not web-based.  Open 
Remote Desktop and attempt to connect to web1.credlab.local.  The initial connection will fail, but
the GlobalProtect client will display a pop-up asking you to authenticate.  Click the link to open
the captive portal page, and authenticate as before.

![MFA Test 5](img/mfa_test_5.png)

After authenticating, attempt the Remote Desktop connection again and you should be able to connect
to the server.

![MFA Test 6](img/mfa_test_6.png)

You're done with this section of the lab.