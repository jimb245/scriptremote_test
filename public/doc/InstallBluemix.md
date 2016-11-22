<h2>
Installation On IBM Bluemix Platfrom
</h2>

<p>
<a href="https://console.ng.bluemix.net">Bluemix</a> is a Cloud Foundry compatible platform-as-a-service.
For small applications the server should be able to run 
in a free or low-cost tier. Scaling up in the platform is also
potentially easy. There is extensive documentation
available on the Bluemix site.

<p>
Bluemix apps automatically provide TLS using an IBM certificate 
so there is no requirement to have your own domain name
and certificate.

<ol>
<li>
<b>Create a <a href="https://console.ng.bluemix.net">Bluemix</a> account if you do not already have one.</b>
</li>
<br>

<li>
<b>Install cf and git locally</b>
<p>
The app will be pushed to Bluemix from a local git repository using the
Cloud Foundry command line utility cf.  Install cf on your machine from:

<pre>
<a href="https://github.com/cloudfoundry/cli">Cloud Foundry CLI</a>
</pre>
Install <a href="https://git-scm.com">git</a> using whatever package manager is standard on your machine.
</li>
<br>

<li>
<b>Clone ScriptRemote repository</b>
<p>
<pre>
>$ cd ~
>$ git clone ...
>$ cd scriptremote
>$ npm install --production
</pre>
</li>
<br>

<li>
<b>Set up credentials/secrets</b>
<p>

First set up an email forwarding account. This will be used to 
send security-related messages and user notifications.
Rather than using an existing account, you may want to create one 
just for this purpose with one of the free email services (gmail,
hotmail, mailgun, ...). 
Then copy the sample credentials file and substitue your values 
for the <code>MAILER</code> dummy values:

<pre>
>$ cp credentials.env .env
>$ chmod 600 .env
>$ vim .env
</pre>
<p>
Some services are stricter than others about relaying mail.
To help ensure that mail can be sent choose the same 
provider as expected for registered user emails, and set 
MAILER_FROM to the same address as MAILER_EMAIL_ID.
If users will have a variety of email providers then
consider using a service like mailgun.  Any mailing errors 
will be logged to the console.  If there is a problem it 
will probably first show up when registering the admin 
user below.
<p>
Second, generate a random string for the session middleware.
For example:

<pre>
>$ openssl rand -base64 32
</pre>

Substitute the result for the <code>SESSION_SECRET</code> value in <code>.env</code>
</li>
<br>

<li>
<b>Push the app</b>
<p>

First login to Bluemix and create the mongo service.

<pre>
>$ cf login -a api.ng.bluemix.net
>$ cf create-service mongodb 100 mean-mongo
</pre>

The manifest.yaml file contains parameters for the push, such as
the app name and memory allocation.

<pre>
>$ cf push
</pre>

Bluemix will print the url assigned to the app at the end of the
push process. 
<p>
If the app is pushed again with the same manifest it will replace 
the earlier instance, but the mongo database contents will carry 
over to the new instance.  If you want to delete the database and 
start over then delete the mongodb service and recreate it:

<pre>
>$ cf delete your-app-name
>$ cf delete-service mean-mongo
>$ cf create-service mongodb 100 mean-mongo
>$ cf push
</pre>
</li>
<br>


<li>
<b>Register the admin user</b>
<p>
Check that the server can be reached in your browser at the 
url obtained above. The protocol should be https.
<p>
Select <b>Login/Register</b> on the home screen and then
<b>Register Here</b> on the login screen. The registration screen should
display a message that the admin account is being registered. 
<p>
Continue the registration by entering at least an email and password,
and by selecting one of the options for registration of other users. The
default is to allow other users to register themselves. You can also
select a timeout for idle sessions.
<p>
A confirmation email should be sent to the registered address.
Complete the registration by clicking the link in the email or
by submitting the token value from the email into the form
displayed when attempting to login.
<p>
Return to the home page and login using the admin account.
<p>
Get script credentials by selecting <b>Settings</b> in menu bar and
then selecting <b>Generate</b> in the <b>API Credentials</b> section. The
<b>User Id</b> and <b>Token</b> values will be needed to authenticate messages
to the server from scripts running in the private network.

</li>
<br>

<li>
<b>Check that the server can be reached from the private network</b>
<p>
Copy the API credentials obtained above to a machine in the private
network, then copy the bash utility script from the local 
scriptremote repository or download it:

<pre>
>$ wget https://scriptremote/dist/srjob.sh
</pre>

Set <code>SRSERVER</code> to the url of your Bluemix server, by editing the script
or as an environment variable:

<pre>
>$ export SRSERVER=https://&lt;your-url&gt;
</pre>

Create a simple test:

<pre>
>$ cat > test.sh
#!/bin/bash
. ./srjob.sh
SR_start ${SRUSER} ${SRTOKEN} 'myproject' 'myjob'
SR_set 'msg1' 'Hello World' 'false'
SR_send 'mylocation'
SR_end
</pre>

Export the API credentials to the test script and run it:

<pre>
>$ export SRUSER=&lt;your-userid&gt;
>$ export SRTOKEN=&lt;your-token&gt;
>$ bash test.sh
</pre>

Check that the test message can be viewed in the browser by
selecting <b>Projects</b> in the menu bar.
<p>
If the script fails with a certificate verification error
it may mean that there is no CA cert store available in the
test machine OS. In that case you would need to install
a root certificate.
</li>
<br>

<li>
<b>Create non-admin user</b>
<p>
Since the admin has elevated priviledges to do things
like viewing user details and registering new users, it is best
for security purposes to minimize use of that account.
Instead register as a normal user for actual projects.
This will require a different email address from the one
used for admin registration.
</li>
<br>

<li>
<b>Server logging</b>
<p>
Bluemix logs server responses by default.
The logs can be viewed in the Bluemix dashboard or
using the <code>cf log</code> command.
It is recommended to monitor them
for signs of instrusion, such as an unknown source ip
successfully accessing an endpoint other than the
root or documentation.
</li>
</br>

<li>
<b>Optional: Enable project data limits</b>
<p>
You may want to enable limits on the amount of message data that can be
sent to the server, for example to help protect against scripting errors
that could produce very large or very many messages.
The available limits are defined in <pre>config/env/all.js</pre>
Any of them may be set as environment variables prior to starting
the server. An easy way to do this with Bluemix is to add them
to the .env file.
</li>
<br>
</ol>
