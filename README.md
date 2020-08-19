libphremoteuser
===============

This extension to [Phabricator](http://phabricator.org/) performs basic authentication 
via a web server's REMOTE_USER variable.  It should be able to work with a variety of 
major servers such as Apache and Nginx, but I have only tested it with Apache.

This version is modified to work with mod_auth_openidc.

Installation
------------

To install this library, simply clone this repository alongside your phabricator installation:

    cd /path/to/install
    git clone https://github.com/jdelker/phabricator-extensions-remoteuser.git    

Then, simply add the path to this library to your phabricator configuration:

    cd /path/to/install/phabricator
    ./bin/config set load-libraries '["/path/to/install/phabricator-extensions-remoteuser/src/"]'
    
When you next log into Phabricator as an Administrator, go to **Auth > Add Authentication Provider**.  
In the list, you should now see an entry called **Web Server**.  Enabling this provider should add a 
new button to your login screen.

In order to actually log in, your web server needs to populate the **$REMOTE_USER**, **OIDC_CLAIM_name** and
**OIDC_CLAIM_email** variable when the login button is pressed.  You can do this by forcing the login URI
that Phabricator uses to be restricted and adding the information to the rest of the server, by adding
something like the following directives to your Apache2 webserver configuration (assuming you have configured the basic mod_auth_openidc stuff already):

    # maps the prefered_username claim to the REMOTE_USER environment variable
    OIDCRemoteUserClaim preferred_username
    OIDCScope "openid email"
     
    <Location "/auth/login/RemoteUser:self/">
      Authtype openid-connect
      Require valid-user
    </Location>


Security
--------

I make no guarantees about this library being totally secure.  It's not __obviously__ insecure.  
However, please make sure to at least 
**REDIRECT THE LOGIN URI TO SSL, OTHERWISE YOU ARE SENDING PLAIN TEXT PASSWORDS.**

If you care about security consider:
  * Hosting Phabricator entirely on https/SSL
  * Restricting access to the whole Phabricator installation directory, also using SSL.
