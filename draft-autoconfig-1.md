---
title: "Mail Autoconfig"
abbrev: "autoconfig"
category: info

docname: draft-bucksch-autoconfig
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 1
# area: AREA
# workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "benbucksch/autoconfig-spec"
  latest: "https://benbucksch.github.io/autoconfig-spec/draft-autoconfig-1.html"

author:
 -
    fullname: "Ben Bucksch"
    organization: Beonex
    email: "ben.bucksch@beonex.com"

normative:

informative:


--- abstract

Set up a mail account with only email address and password.

--- middle

# Introduction

This protocol allows users to set up their existing email account in a new mail client application,
by entering only their name, email address, and password.
The mail application, by means of mail autoconfig specified here, will determine all the other
parameters that are required, including IMAP or POP3 hostname, TLS configuration,
form of username, authentication method, and other settings, and likewise for SMTP.
Contact sync and calendar, file sharing and other services can also be set up automatically.

The protocol works by first determining the domain from the email address, and the querying
well-known URLs at the email provider, which return the configuration parameters in computer-readable form. Failing that, various fallback sources can be applied, like a common database of
configurations for large email providers who do not directly support this protocol,
or other mechanisms to determine the configuration.

# Implementations

This protocol is in production use since 15 years by major email clients, and the
config database (used as fallback) contains configurations for over 50% of all email accounts.

Currently, this protocol or parts of it has been implemented by:

* [Thunderbird](https://thunderbird.net)
* [Evolution](https://projects.gnome.org/evolution/)
* [KMail](https://userbase.kde.org/KMail)
* [Kontact](https://www.kontact.org)
* [K9 Mail](https://k9mail.app)
* [FairEmail](https://email.faircode.eu)
* [NextCloud email](https://apps.nextcloud.com/apps/mail)
* [Delta Chat](https://delta.chat/)

and likely other mail clients.

The purpose of this paper is to document and specify what is deployed in the wild. A later version 2 of the protocol might be based on JSON.

Additionally, there are known problems with OAuth2 in combination with mail clients, which would need to be solved by another specification.

# Data format

Whether the ISP or a common central database returns the configuration, the resulting
document has the following data format.

The MIME type is `text/xml` or `text/xml+autoconfig`.

## Sample config file

    <?xml version="1.0"?>
    <clientConfig version="1.1">
        <emailProvider id="example.com">
          <domain>example.com</domain>
          <domain>example.net</domain>

          <displayName>Google Mail</displayName>
          <displayShortName>GMail</displayShortName>

          <!-- type=
              "imap": IMAP
              "pop3": POP3
              "jmap": JMAP
              -->
          <incomingServer type="pop3">
            <hostname>pop.example.com</hostname>
            <port>995</port>
              <!-- "plain": no encryption
                    "SSL": SSL 3 or TLS 1 on SSL-specific port
                    "STARTTLS": on normal plain port and mandatory upgrade to TLS via STARTTLS
                    -->
            <socketType>SSL</socketType>
            <username>%EMAILADDRESS%</username>
                <!-- Authentication methods:
                    "password-cleartext",
                              Send password in the clear
                              (dangerous, if SSL isn't used either).
                              AUTH PLAIN, LOGIN or protocol-native login.
                    "password-encrypted",
                              A secure encrypted password mechanism.
                              Can be CRAM-MD5 or DIGEST-MD5. Not NTLM.
                    "NTLM":
                              Use NTLM (or NTLMv2 or successors),
                              the Windows login mechanism.
                    "GSSAPI":
                              Use Kerberos / GSSAPI,
                              a single-signon mechanism used for big sites.
                    "client-IP-address":
                              The server recognizes this user based on the IP address.
                              No authentication needed, the server will require no username nor password.
                    "TLS-client-cert":
                              On the SSL/TLS layer, the server requests a client certificate and the client sends one (possibly after letting the user select/confirm one), if available.
                    "OAuth2":
                              OAuth2. Works only on specific hardcoded servers, please see below. Should be added only as second alternative.
                    "none":
                              No authentication
                    -->
            <authentication>password-cleartext</authentication>
            <pop3>
                <!-- remove the following and leave to client/user? -->
                <leaveMessagesOnServer>true</leaveMessagesOnServer>
                <downloadOnBiff>true</downloadOnBiff>
                <daysToLeaveMessagesOnServer>14</daysToLeaveMessagesOnServer>
                <!-- only for servers which don't allow checks more often -->
                <checkInterval minutes="15"/><!-- not yet supported -->
            </pop3>
            <password>optional: the user's password</password>
          </incomingServer>

          <!-- You can have multiple incoming servers,
            and even multiple IMAP server configs.
            The first config is the preferred one, but the user or
            or client can choose the alternative configs. -->
          <incomingServer type="jmap">
            <hostname>jmap.example.com</hostname>
            <port>443</port>
            <socketType>SSL</socketType>
            <username>%EMAILADDRESS%</username>
                <!-- Authentication methods:
                    "password-cleartext",
                              Send password in the clear
                              (dangerous, if SSL isn't used either).
                              AUTH PLAIN, LOGIN or protocol-native login.
                    "password-encrypted",
                              A secure encrypted password mechanism.
                              Can be CRAM-MD5 or DIGEST-MD5. Not NTLM.
                    "NTLM":
                              Use NTLM (or NTLMv2 or successors),
                              the Windows login mechanism.
                    "GSSAPI":
                              Use Kerberos / GSSAPI,
                              a single-signon mechanism used for big sites.
                    "client-IP-address":
                              The server recognizes this user based on the IP address.
                              No authentication needed, the server will require no username nor password.
                    "TLS-client-cert":
                              On the SSL/TLS layer, the server requests a client certificate and the client sends one (possibly after letting the user select/confirm one), if available.
                    "OAuth2":
                              OAuth2. Works only on specific hardcoded servers, please see below. Should be added only as second alternative.
                    "none":
                              No authentication
                    -->
            <authentication>password-cleartext</authentication>
            <pop3>
                <!-- remove the following and leave to client/user? -->
                <leaveMessagesOnServer>true</leaveMessagesOnServer>
                <downloadOnBiff>true</downloadOnBiff>
                <daysToLeaveMessagesOnServer>14</daysToLeaveMessagesOnServer>
                <!-- only for servers which don't allow checks more often -->
                <checkInterval minutes="15"/><!-- not yet supported -->
            </pop3>
            <password>optional: the user's password</password>
          </incomingServer>

          <outgoingServer type="smtp">
            <hostname>smtp.googlemail.com</hostname>
            <port>587</port>
            <socketType>STARTTLS</socketType> <!-- see <incomingServer> -->
            <username>%EMAILADDRESS%</username> <!-- if smtp-auth -->
                <!-- smtp-auth (RFC 2554, 4954) or other auth mechanism.
                    For values, see incoming.
                    Additional options here:
                    "SMTP-after-POP":
                        authenticate to incoming mail server first
                        before contacting the smtp server.
                -->
            <authentication>password-cleartext</authentication>
            <password>optional: the user's password</password>
                <!-- If the server makes some additional requirements beyond <authentication>:
                    "client-IP-address": The server is only reachable or works,
                        if the user is in a certain IP network, e.g.
                        the dialed into the ISP's network (DSL, cable, modem) or
                        connected to a company network.
                        Note: <authentication>client-IP-address</>
                        means that you may use the server without any auth.
                        <authentication>password-cleartext</> *and*
                        <restriction>client-IP-address</> means that you need to
                        be in the correct IP network *and* (should) authenticate.
                        Servers which do that are highly discouraged and
                        should be avoided, see {{bug|556267}}.
                    Not yet implemented. Spec (element name?) up to change.
                -->
            <restriction>client-IP-address</restriction>
            <!-- remove the following and leave to client/user? -->
            <addThisServer>true</addThisServer>
            <useGlobalPreferredServer>true</useGlobalPreferredServer>
          </outgoingServer>

          <!-- A page where the ISP describes the configuration.
              This is purely informational and currently mainly for
              maintenance of the files and not used by the client at all.
              Note that we do not necessarily use exactly the config suggested
              by the ISP, e.g. when they don't recommend SSL, but it's available,
              we will configure SSL.
              The text content should contains a description in the native
              language of the ISP (customers), and a short English description,
              mostly for us.
          -->
          <documentation url="http://www.example.com/help/mail/">
            <descr lang="en">Configure mail app for IMAP</descr>
            <descr lang="de">Email mit IMAP konfigurieren</descr>
          </documentation>

        </emailProvider>

        <!-- Syncronize the user's address book / contacts. -->
        <addressBook type="carddav">
          <username>%EMAILADDRESS%</username>
            <!-- Authentication methods. See also <incomingServer>.
                  "http-basic":
                            Authenticate to the HTTP server using
                            WWW-Authenticate: Basic
                  "http-digest":
                            Authenticate to the HTTP server using
                            WWW-Authenticate: Digest
                  "OAuth2":
                            OAuth2. Uses the same token as for email. <scope> needs to include
                             addressbook/calendar.
                  -->
          <authentication>http-basic</authentication>
          <serverURL>https://jmap.example.com/remote.php/dav<serverURL>
        </addressBook>

        <addressBook type="jmap">
          <username>%EMAILADDRESS%</username>
          <authentication>http-basic</authentication>
          <serverURL>https://jmap.example.com<serverURL>
        </addressBook>

        <!-- Syncronize the user's calendar. -->
        <calendar type="caldav">
          <username>%EMAILADDRESS%</username>
          <authentication>http-basic</authentication> <!-- see <addressBook> -->
          <serverURL>https://calendar.example.com/remote.php/dav<serverURL>
        </calendar>

        <calendar type="jmap">
          <username>%EMAILADDRESS%</username>
          <authentication>http-basic</authentication> <!-- see <addressBook> -->
          <serverURL>https://calendar.example.com<serverURL>
        </calendar>

        <!-- Upload files, allowing the user to share them.
            This can be used for Thunderbird's FileLink feature,
            or to set up a file sync folder on the user's desktop. -->
        <fileShare type="webdav">
          <username>%EMAILADDRESS%</username>
          <authentication>http-basic</authentication> <!-- see <addressBook> -->
          <serverURL>https://share.example.com/remote.php/dav<serverURL>
        </fileShare>

        <!-- This allows to login in to the webmail service of the provider.
            The URLs are loaded into a standard webbrowser for the user.
            This is optional. -->
        <webMail>
          <!-- Webpage where the user has to log in manually by entering username
              and password himself.
              HTTPS required. -->
          <loginPage url="https://mail.example.com/login/" />

          <!-- Same as loginAutomaticDOM, but the website makes checks that
              the user comes from the login page. So, open the login page
              in the browser, get the page's DOM, fill out name and password
              fields for the user, and trigger the login button.
              The login button might not be an HTML button, just a div, so
              to trigger it, send a click event to it.
              HTTPS is required for the URL. -->
          <loginPageInfo url="https://mail.example.com/login/">
            <!-- What to fill into the usernameField.
                Format is the same as for <username> within <incomingServer>,
                including placeholders. See below for valid placeholders. -->
            <username>%EMAILADDRESS%</username>
            <!-- Allows to find the textfield on the page, to fill it out.
                The id attribute give the DOM ID,
                The name attribute give the DOM name attribute.
                One or both of id and name attributes must exist.
                Try the ID first (e.g. using getElementById()), if existing.
                Otherwise, try finding the element by name.
                Don't treat the IDs given in this XML file as trusted,
                but before using them, verify the format
                (e.g. only characters and digits for IDs).
                If you use powerful functions like jQuery, and the XML returns
                you code in the username ID, and you feed it unchecked to jQuery,
                it may be executed. -->
            <usernameField id="email_field" name="email" />
            <passwordField name="password" />
            <!-- The submit button to trigger the server submit
                after filling in the fields.
                id and name attributes: See <usernameField> -->
            <loginButton id="submit_button" name="login"/>
          </loginPageInfo>
        </webMail>

        <!-- Ask user for custom input,
           and use them as placeholders in the values.
           Optional. -->
        <inputField key="USERNAME" label="Screen name"></inputField>
        <inputField key="GRANDMA" label="Grandma">Elise Bauer</inputField>

        <!-- oAuth2 specced for mail apps,
            e.g. clientID, expiry, and login page -->
        <mAuth>
          <authURL>https://login.example.com/common/oauth2/v2.0/authorize</authURL>
          <tokenURL>https://login.example.com/common/oauth2/v2.0/token</tokenURL>
          <issuer>login.example.com</issuer>
          <scope>imap pop3 smtp webdav caldav carddav offline_access</scope>
          <clientID>autoconfig</clientID>
        </mAuth>

        <!-- Add this only when users (who already have an account) have to
            do something manually before the account can work with IMAP/POP or SSL.
            Note: Per XML, & (ampersand) needs to be escaped to
            & a m p ; (without spaces).
            Mandatory only if the ISP requires such settings
            before the configs above work. -->
        <enable
          visiturl="https://mail.google.com/mail/?ui=2&shva=1#settings/fwdandpop">
          <instruction>Check 'Enable IMAP and POP' in Google settings page</instruction>
          <instruction lang="de">Schalten Sie 'IMAP und POP aktivieren' auf der Google Einstellungs-Seite an</instruction>
        </enable>

        <clientConfigUpdate url="https://www.example.com/config/mail.xml" />

    </clientConfig>

## Formal definition

TODO Schema for XML

# Config retrieval for mail clients

The mail client application, when it needs the configuration for a given email address,
will perform several steps to retrieve the configuration from various sources.

The steps are ordered by priority. They may all be requested at the same time, but a higher priorty
result that is available MUST be preferred over a lower priority one, even if the lower priority one is available earlier. Lower priority requests MAY be cancelled, if a valid higher priority result has been successfully received. The priority is expressed below with the number before the URL or location, with lower numbers meaning higher priority, e.g. 1.2 has higher priority than 4.1.

In the URLs below,`%EMAILADDRESS` shall be replaced with the email address that the user entered and wishes to use, and `%EMAILDOMAIN%` shall be replaced with the email domain extracted from the email address. For example, for "fred@example.com", the email domain is "example.com", and for "fred@test.cs.example.net", the email domain is "test.cs.example.net".

For full support of this specification, all "Required" and "Recommended" mechanisms MUST be implemented and working. For partial support of this specification, all "Required" mechanisms MUST be implemented and working, and in this case, you shall make explicit when advertizing or referring to auto config that there is only partial support of this specification.


## Mail provider

First step is to directly ask the mail provider and allow it to return the configuration. This step ensures that the protocol is decentralized and the mail provider is in control of the configuration issued to mail clients.

Mail providers MUST return a working configuration, with state-of-the-art security. Configuration fields MUST NOT contain invalid or non-working configuration data.

To allow the mail provider to return a configuration adjusted for that mailbox, the client sends the email address as query parameter. This allows the mail provider to e.g. separate mailboxes on geographically local mail servers, e.g. a mail server located in the same office building where an employee works. However, while the protocol allows for such heterogenous configurations, mail providers are discouraged from doing so, and are instead encouraged to provide one single configuration for all their users. For example, DNS resolution based on location, mail proxy servers, or other techniques as necessary, can be used to route the traffic and host the mail efficiently.

* 1.1. `https://autoconfig.%EMAILDOMAIN%/.well-known/mail-v1.xml?emailaddress=%EMAILADDRESS%` (Required)
* 1.2. `https://%EMAILDOMAIN%/.well-known/autoconfig/mail/config-v1.1.xml` (Optional)
* 1.3. `http://autoconfig.%EMAILDOMAIN%/mail/config-v1.1.xml` (Optional)

For example:

* 1.1. https://autoconfig.example.com/.well-known/mail-v1.xml?emailaddress=fred@example.com
* 1.2. https://example.com/.well-known/autoconfig/mail/config-v1.1.xml
* 1.3. http://autoconfig.example.com/mail/config-v1.1.xml

Step 1.3. is mainly for legacy servers. Many current deployments
use this HTTP URL.

## Central database

The ISPDB contains the configurations for most mail providers with a market share larger than 0.1%, and contains configurations for half of the email accounts in the world.

This is a useful fallback for mail providers which do not host a config server described in the previous step. Using a central database (ISPDB) of mail configurations for the large mail providers will increase the success rate of finding a valid configuration drastically, up to 10-fold.

The mail client application may choose the mail config database provider. A public mail config database is available at base URL `https://autoconfig.ispdb.net/v1.1/`.

`%ISPDB%` below is the base URL of that database.

* 2.1. `%ISPDB%%EMAILDOMAIN%` (Recommended)

For example:

* 2.1. [https://autoconfig.ispdb.net/v1.1/geologist.com](https://autoconfig.ispdb.net/v1.1/geologist.com)


## MX

Many companies do not maintain their own mail server, but let their email be hosted by a hosting company, which is then responsible for the email of dozens or thousands of domains. For these hosters, it may be difficult to set up the configuration server (step 1.1.) with valid TLS certificate for each of their customers, and to convince their customers to modify their root DNS specifically for autoconfig. On the other side, the ISPDB can only contain the hosting company and cannot know all their customers. To handle such domains, the protocol first needs to find the server hosting the email.

If the previous mechanisms yield no result, the client may perform a DNS MX lookup on the email domain, and retrieve the MX server (incoming email server) for that domain. Only the highest priority MX hostname is considered. From that MX hostname, 2 values are extracted:

* Remove the first component from the MX hostname, i.e. everything up to and including the first `.`, and use that as value for `%MXFULLDOMAIN%`.
* Extract only the second-level domain from the MX hostname, and use that as value for `%MXBASEDOMAIN%`. To determine the second-level domain, use the [Public Suffic List](https://publicsuffix.org) or a similarly suited method, to correctly handle domains like ".co.uk" and ".com.au".

For example:

 * For "mx.example.com", the MXFULLDOMAIN and MXBASEDOMAIN are both "example.com".
 * For "mx.example.co.uk", the MXFULLDOMAIN and MXBASEDOMAIN are both "example.co.uk".
 * For "mx.premium.europe.example.com", the MXFULLDOMAIN is "premium.europe.example.com" and the MXBASEDOMAIN is "example.com".

Then, attempt to retrieve the config for these MX domains, using the previous methods:

* 3.1. `https://autoconfig.%MXFULLDOMAIN%/.well-known/mail-v1.xml?emailaddress=%EMAILADDRESS%` (Required)
* 3.2. `https://autoconfig.%MXBASEDOMAIN%/.well-known/mail-v1.xml?emailaddress=%EMAILADDRESS%` (Recommended)
* 3.3. `%ISPDB%%MXFULLDOMAIN%` (Recommended)
* 3.4. `%ISPDB%%MXBASEDOMAIN%` (Recommended)

For example:

* 3.1. https://autoconfig.premium.europe.example.com/.well-known/mail-v1.xml?emailaddress=fred@example.com
* 3.2. https://autoconfig.example.com/.well-known/mail-v1.xml?emailaddress=fred@example.com
* 3.3. https://autoconfig.ispdb.net/v1.1/premium.europe.example.com
* 3.4. https://autoconfig.ispdb.net/v1.1/example.com


## Local disk

For testing purposes, you may want to define a location on the disk, relative to the application installation directory, or relative to the user configuration directory, which may contain a configuration file for a specific domain, and which your application will use, if the above methods fail.

* 4.1. `%USER_CONFIGURATION_DIR%/isp/%EMAILDOMAIN%.xml` (Optional)
* 4.2. `%APP_INSTALL_DIR%/isp/%EMAILDOMAIN%.xml` (Optional)

For example:

* 4.1. /home/fred/.config/yourapp/isp/example.com.xml
* 4.2. /opt/yourapp/isp/example.com.xml


## Other mechanisms

You may want to implement other mechanisms to find a configuration, for example Exchange AutoDiscover, DNS SRV, or heuristic guessing. If you implement such alternative methods, and if they are less secure than some of the mechanisms provided here, the alternative methods SHOULD be considered only with lower priority (as defined above) than the more secure mechanisms defined here. For evaluating other mechanisms, use similar criteria as outlined in section "Security considerations".


## Manual configuration

If the above mechanisms fail to provide a working configuration, or if the user explicitly chooses so, you SHOULD give the end user the ability to manually enter a configuration, and use that configuration to configure the account.


# Config validation

## User approval

Independent of the mechanisms used to find the configuration, before using that configuration, you SHOULD display that configuration to the end user and let him confirm it. While doing so:

* At least the second-level domain name(s) of the hostnames involved MUST be shown clearly and with high prominence.
* The client MUST NOT cut off parts of long second-level domains, to avoid spoofing. At least 63 characters MUST be displayed.
* Care SHOULD be taken with international characters that look like ASCII characters, but have a different code.

## Login testing

After the user confirmed the configuration, you SHOULD test the configuration, by attempting a login to each server configured. Only if the login succeeded, and the server is working, should the configuration be saved and retrieving and sending mail be started.

## OAuth2 windows

If the configuration contains OAuth2 authentication, or any other authentication that uses a web browser with URL redirects, you MUST show the full URL or the second-level domain of the current page to the end user, at all times, including after page changes, URL changes or redirects. This allows the end user to verify that he is logging in at the expected page, e.g. the login server of their company.

(Editor's note: Not really part of autoconfig, but autoconfig can enable OAuth2, and it's important that this is implemented correctly by mail applications.)

# Config publishing for mail providers

Mail service providers who want to support this specification
and publish the mail configuration for their own mail service,
so that mail client apps can be automatically configured,
SHOULD follow this section as guideline and MUST respect the
definitions in this specification.

## Config location for single domain

The preferred location to publish the configuration file is
step 1.1. above, i.e.
`https://autoconfig.%EMAILDOMAIN%/.well-known/mail-v1.xml?emailaddress=%EMAILADDRESS%`
e.g. for fred@example.com:
https://autoconfig.example.com/.well-known/mail-v1.xml?emailaddress=fred@example.com
For backwards compatibility, step 1.2. should also be implemented.

## Config location based on MX server

For mail providers which host entire domains for their customers,
the same URL is still preferred. Alternatively and less preferred,
the location from step 3.1. above should be used, i.e.
`https://autoconfig.%MXFULLDOMAIN%/.well-known/mail-v1.xml?emailaddress=%EMAILADDRESS%`
E.g. if the MX server for customer domain example.net is "mx.premium.europe.example.com", then the config file should be at
https://autoconfig.premium.europe.example.com/.well-known/mail-v1.xml?emailaddress=fred@example.net
For backwards compatibility, step 3.2. should also be implemented.

## No authentication for config

Any of the above URLs for retrieving the config file MUST NOT
require authentication, but MUST be public.

This is because the config contains the authentication method.
Without knowing the config, the client does not know which
authentication method is required and which username form
(e.g. username "fred" or "fred@example.com" or "fred\EXAMPLE")
to use. Given that the config is required for authentication,
the config itself cannot require authentication.

## Return config for non-existing email addresses

Servers SHOULD return a valid config, even if the email address
sent as URL parameter does not exist. Otherwise, spammers
or attackers would be able to test the validity of email addresses.
This is true even if the config server needs the email address
to determine which of multiple configurations is correct.
In such a configuration, if the client sends a non-existing
email address, the config server SHOULD return one of the
valid configuations, so that valid and invalid email addresses
are indistiguishable.

## oAuth2 requirements

If oAuth2 is used, the oAuth2 server MUST adhere to the
mAuth specification. The oAuth2 server MUST
either accept the public client ID as given in the config file,
without secret, or MUST allow any string as client ID, without
client registration. There are also specific requirements for
expiry times and the login page, which are needed for
mail client applications to work.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Alternatives considered

## DNSSEC

Due to their top-level domain, some domains do not have DNSSEC available to them, even if they would like to deploy it.

Even where the top-level domain supports it, DNSSEC is currently deployed in only 1% of domains, with adoption rates falling instead of rising, due to the difficulties of administrating it correctly.

Therefore, DNSSEC cannot be relied on in this specification, and DNS must be considered insecure for the purposes of this specification.

## DNS SRV

DNS SRV protocols (RFC 2782, RFC 6186) are not used here, for 2 reasons:

1. DNS SRV relies on insecure DNS and the config can therefore be trivially spoofed by an attacker. See also DNSSEC above.
2. DNS SRV does not provide all the necessary configuration parameters. For example, we need all of:

* the username form ("fred@example.com", or "fred", or "fred\EXAMPLE", or even a username with no relation to the email address)
* authentication method (password, CRAM-MD5, OAuth2, SSL client certificate)
* authentication method parameters (e.g. OAuth parameters)

and other parameters. If any of these parameters are not configured right, the configuration won't work. While these parameters could theoretically be added to DNS SRV, that would mean a new specification and render the idea void that this is a protocol that already exists, is standardized and deployed. It is unlikely that all DNS SRV records would be updated with the new values. Therefore, it does not solve the problem.

This specification was created as an answer to these deficiencies and provides an alternative to DNS SRV.

## CAPABILITIES

In the wild deployments from actual ISPs show that protocol-specific commands to find available authentication methods, e.g. IMAP `CAPABILITIES` or POP3 `CAPA`, are not reliable. Many email servers advertize authentication methods that do not work.

Some IMAP servers are default configured to list all SASL authentication methods that have corresponding libraries installed on the system, independent on whether they are actually configured to work. The client receives a long list of authentication methods, and many of them do not work. Additionally, the server response may be only "authentication failed" and may not indicate whether the method failed due to lack of configuration, or because the password was wrong. Because some authentication servers lock the account after 3 failed login attempts, and it may also fail due to unrelated reasons (e.g. username form, wrong password, etc.), the client cannot blindly issue countless login attempts. Locking the account must be avoided. So, simply attempting all methods and seeing which one works is not an option for the email client.

Additionally, some email servers advertize Kerberos / GSSAPI, but when trying to use it, the method fails, and also runs into a long 2 minute timeout in some cases. End users consider that to be a broken app.

Additionally, such commands are protocol specific and have to be implemented in multiple different ways.

Finally, some non-mail protocols may not support capabilties commands that include authentication methods.


# Security Considerations

## Risk

If an attacker can provide a forged configuration, the provided mail hostname and authentication server can be controlled by the attacker, and the attacker can get access to the plain text password of the user. The attack can be implemented as man-in-the-middle, so the end user might receive mail as expected and never notice the attack.

An attacker gaining the plain text password of a real user is a very significant threat for the organization, not only because mail itself can contain sensitive information and can be used to issue orders within the organization that have wide-ranging impact, but given single-sign-on solutions, the same username and password may give access to other resources at the organization, including other computers or, in the case of admin users, even adminstrative access to the core of the entire organization.

Multi-factor authentication might not defend against such attacks, because the user may believe to be logging into his email and therefore comply with any multi-factor authentication steps required.

## DNS

Any protocol that relies on DNS without further validation, e.g. http, should be considered insecure. Even if an http URL redirects to a https URL, and the domain of the https URL cannot be validated against the email domain, that is still insecure. This also applies to the DNS MX lookup and the https calls that base on its results, as described in section "MX".

One possible mitigation is to use multiple different DNS servers
and verify that the results match, e.g. to use the native DNS
resolver of the operating system, and additionally also query
a hardcoded DoH (DNS over HTTPS) server. Nonetheless,
the result should be used with care.

Such insecure configs may only be used, if the end user confirms the config, particularly the resulting second-level domains. See section "User approval".

## Config updates

Part of the security properties of this protocol assume that the timeframe of possible attack is limited to the moment when the user manually sets up a new mail client. This moment is triggered by the end user, and a rare action - e.g. maybe once per year or even less, for typical setups -, so an attacker has limited chances to run an attack. While not a complete protection on its own, this reduces the risk significantly.

However, if the mail client does regular configuration updates using this protocol, this security property is no longer given. For regular configuration updates, the mail client MUST use only mechanisms that are secure and cannot be tampered with by an active attacker. Furthermore, the user SHOULD still approve config changes.

But even with all these protections implemented, the mail client vendor should make a security assessment for the risks of making such regular updates. The mail client vendor should consider that servers can be hacked, and most users simply approve changes proposed by the app, so these give only a limited protection.

## Server security

Given that mail clients will trust the configuration, the server delivering it needs to be secure. Even though we call it "database", static configuration files that are generated before deployment in combination with a static web server offer better security and significantly better performance than dynamic queries from a database and responses generated on-the-fly on request. If a custom server is used, it SHOULD be updated regularly and hosted on a dedicated secure server with all unnecessary services and server features turned off.

Additions and modifications to the configurations are applicable to all respective users and must be made with care. The authenticity of the configuration MUST be verified from authorative sources. Server hostnames MUST be compared with the email domain names they are serving, and if they differ, the ownership of the server hostnames MUST be validated.

For these reasons, mail clients may use the public mail config database mentioned above.

The risk is mitigated to some degree by section "User approval".


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
