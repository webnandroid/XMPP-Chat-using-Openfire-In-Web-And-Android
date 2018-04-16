# XMPP-Chat-using-Openfire-In-Web-And-Android
 
 For real time messaging or Push Notification we need a server that manage all of that. The Openfire is the solution for it. 
 
 To install openfire we need linux server. I am providing the links to install openifre.
 
  i) https://linode.com/docs/applications/messaging/install-openfire-on-ubuntu-12-04-for-instant-messaging/
  
  ii) https://www.digitalocean.com/community/tutorials/how-to-install-openfire-xmpp-server-on-a-debian-or-ubuntu-vps
  
  You can also install install it on AWS. You need to open port from AWS dashboard. Just add it in the rules. 
  <br><br><br>

  After installing openfire server lets start to implement Android Client
  
  
  ## Android Client Implementation
  
  We are goind to use Asmack android library to implement XMPP client in android. Lets include asmack gradle files
  
  ``` java
    compile 'org.igniterealtime.smack:smack-android:4.1.1'
    compile 'org.igniterealtime.smack:smack-tcp:4.1.1'
    compile 'org.igniterealtime.smack:smack-core:4.1.1'
    compile 'org.igniterealtime.smack:smack-im:4.1.1'
    compile 'org.igniterealtime.smack:smack-extensions:4.1.1'
    compile 'org.igniterealtime.smack:smack-android-extensions:4.1.1'
  ```
  <br><br>
  Also include Google gson library.
  ``` java
      compile 'com.google.code.gson:gson:1.7.2'
  ```
  
  Lets Define Server details in <b> Config.java </b>
  ``` java
  public class Config {
    public static final String HOST="Your Server IP address  on which your Openfire Server is installed";
    public static final int PORT=5222; // This is default PORT address of Openfire Server.
  }
  ```

  
