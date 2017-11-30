<h1>Credential Phishing Prevention</h1>

There are 3 methods to check for corporate credential submissions that you can use on the firewall
to detect users when they submit credentials to web pages.

The three methods are:

1. Group Mapping
2. IP User Mapping
3. Domain Credential Filter

In this lab, you will configure the firewall to check submitted credentials via the **Domain 
Credential Filter** method.

---

<h1>Configuration Steps</h1>

### Phishing Campaign Configuration

From your laptop, open the GlobalProtect Agent.  If you don't have it installed, install it when 
prompted by the portal.  Connect to your lab environment's GlobalProtect portal using any of the 
user accounts listed in Table 2.

Access your phishing campaign admin site with the credentials **admin/gophish**.

![Phishing Campaign 1](img/phishing_campaign_1.png)

In the **Users & Groups** tab, edit the **Phishing Recipients** 

![Phishing Campaign 2](img/phishing_campaign_2.png)

Change the recipient to be your test email account.

![Phishing Campaign 3](img/phishing_campaign_3.png)

Now, in the **Campaigns** tab, click the **New Campaign** button.

![Phishing Campaign 4](img/phishing_campaign_4.png)

Fill out the campaign as shown, then click **Launch Campaign**.  Pay particular attention to the 
misspelled domain name **paloaito.sso.com** that we are spoofing!

![Phishing Campaign 5](img/phishing_campaign_5.png)

GoPhish will queue and send the emails.  You can refresh this page and check that an email has been
sent to your test user.

![Phishing Campaign 6](img/phishing_campaign_6.png)

If you want, log into your test email account, and verify that you've received your phishing email.

![Phishing Campaign 7](img/phishing_campaign_7.png)

---

### Firewall Configuration

Log in to the GUI of your firewall via the public management IP, and add a User-ID agent.

   - **Name:** RODC
   - **Host:** 10.0.1.11
   - **Port:** 5007
   - Make sure the **Enabled** check box is ticked.

![User-ID Agent](img/uid_agent.png)

**Note:** The service route configuration of the firewall has already been modified to communicate
with the User-ID agent via the firewall's trust interface.  To confirm it has been set up correctly,
under **Device > Setup > Services**, select **Service Route Configuration** and choose 
**Customize**.  In the IPv4 tab, select UID Agent service and verify that it is the interface in 
the *trust* zone (ethernet1/2).

Commit your changes.  After the commit completes, go back to the User-ID Agents tab.  The connected
column will change from orange to green.

![User-ID Agent 2](img/uid_agent_2.png)

To make sure our test URL is categorized the way we want it, create a custom URL category for the 
URL **paloaito.sso.com**.  *(Note the misspelling!)*

![Custom URL Category](img/custom_category.png)

Select **Objects > Security Profiles > URL Filtering** and select the default URL Filtering profile.
Clone it, and change the cloned profile name to "credential-phish-block".  Use the Categories tab 
to set all URL categories to alert, but to block credential submissions.  

![Credential Phish Profile 1](img/credential_phish_profile_1.png)

Now go to the **User Credential Detection** tab.  Choose **Use Domain Credential Filter** for the 
User Credential Detection method, and set the log severity to "high".

![Credential Phish Profile 2](img/credential_phish_profile_2.png)

Go to the **Policies > Security** tab, and create a new security policy.  Associate your new URL
filtering profile to the security policy with traffic from the GP zone to the PHISH zone.

  - **Policy Name:** Allow to Intranet
  - **Source Zone:** GP
  - **Destination Zone:** PHISH, TRUST
  - **Application**: any
  - **Service:** application-default
  - **Action:** allow
  - **URL Profile:** credential-phish-block

![Credential Phish Security Policy](img/credential_phish_sec_policy.png)

Commit the configuration.

---

### Verification

Log into your test email account.  You should see that a phishing email has been received.  The
email contains a link that will open a phishing webpage mimicking the Palo Alto Networks corporate
SSO page.

Upon clicking the link provided in the email, you will get a phishing page with a form to submit
user credentials.

![Credential Theft Test 1](img/credential_phish_test_1.png)


**Note:** If you are NOT connected via GlobalProtect to your lab environment, you will get an 
"under construction" web page similar to below.  Do not proceed until you get the test phishing 
page.

Enter a username **not** in Table 2, and any password you'd like.  Click **Login**.

![Credential Theft Test 2](img/credential_phish_test_2.png)

At this point, those credentials have been phished.  You will be redirected to a legitimate site.

![Credential Theft Test 3](img/credential_phish_test_3.png)

Now log in to your credential phishing campaign's admin web site.

![Credential Theft Test 4](img/credential_phish_test_4.png)

Next to the PANW campaign, click the View Results icon in the bottom right corner to get the 
campaign details.  Scroll down to your username, click the triangle icon, and expand the details:

Scroll down to the bottom "Submitted Data" section, and click the triangle icon next to "View 
Details".  You will see the username and password you entered.

![Credential Theft Test 5](img/credential_phish_test_5.png)

The above steps you performed show how easy it is to steal user credentials.  Credential phishing
is rampant.  It is easier than ever to phish a user for credentials.  This is especially true in
targeted attacks, where the attacker wants to get into a specific organization.  A phishing attack
makes password complexity irrelevant; the attacker steals the password no matter how difficult it
is.

You will do a second test, but first, let's find out what usernames are defined on your firewall.
Connect to your firewall via SSH, and run the `show user ip-user-mapping all` command.

![Credential Theft Test 6](img/credential_phish_test_6.png)

Confirm that your username is listed (it will be if you are connected to the lab environment via 
GlobalProtect).  You may see additional IP/user mappings.

Go back to your test phishing account, and open the phishing email.  By clicking the link provided
in the email, you will get to the phishing page again.

![Credential Theft Test 7](img/credential_phish_test_7.png)

Try submitting true "corporate" credentials by submitting the credentials you are connected to 
GlobalProtect with.

Since the source IP address of a session had a known User-ID, and the HTTP POST parameter in that
session matched the password belonging to that User-ID, the firewall detected a credential 
submission and the URL will be blocked.

You will get this message:

![Credential Theft Test 8](img/credential_phish_test_8.png)

You can see more info about this process using these commands:

* `show user user-id-agent state all`
* `less mp-log useridd.log` - You can find detailed logs in useridd.log.  This log now includes 
  credential related logs, as well as bloom filter updates.

In the output of `show user user-id-agent state all`, look for the following:

![Credential Theft Test 9](img/credential_phish_test_9.png)

Now examine the firewall's URL filtering logs.  Locate the log that matches your username, and view
the log details.  Look for the "credential detected" flag.

![Credential Theft Test 10](img/credential_phish_test_10.png)

You can close the browser tabs to Gmail and Gophish.  You're done with this section of the lab.