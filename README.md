# XMPP Chat using Openfire In Web And Android
 
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
  
  Lets Define Server details in <b><i> Config.java </i></b>
  ``` java
  public class Config {
    public static final String HOST="Your Server IP address  on which your Openfire Server is installed";
    public static final int PORT=5222; // This is default PORT address of Openfire Server.
  }
  ```
  
  <br><br>
  Now Intialize our android client connection with openfire server. Asmack android client provides <b><i>XMPPTCPConnection</i></b> class to implement this.
  
  ``` java
  
        XMPPTCPConnection connection;
        XMPPTCPConnectionConfiguration.Builder config = XMPPTCPConnectionConfiguration.builder();
        config.setSecurityMode(ConnectionConfiguration.SecurityMode.disabled);
        config.setServiceName(Config.HOST);
        config.setHost(Config.HOST);
        config.setPort(Config.PORT);
        config.setDebuggerEnabled(true);
        XMPPTCPConnection.setUseStreamManagementResumptiodDefault(true);
        XMPPTCPConnection.setUseStreamManagementDefault(true);
        connection = new XMPPTCPConnection(config.build());
        XMPPConnectionListener connectionListener = new XMPPConnectionListener();
        connection.addConnectionListener(connectionListener);
        PingManager pingManager = PingManager.getInstanceFor(connection);
        pingManager.registerPingFailedListener(this);

  ```
  
 <br> 
 Now implement XMPPConnectionListener class. It executes after connection is made with XMPP Server. It is used to check that connection is made successfully with server or not.
 
 ``` java
  public class XMPPConnectionListener implements ConnectionListener {
        @Override
        public void connected(final XMPPConnection connection) {

            Log.d("xmpp", "Connected!");
        
        }

        @Override
        public void connectionClosed() {
            
            Log.d("xmpp", "ConnectionCLosed!");
 
        }
        

        @Override
        public void connectionClosedOnError(Exception arg0) {
           
            Log.d("xmpp", "ConnectionClosedOn Error!");
          
        }

        @Override
        public void reconnectingIn(int arg0) {

            Log.d("xmpp", "Reconnectingin " + arg0);
 
        }

        @Override
        public void reconnectionFailed(Exception arg0) {
            
            Log.d("xmpp", "ReconnectionFailed!");
            
        }

        @Override
        public void reconnectionSuccessful() {
           
            Log.d("xmpp", "ReconnectionSuccessful");
            
        }

        @Override
        public void authenticated(XMPPConnection arg0, boolean arg1) {
            Log.d("xmpp", "Authenticated!");
           

            ChatManager.getInstanceFor(connection).addChatListener(mChatManagerListener);
 
            new Thread(new Runnable() {

                @Override
                public void run() {
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }

                }
            }).start();
            
                
                Log.e("xmpp","Conected");
        }
    }
 ```
  <br> After initializing connection . Now its time to connect with our XMPP Server(Openfire). Don't try to run connecting with XMPP server in UI thread. It should be implemented only in AsyncTask.
  
  ``` java
  
        AsyncTask<Void, Void, Boolean> connectionThread = new AsyncTask<Void, Void, Boolean>() {
            @Override
            protected synchronized Boolean doInBackground(Void... arg0) {
                if (connection.isConnected())
                    return false;
                  
                Log.d("Connect() Function", "Connecting....");

                try {
                    connection.connect();
                    DeliveryReceiptManager dm = DeliveryReceiptManager
                            .getInstanceFor(connection);
                    dm.setAutoReceiptMode(AutoReceiptMode.always);
                    dm.addReceiptReceivedListener(new ReceiptReceivedListener() {

                        @Override
                        public void onReceiptReceived(final String fromid,
                                                      final String toid, final String msgid,
                                                      final Stanza packet) {

                        }
                    }); 
                    
                    

                } catch (IOException e) {
                     

                    Log.e("Connection Error", "IOException: " + e.getMessage());
                } catch (SmackException e) {
                     
                    Log.e("SMACKException", "SMACKException: " + e.getMessage());
                } catch (XMPPException e) {
                      
                    Log.e("XMPPException",  "XMPPException: " + e.getMessage());

                }
                return true;
            }
        };
        connectionThread.execute();
  ```
  <br><br>
  After connecting with server. Lets register our user from android app. If user is already register its just overwrite current details   with new one.
  
  ``` java
        AccountManager account = AccountManager.getInstance(connection);
        Map<String, String> details = new HashMap<String, String>();
        details.put("name", "Your Name");

        try {
         if (accountManager.supportsAccountCreation()) {
             accountManager.sensitiveOperationOverInsecureConnection(true);
             account.createAccount("Your Username", "Your Password",details);
            }
        } catch (SmackException.NoResponseException e) {
            e.printStackTrace();
        } catch (XMPPException.XMPPErrorException e) {
            e.printStackTrace();
        } catch (NotConnectedException e) {
            e.printStackTrace();
        }

  ```
  <br><br> Now lets try to login our newly created user with XMPP server
  
  ``` java
  
   try {
            connection.login("Your Username", "Your Password");
            Log.i("LOGIN", "Yey! We're connected to the Xmpp server!");
           
        } catch (XMPPException | SmackException | IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
        }
  ```
  <br><br> Lets focus on our main part i.e <b><i>Chat</i></b>. Lets create <b><i>MessageListener</i></b> which is called when other user send message to me. It implements the ChatMessageListener class.
  ``` java
   class MessageListener implements ChatMessageListener {

         

        @Override
        public void processMessage(final org.jivesoftware.smack.chat.Chat chat,
                                   final Message message) {
            Log.i("MyXMPP_MESSAGE_LISTENER", "Xmpp message received: '"  + message);

            if (message.getType() == Message.Type.chat    && message.getBody() != null) {
               
               Log.e("Message from "+message.getFrom(),message.getBody());
            }
        }
      
    }

  ```
  <br><br>
  
  Now implement to send message to other user
  <br>
  ``` java
   Chat mychat;
   String to_username=""; // This is the username of your friend
   MessageListener messageListener = new MessageListener(context);
   mychat = ChatManager.getInstanceFor(connection).createChat( to_username +"@"+Config.HOST,messageListener);
   
         Message message = new Message();
         message.setBody("Your Message");
         message.setType(Message.Type.chat);

        try {
            if (connection.isAuthenticated()) {

                mychat.sendMessage(message);

            } else {

                // login first
            }
        } catch (NotConnectedException e) {
            Log.e("xmpp.SendMessage()", "msg Not sent!-Not Connected!");

        } catch (Exception e) {
            Log.e("xmpp.SendMessage","msg Not sent!" + e.getMessage());
        }

  ```
  
  
  XMPP provides better way to implementing the real time messaging for various application like chat, push notification. If you have    your own server then its free for you. No need to pay for third pary libraries. You can even customize XMPP server (Openfire) for your own requirements.
