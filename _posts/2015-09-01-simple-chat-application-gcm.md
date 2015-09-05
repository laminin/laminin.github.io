---
layout:     post
title:      Creating simple chat application using GCM and rails.
date:       2015-09-01 10:14:55
author:     Franklin
summary:    An article about how to create a simple chat application in Android using GCM and Rails.
categories: Gcm android rails
tags:
 - Chat
 - GCM
 - Rails
 - Android
 - Push notifications
---

In this article we are going to create a simple chat application using [GCM][0]. This application contains two parts a) server side b) client side. For server side we will be using [Rails][1] and for client side we will be using [Android.][2] Since We are going to build the client, server applications by referring [GCM documentation][3] i highly recommend to read it at least once.

## Server side

__Create a [new rails project][1] using the following command__
{% highlight sh %}
rails new gcmmer
cd gcmmer
bundle install
{% endhighlight %}

__User model and Message model__

We will have two models. One for storing the Users another for storing Messages. User model has two fields. 1. name 2. email. Message model has three fields 1. sender 2. message 3. receiver

* [Associations.][4]

User - has many Messages.
Message - belongs to User.

* Lets generate model and create db.

* To generate User model
{% highlight sh %}
rails g model User gcm_id:string name:string email:string
{% endhighlight %}

* To generate Message model
{% highlight sh %}
rails g model Message sender:string receiver:string content:string user:references
{% endhighlight %}

* user:reference makes the belongs to relationship with User.

* Create database and table using
{% highlight sh %}
rake db:migrate
{% endhighlight %}

__User controller and Gcm controller__

We need two Controllers one for creating and sending the user list client.

* let generate the controllers using
{% highlight sh %}
rails g controller gcm
rails g controller users
{% endhighlight %}

__Routes__

Now lets define the Routes. Open config/routes.rb file and add the following
{% highlight ruby %}
resources :users, only: [:index, :create]
match '/send_notification' => 'gcm#send_notification', via: :post
match '/get_messages' => 'gcm#get_messages', via: :post
{% endhighlight %}

To learn more about [rails routing and resources][5] refer [this.][5]

now if you run
{% highlight sh %}
rake routes
{% endhighlight %}

you should be able to see all the routes and its corresponding controllers and the HTTP methods.

__Adding a user__

__Setting up users controller.__

open app/controller/users_controller and add the following code.
{% highlight ruby%}
class UsersController < ApplicationController
  skip_before_action :verify_authenticity_token
  def index
    result = {}
    result[:status] = '0'
    result[:message] = 'User list'
    result[:users] = User.pluck(:name, :email)
    render json: result
  end
  def create
    result = {}
    if User.find_or_create_by! user_params
      result[:status] = '0'
      result[:message] = 'User successfully added'
    else
      result[:status] = '1'
      result[:message] = 'Something went wrong could not add user.'
    end
    render json: result
  end
  private
  def user_params
    params.permit(:gcm_id, :email, :name)
  end
end
{% endhighlight %}

__Setting up User model__

Then open app/models/users.rb and add the following associations and validations.
{% highlight ruby %}
class User < ActiveRecord::Base
  has_many :messages
  validates_uniqueness_of :gcm_id
  validates_presence_of :email
end
{% endhighlight %}


__Setting up GCM pushnotification controller.__

Open app/controllers/gcm_controller.rb file and add the following

{% highlight ruby %}
require 'open-uri'
require 'nokogiri'
require 'net/http'
class GcmController < ApplicationController
  skip_before_action :verify_authenticity_token
  def send_notification
    # lets find the sender, receiver.
    if (sender = User.find_by_email params[:sender]) && (receiver = User.find_by_email params[:receiver] )
      if sender.messages.create notification_params
        uri = URI('https://android.googleapis.com/gcm/send')
        https = Net::HTTP.new(uri.host, uri.port)
        https.use_ssl = true
        post_data = ActiveSupport::JSON.encode({ to: receiver.gcm_id, data: {message: params[:content]}})
        headers = {
            'Authorization' => 'key=yourapplicationkey',
            'Content-Type' => 'application/json'
        }
        resp, data = https.post(uri.path, post_data, headers)
      end
    end
    render json: collect_messages
  end
  def get_messages
    render json:  collect_messages
  end
  private
  def notification_params
    params.permit(:sender, :content, :receiver)
  end
  def get_messages_param
    params.permit(:sender, :receiver)
  end
  def collect_messages
    result = {}
    messages = Message.where("sender = ? AND receiver = ? OR sender = ? AND receiver = ?", params[:sender], params[:receiver], params[:receiver], params[:sender]).pluck(:content, :sender).last(20)
    if messages
      result[:status] = 0
      result[:messages] = messages
    else
      result[:status] = 1
      result[:messages] = 'Could not found any messages.'
    end
    result
  end
end
{% endhighlight %}

for more information about gcm notification url, application key, message body structure please refer [GCM server side documentation.][6]

__Setting up Message model__

Open app/models/message.rb file and add the following
{% highlight ruby %}
class Message < ActiveRecord::Base
  belongs_to :user
  validates_presence_of :sender, :receiver, :content
end
{% endhighlight %}

Thats all required from servers side, you can run the server using
{% highlight sh %}
rails s
{% endhighlight %}


## Client side

### Android

Login to [google developer console][7] and create an app. Then create an  api key for the app. This is the key you should use with your servers's gcm-controller.rb file.

Now open android studio and crate a new project.

__As mentioned in the [gcm client documentation][8] add the following in your app's manifest file.__

{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.laminin.gcmer" >
    <uses-sdk android:minSdkVersion="8" android:targetSdkVersion="17" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <permission
        android:name="com.laminin.gcmer.permission.C2D_MESSAGE"
        android:protectionLevel="signature" />
    <uses-permission android:name="com.laminin.gcmer.permission.C2D_MESSAGE" />
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <receiver
            android:name="com.google.android.gms.gcm.GcmReceiver"
            android:exported="true"
            android:permission="com.google.android.c2dm.permission.SEND" >
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <action android:name="com.google.android.c2dm.intent.REGISTRATION" />

                <category android:name="com.laminin.gcmer" />
            </intent-filter>
        </receiver>
        <activity
            android:name=".GcmActivity"
            android:label="@string/title_activity_gcm" >
        </activity>
        <service
            android:name=".GcmListener"
            android:exported="false" >
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            </intent-filter>
        </service>
        <service
            android:name=".InstanceIDListener"
            android:exported="false" >
            <intent-filter>
                <action android:name="com.google.android.gms.iid.InstanceID" />
            </intent-filter>
        </service>
        <service
            android:name=".RegistrationIntentService"
            android:exported="false" >
        </service>
        <activity
            android:name=".ChatActivity"
            android:label="@string/title_activity_chat" >
        </activity>
    </application>
</manifest>
{% endhighlight %}


__Add the following string resources in your string.xml file__
{% highlight xml %}
<resources>
    <string name="app_name">GCMer</string>
    <string name="hello_world">Hello world!</string>
    <string name="action_settings">Settings</string>
    <string name="title_activity_gcm">GcmActivity</string>
    <string name="gcm_send_message">Token retrieved and sent to server!</string>
    <string name="registering_message">Generating InstanceID token...</string>
    <string name="token_error_message">An error occurred while either fetching the InstanceID token,
        sending the fetched token to the server or subscribing to the PubSub topic. Please try
        running the sample again.</string>
    <string name="try_later">Could not generate google play service token, please try later</string>
    <string name="title_activity_chat">ChatActivity</string>
</resources>
{% endhighlight %}

__Add the following color in our colors.xml file__
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="blue">#46adff</color>
</resources>
{% endhighlight %}

__Create Constants.java file and add the following constants__

{% highlight java %}
public class Constants {
    public static final String NAME = "name";
    public static final String EMAIL = "email";
    public static final String SENDER = "sender";
    public static final String RECEIVER = "receiver";

    public static final String SENT_TOKEN_TO_SERVER = "sentTokenToServer";
    public static final String REGISTRATION_COMPLETE = "registrationComplete";
    public static final String NOTIFICATION_RECEIVED = "notificationReceived";

//    public static final String FETCH_ALL_USERS_URL = "http://10.0.0.6:3000/users";
//    public static final String GET_MESSAGES_URL = "http://10.0.0.6:3000/get_messages";
//    public static final String SEND_NOTIFICATIONS_URL = "http://10.0.0.6:3000/send_notification";
}
{% endhighlight %}

Change the ip address to your ip address. you can find your local ip address using [ifconfig command.][9] :3000 is rails default port, if your server app runs on different port change accordingly.

__Lets create our login / register page.__

MainActivity is our login activity. When the user login for the first time we collect the user's email and name and send it to our server and store it. if we successfully store the user details then we set a flag in our sharedPreferences, so the next time user does not have to login rather he can directly move to GcmActivity. So during login we check the sharedPreferences flag, if its set to true then we launch GcmActivity else collect email and name from user and launch GcmActivity.

__Open activity_main.xml and add the following code__
{% highlight xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:layout_margin="10dp"
    tools:context=".MainActivity">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:textColor="@color/blue"
            android:text="Name"/>
        <EditText
            android:id="@+id/edit_text_name"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"/>
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:textColor="@color/blue"
            android:text="Email"/>
        <EditText
            android:id="@+id/edit_text_email"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"/>
    </LinearLayout>
    <Button
        android:id="@+id/button_register"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Register"
        android:background="@color/blue"/>
</LinearLayout>
{% endhighlight %}


__Open MainActivity.java and add the following__
{% highlight java %}
public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    EditText nameEditText;
    EditText emailEditText;
    Button registerButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        nameEditText = (EditText) findViewById(R.id.edit_text_name);
        emailEditText = (EditText) findViewById(R.id.edit_text_email);
        registerButton = (Button) findViewById(R.id.button_register);
        registerButton.setOnClickListener(this);

        SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);

        // already registered!
        if(sharedPreferences.getBoolean(Constants.SENT_TOKEN_TO_SERVER, false)){
            Intent intent = new Intent(this, GcmActivity.class);
            startActivity(intent);
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }

    @Override
    public void onClick(View view) {
        switch (view.getId()){
            case R.id.button_register:
                String name = nameEditText.getText().toString();
                String email = emailEditText.getText().toString();

                if (name.trim().length() > 1 && email.trim().length() > 1){
                    Intent intent = new Intent(this, GcmActivity.class);
                    intent.putExtra(Constants.NAME, name);
                    intent.putExtra(Constants.EMAIL, email);
                    startActivity(intent);
                }else{
                    Toast.makeText(this, "Error: while validating credentials", Toast.LENGTH_LONG).show();
                }

                break;
            default:
                break;
        }
    }
}
{% endhighlight %}

__Create new Activity called GcmActivity.__

If the user is not registered then we register the user using RegistrationIntentService then we fetch all the registered user's from our server.

__Open activity_gcm.xml and add the flollwoing__
{% highlight xml %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.laminin.gcmer.GcmActivity">
    <LinearLayout
        android:id="@+id/progress_bar_linear_layout"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:layout_centerInParent="true">
        <TextView android:text="@string/registering_message"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/informationTextView"
            android:textAppearance="?android:attr/textAppearanceMedium" />
        <ProgressBar
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:id="@+id/registrationProgressBar" />
    </LinearLayout>
    <ListView
        android:id="@+id/user_list"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:visibility="gone"/>
</RelativeLayout>
{% endhighlight %}

__Open GcmActivity.java file and add the following code.__

{% highlight java %}
public class GcmActivity extends AppCompatActivity {

    private Bundle bundle;
    private String name;
    private String email;
    private String sender;

    private static final int PLAY_SERVICES_RESOLUTION_REQUEST = 9000;
    private static final String TAG = "GCMActivity";

    private BroadcastReceiver mRegistrationBroadcastReceiver;
    private ProgressBar mRegistrationProgressBar;
    private TextView mInformationTextView;
    private ListView userListView;
    private AsyncTask<Void, Void, JSONObject> getRegisteredUsersTask;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_gcm);

        bundle = getIntent().getExtras();
        if (null != bundle){
            name = bundle.getString(Constants.NAME);
            email = bundle.getString(Constants.EMAIL);
        }
        mRegistrationProgressBar = (ProgressBar) findViewById(R.id.registrationProgressBar);
        mRegistrationBroadcastReceiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                mRegistrationProgressBar.setVisibility(ProgressBar.GONE);
                SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(context);
                boolean sentToken = sharedPreferences
                        .getBoolean(Constants.SENT_TOKEN_TO_SERVER, false);
                if (sentToken) {
                    mInformationTextView.setText(getString(R.string.gcm_send_message));
                    fetchRegisteredUsers();
                } else {
                    mInformationTextView.setText(getString(R.string.token_error_message));
                }
            }
        };

        mInformationTextView = (TextView) findViewById(R.id.informationTextView);
        userListView = (ListView) findViewById(R.id.user_list);

        /*if (checkPlayServices()) {
            // Start IntentService to register this application with GCM.
            // name and phone number we need to register with our server
            Intent intent = new Intent(this, RegistrationIntentService.class);
            intent.putExtra(Constants.NAME, name);
            intent.putExtra(Constants.EMAIL, email);
            startService(intent);
        }else{
            mInformationTextView.setText(R.string.try_later);
            mRegistrationProgressBar.setVisibility(View.GONE);

            // just for testing.
            Intent intent = new Intent(this, RegistrationIntentService.class);
            intent.putExtra(Constants.NAME, name);
            intent.putExtra(Constants.EMAIL, email);
            startService(intent);
        }*/

        SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);

        if(sharedPreferences.getBoolean(Constants.SENT_TOKEN_TO_SERVER, false)){
            // fetch all the registered users
            sender = sharedPreferences.getString(Constants.SENDER, null);
            fetchRegisteredUsers();
        }else{ // we have not registered, lets register.
            sender = email;
            Intent intent = new Intent(this, RegistrationIntentService.class);
            intent.putExtra(Constants.NAME, name);
            intent.putExtra(Constants.EMAIL, email);
            startService(intent);
        }

        userListView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
                try{
                    Intent intent = new Intent(GcmActivity.this, ChatActivity.class);
                    intent.putExtra(Constants.SENDER, sender);
                    intent.putExtra(Constants.RECEIVER, ((JSONArray)adapterView.getAdapter().getItem(i)).get(1).toString());
                    startActivity(intent);
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }
        });
    }

    private void fetchRegisteredUsers() {
        getRegisteredUsersTask = new AsyncTask<Void, Void, JSONObject>() {
            @Override
            protected JSONObject doInBackground(Void... voids) {
                try {
                    URL url = new URL(Constants.FETCH_ALL_USERS_URL);
                    HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                    conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
                    InputStream responseInputStream = conn.getInputStream();
                    StringBuffer responseStringBuffer = new StringBuffer();
                    byte[] byteContainer = new byte[1024];
                    for (int i; (i = responseInputStream.read(byteContainer)) != -1; ) {
                        responseStringBuffer.append(new String(byteContainer, 0, i));
                    }
                    JSONObject response = new JSONObject(responseStringBuffer.toString());
                    return response;
                } catch (MalformedURLException e) {
                    e.printStackTrace();
                } catch (ProtocolException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                return null;
            }

            @Override
            protected void onPostExecute(JSONObject response) {
                super.onPostExecute(response);

                // update the UI.

                if(null != response) {
                    try {
                        userListView.setAdapter(new UsersAdapter(GcmActivity.this, response.getJSONArray("users")));
                        userListView.setVisibility(View.VISIBLE);
                        findViewById(R.id.progress_bar_linear_layout).setVisibility(View.GONE);
                    } catch (JSONException e) {
                        e.printStackTrace();
                    }
                }else{
                    ((TextView)findViewById(R.id.informationTextView)).setText("Could not connect with server, please try later");
                    mRegistrationProgressBar.setVisibility(View.GONE);
                }

                getRegisteredUsersTask.cancel(true);
                getRegisteredUsersTask = null;
            }
        }.execute();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_gcm, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();
        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }
        return super.onOptionsItemSelected(item);
    }

    private boolean checkPlayServices() {
        int resultCode = GooglePlayServicesUtil.isGooglePlayServicesAvailable(this);
        if (resultCode != ConnectionResult.SUCCESS) {
            if (GooglePlayServicesUtil.isUserRecoverableError(resultCode)) {
                GooglePlayServicesUtil.getErrorDialog(resultCode, this,
                        PLAY_SERVICES_RESOLUTION_REQUEST).show();
            } else {
                Log.i(TAG, "This device is not supported.");
                finish();
            }
            return false;
        }
        return true;
    }

    @Override
    protected void onPause() {
        LocalBroadcastManager.getInstance(this).unregisterReceiver(mRegistrationBroadcastReceiver);
        super.onPause();
    }

    @Override
    protected void onResume() {
        super.onResume();
        LocalBroadcastManager.getInstance(this).registerReceiver(mRegistrationBroadcastReceiver, new IntentFilter(Constants.REGISTRATION_COMPLETE));
    }

    class UsersAdapter extends BaseAdapter{
        JSONArray mUserList;
        LayoutInflater mInflater;
        UsersAdapter(Context context, JSONArray userList){
            mInflater = LayoutInflater.from(context);
            mUserList = userList;
        }

        @Override
        public int getCount() {
            if (null != mUserList) return mUserList.length();
            else return 0;
        }

        @Override
        public Object getItem(int i) {
            try {
                return mUserList.get(i);
            } catch (JSONException e) {
                e.printStackTrace();
            }
            return null;
        }
        @Override
        public long getItemId(int i) {
            return 0;
        }
        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            View view;
            ViewHolder holder;
            if(convertView == null) {
                view = mInflater.inflate(R.layout.row_user_list_layout, parent, false);
                holder = new ViewHolder();
                holder.name = (TextView)view.findViewById(R.id.name);
                holder.email = (TextView)view.findViewById(R.id.email);
                view.setTag(holder);
            } else {
                view = convertView;
                holder = (ViewHolder)view.getTag();
            }

            try {
                JSONArray user = mUserList.getJSONArray(position);
                holder.name.setText(user.get(0).toString());
                holder.email.setText(user.get(1).toString());
            } catch (JSONException e) {
                e.printStackTrace();
            }
            return view;
        }

        private class ViewHolder {
            public TextView name, email;

        }
    }
}

{% endhighlight %}

__Lets create a class named RegistrationIntentService.class and add the following code.__

The RegistrationIntentService will generate GCMRegistrationToken for the device and it will register it in our server. This token will be used as the gcm_id of the receiver in our server. (you can refer gcm_controller.rb's send_notification function). If we are able to register is successfully then we update the sharedPreferences flag, and broadcast a local message that the registration is successfull. The GcmActivity has a broad cast receiver which receives the message and fetch all the available users and display it on the list view.

{% highlight java %}
public class RegistrationIntentService extends IntentService {

    private static final String TAG = "RegIntentService";
    private static final String[] TOPICS = {"global"};

    private String name;
    private String email;

    AsyncTask<Void, Void, Boolean> registerUserDetails;
    SharedPreferences sharedPreferences;
    public RegistrationIntentService() {
        super(TAG);
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);

        try {
            // [START register_for_gcm]
            // Initially this call goes out to the network to retrieve the token, subsequent calls
            // are local.
            // [START get_token]

            name = intent.getStringExtra(Constants.NAME);
            email = intent.getStringExtra(Constants.EMAIL);

            InstanceID instanceID = InstanceID.getInstance(this);
            String token = instanceID.getToken(getString(R.string.gcm_defaultSenderId), GoogleCloudMessaging.INSTANCE_ID_SCOPE, null);
            // [END get_token]
            Log.d(TAG, "GCM Registration Token: " + token);

            // TODO: Implement this method to send any registration to your app's servers.
            sendRegistrationToServer(token);

            // Subscribe to topic channels
            subscribeTopics(token);


            // [END register_for_gcm]
        } catch (Exception e) {
            Log.d(TAG, "Failed to complete token refresh", e);
            // If an exception happens while fetching the new token or updating our registration data
            // on a third-party server, this ensures that we'll attempt the update at a later time.
            sharedPreferences.edit().putBoolean(Constants.SENT_TOKEN_TO_SERVER, false).apply();
        }

    }

    private void sendRegistrationToServer(final String token) {
        // should be a async task to server to store the details.

        registerUserDetails = new AsyncTask<Void, Void, Boolean>() {
            @Override
            protected Boolean doInBackground(Void... voids) {
                // 10.0.0.4

                try {
                    URL url = new URL(Constants.FETCH_ALL_USERS_URL);
                    String postData = "gcm_id="+ token + "&name=" + name + "&email=" + email;
                    byte[] postParamsByte =postData.getBytes("UTF-8");
                    HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                    conn.setRequestMethod("POST");
                    conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
                    conn.setRequestProperty("Content-Length", String.valueOf(postParamsByte.length));
                    conn.setDoOutput(true);
                    conn.getOutputStream().write(postParamsByte);

                    InputStream responseInputStream = conn.getInputStream();
                    StringBuffer responseStringBuffer = new StringBuffer();
                    byte[] byteContainer = new byte[1024];
                    for (int i; (i = responseInputStream.read(byteContainer)) != -1; ) {
                        responseStringBuffer.append(new String(byteContainer, 0, i));
                    }

                    JSONObject response = new JSONObject(responseStringBuffer.toString());

                    if(response.getString("status").contentEquals("0")) return true;
                    else return false;

                } catch (ProtocolException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                return false;
            }

            @Override
            protected void onPostExecute(Boolean status) {
                super.onPostExecute(status);

                updatePreferencesAndBroadcast(status);

                registerUserDetails.cancel(true);
                registerUserDetails = null;
            }
        }.execute();

    }

    private void updatePreferencesAndBroadcast(boolean status) {
        // You should store a boolean that indicates whether the generated token has been
        // sent to your server. If the boolean is false, send the token to your server,
        // otherwise your server should have already received the token.
        SharedPreferences.Editor editor = sharedPreferences.edit();
        if(status){
            editor.putBoolean(Constants.SENT_TOKEN_TO_SERVER, true);
            editor.putString(Constants.SENDER, email);
            editor.apply();
        }
        else editor.putBoolean(Constants.SENT_TOKEN_TO_SERVER, false).apply();

        // Notify UI that registration has completed, so the progress indicator can be hidden.
        Intent registrationComplete = new Intent(Constants.REGISTRATION_COMPLETE);
        LocalBroadcastManager.getInstance(this).sendBroadcast(registrationComplete);
    }

    private void subscribeTopics(String token) throws IOException {
        for (String topic : TOPICS) {
            GcmPubSub pubSub = GcmPubSub.getInstance(this);
            pubSub.subscribe(token, "/topics/" + topic, null);
        }
    }
}
{% endhighlight %}

More likely you get an error at
{% highlight java %}
String token = instanceID.getToken(getString(R.string.gcm_defaultSenderId), GoogleCloudMessaging.INSTANCE_ID_SCOPE, null);
{% endhighlight %}

to avoid that open root level build.gradle file and add the following line
{% highlight sh %}
  classpath 'com.google.gms:google-services:1.3.0-beta1'
{% endhighlight %}

then open app level build.sh file and add the following dependency and the plugin.
{% highlight sh %}
apply plugin: 'com.google.gms.google-services'
.....
dependencies {
    .....
    compile 'com.android.support:appcompat-v7:23.0.0'
    compile 'com.google.android.gms:play-services:7.8.0'
}

{% endhighlight %}


__Now create an class called ChatActivity__

This helps to compose and send message to other users. Chat activity fetches last 20 conversations between the sender and the receiver. Sender is the one who logged in and the receiver is the user who you selected from the previous activity. Chat activity also send the message and send it to our server. Our server send the data to GCM server.

__Open activity_chat.xml file and add the following code.__

{% highlight xml %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.laminin.gcmer.ChatActivity">
    <LinearLayout
        android:id="@+id/send_linear_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_alignParentBottom="true">
        <EditText
            android:id="@+id/message_edit_text"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"/>
        <Button
            android:id="@+id/send_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Send"/>
    </LinearLayout>
    <ListView
        android:id="@+id/message_list_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:transcriptMode="alwaysScroll"
        android:stackFromBottom="true"
        android:layout_above="@+id/send_linear_layout">
    </ListView>
</RelativeLayout>
{% endhighlight %}

__Open ChatActivity.java file and add the following.__
{% highlight java %}
public class ChatActivity extends AppCompatActivity implements View.OnClickListener{

    private String sender;
    private String receiver;

    private ListView messageListView;
    private Button sendButton;
    private EditText messageEditText;

    RelativeLayout.LayoutParams paramsLeftAlign;
    RelativeLayout.LayoutParams paramsRightAlign;

    private AsyncTask<Void, Void, JSONArray> fetchMessagesTask;
    private AsyncTask<Void, Void, JSONArray> sendMessageTask;

    private BroadcastReceiver mNotificationBroadcastReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_chat);

//        lets find the sender, receiver
        Intent intent = getIntent();
        sender = intent.getStringExtra(Constants.SENDER);
        receiver = intent.getStringExtra(Constants.RECEIVER);

        messageListView = (ListView) findViewById(R.id.message_list_view);
        messageEditText = (EditText) findViewById(R.id.message_edit_text);
        (sendButton = (Button) findViewById(R.id.send_button)).setOnClickListener(this);

        paramsLeftAlign = new RelativeLayout.LayoutParams(RelativeLayout.LayoutParams.WRAP_CONTENT, RelativeLayout.LayoutParams.WRAP_CONTENT);
        paramsLeftAlign.addRule(RelativeLayout.ALIGN_PARENT_LEFT, RelativeLayout.TRUE);

        paramsRightAlign = new RelativeLayout.LayoutParams(RelativeLayout.LayoutParams.WRAP_CONTENT, RelativeLayout.LayoutParams.WRAP_CONTENT);
        paramsRightAlign.addRule(RelativeLayout.ALIGN_PARENT_RIGHT, RelativeLayout.TRUE);

        fetchMessages();

        mNotificationBroadcastReceiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                fetchMessages();
            }
        };

    }

    private void fetchMessages() {

        fetchMessagesTask = new AsyncTask<Void, Void, JSONArray>() {

            @Override
            protected JSONArray doInBackground(Void... voids) {
                try {
                    URL url = new URL(Constants.GET_MESSAGES_URL);
                    String postData = "sender=" + sender + "&receiver=" + receiver;
                    byte[] postParamsByte = postData.getBytes("UTF-8");

                    HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                    conn.setRequestMethod("POST");
                    conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
                    conn.setRequestProperty("Content-Length", String.valueOf(postParamsByte.length));
                    conn.setDoOutput(true);
                    conn.getOutputStream().write(postParamsByte);

                    InputStream responseInputStream = conn.getInputStream();
                    StringBuffer responseStringBuffer = new StringBuffer();
                    byte[] byteContainer = new byte[1024];
                    for (int i; (i = responseInputStream.read(byteContainer)) != -1; ) {
                        responseStringBuffer.append(new String(byteContainer, 0, i));
                    }

                    JSONObject response = new JSONObject(responseStringBuffer.toString());
                    if(response.getString("status").contentEquals("0")){
                        return response.getJSONArray("messages");
                    }

                } catch (MalformedURLException e) {
                    e.printStackTrace();
                } catch (ProtocolException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                return null;
            }

            @Override
            protected void onPostExecute(JSONArray jsonArray) {
                super.onPostExecute(jsonArray);

                // update ui;

                updateUi(jsonArray);

                fetchMessagesTask.cancel(true);
                fetchMessagesTask = null;

            }
        }.execute();
    }

    private void updateUi(JSONArray jsonArray) {
        messageListView.setAdapter(new MessagesAdapter(ChatActivity.this, jsonArray));
        messageEditText.getText().clear();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_chat, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }

    @Override
    public void onClick(View view) {
        switch (view.getId()){
            case R.id.send_button:
                String message = messageEditText.getText().toString();
                if(message.trim().length() > 1) sendMessage(message);
                break;
            default:
                break;
        }
    }

    private void sendMessage(final String message) {
        sendMessageTask = new AsyncTask<Void, Void, JSONArray>() {
            @Override
            protected JSONArray doInBackground(Void... voids) {

                try {
                    URL url = new URL(Constants.SEND_NOTIFICATIONS_URL);
                    String postData = "sender=" + sender + "&receiver=" + receiver + "&content=" + message;
                    byte[] postParamsByte = postData.getBytes("UTF-8");

                    HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                    conn.setRequestMethod("POST");
                    conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
                    conn.setRequestProperty("Content-Length", String.valueOf(postParamsByte.length));
                    conn.setDoOutput(true);
                    conn.getOutputStream().write(postParamsByte);

                    InputStream responseInputStream = conn.getInputStream();
                    StringBuffer responseStringBuffer = new StringBuffer();
                    byte[] byteContainer = new byte[1024];
                    for (int i; (i = responseInputStream.read(byteContainer)) != -1; ) {
                        responseStringBuffer.append(new String(byteContainer, 0, i));
                    }

                    JSONObject response = new JSONObject(responseStringBuffer.toString());
                    if(response.getString("status").contentEquals("0")){
                        return response.getJSONArray("messages");
                    }

                } catch (MalformedURLException e) {
                    e.printStackTrace();
                } catch (ProtocolException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                return null;
            }

            @Override
            protected void onPostExecute(JSONArray jsonArray) {
                super.onPostExecute(jsonArray);

                // update ui.
                updateUi(jsonArray);

                sendMessageTask.cancel(true);
                sendMessageTask = null;
            }
        }.execute();

    }

    class MessagesAdapter extends BaseAdapter{

        JSONArray mMessagingList;
        LayoutInflater mLayoutInflater;
        MessagesAdapter(Context context, JSONArray messageList){
            mMessagingList = messageList;
            mLayoutInflater = LayoutInflater.from(context);
        }

        @Override
        public int getCount() {
            if(mMessagingList != null) return mMessagingList.length();
            else return 0;
        }

        @Override
        public Object getItem(int i) {
            try {
                return mMessagingList.get(i);
            } catch (JSONException e) {
                e.printStackTrace();
            }
            return null;
        }

        @Override
        public long getItemId(int i) {
            return 0;
        }

        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            View view;
            ViewHolder holder;
            if(convertView == null) {
                view = mLayoutInflater.inflate(R.layout.row_message_list_layout, parent, false);
                holder = new ViewHolder();
                holder.messageTextView = (TextView)view.findViewById(R.id.message_text_view);
                view.setTag(holder);
            } else {
                view = convertView;
                holder = (ViewHolder)view.getTag();
            }

            try {
                JSONArray message = mMessagingList.getJSONArray(position);
                holder.messageTextView.setText(message.get(0).toString());
                if(sender.contentEquals(message.get(1).toString())){ // send by us. align it left side
                    holder.messageTextView.setLayoutParams(paramsLeftAlign);
                }else{ // right align
                    holder.messageTextView.setLayoutParams(paramsRightAlign);
                }
            } catch (JSONException e) {
                e.printStackTrace();
            }
            return view;
        }

        private class ViewHolder {
            public TextView messageTextView;

        }
    }

    @Override
    protected void onPause() {
        LocalBroadcastManager.getInstance(this).unregisterReceiver(mNotificationBroadcastReceiver);
        super.onPause();
    }

    @Override
    protected void onResume() {
        super.onResume();
        LocalBroadcastManager.getInstance(this).registerReceiver(mNotificationBroadcastReceiver, new IntentFilter(Constants.NOTIFICATION_RECEIVED));
    }
}
{% endhighlight %}

__GCM listener__
As soon as our Server send the message to GCM server it send the message as a push notification to the receiver's device. To receive the message from GCM we need to create another service. This service will have a notification builder which notify the user with the notification message.

__Create a class called GcmListener.java and add the following.__
{% highlight java %}
public class GcmListener extends GcmListenerService {
    private static final String TAG = "GcmListener";

    /**
     * Called when message is received.
     *
     * @param from SenderID of the sender.
     * @param data Data bundle containing message data as key/value pairs.
     *             For Set of keys use data.keySet().
     */
    // [START receive_message]
    @Override
    public void onMessageReceived(String from, Bundle data) {
        String message = data.getString("message");
        Log.d(TAG, "From: " + from);
        Log.d(TAG, "Message: " + message);

        if (from.startsWith("/topics/")) {
            // message received from some topic.
        } else {
            // normal downstream message.
        }

        // [START_EXCLUDE]
        /**
         * Production applications would usually process the message here.
         * Eg: - Syncing with server.
         *     - Store message in local database.
         *     - Update UI.
         */

        /**
         * In some cases it may be useful to show a notification indicating to the user
         * that a message was received.
         */
        sendNotification(message);
        // [END_EXCLUDE]
    }
    // [END receive_message]

    /**
     * Create and show a simple notification containing the received GCM message.
     *
     * @param message GCM message received.
     */
    private void sendNotification(String message) {
//        Intent intent = new Intent(this, ChatActivity.class);
//        intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
//        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0 /* Request code */, intent,
//                PendingIntent.FLAG_ONE_SHOT);



        Uri defaultSoundUri= RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
        NotificationCompat.Builder notificationBuilder = new NotificationCompat.Builder(this)
                .setSmallIcon(R.drawable.ic_stat_ic_notification)
                .setContentTitle("GCM Message")
                .setContentText(message)
                .setAutoCancel(true)
                .setSound(defaultSoundUri);
//                .setContentIntent(pendingIntent);

        NotificationManager notificationManager =
                (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);

        notificationManager.notify(0 /* ID of notification */, notificationBuilder.build());

        // Notify Chats activity that Notification  has received, so we update the conversations.
        Intent notificationReceived = new Intent(Constants.NOTIFICATION_RECEIVED);
        LocalBroadcastManager.getInstance(this).sendBroadcast(notificationReceived);
    }
}
{% endhighlight %}

Again we broadcast a message to chat activity that a message has been arrived so the Chat activity can fetch the messages again.

__Instance id listener__

Create one more class called InstanceIDListener and add the following code. This will help us to refresh the token when there is a update in InstanceID.
{% highlight java%}
public class InstanceIDListener extends InstanceIDListenerService {
    @Override
    public void onTokenRefresh() {
        // Fetch updated Instance ID token and notify our app's server of any changes (if applicable).
        Intent intent = new Intent(this, RegistrationIntentService.class);
        startService(intent);
    }
}
{% endhighlight %}

if you install the application on two different devices and if your server is running then you should be able to send message from one device to another.


__Get the source code__

You can clone server project form [github](https://github.com/Franklin2412/gcm-chat-server)

or clone server project using following command.

{% highlight sh %}
  git clone git@github.com:Franklin2412/gcm-chat-server.git
{% endhighlight %}


You can clone client android project from [github](https://github.com/Franklin2412/gcm-chat-client-android)

or clone client android project using following command.

{% highlight sh %}
  git clone git@github.com:Franklin2412/gcm-chat-client-android.git
{% endhighlight %}


[0]: https://developers.google.com/cloud-messaging/
[1]: http://guides.rubyonrails.org/getting_started.html
[2]: https://developer.android.com/training/index.html
[3]: https://developers.google.com/cloud-messaging/gcm
[4]: http://guides.rubyonrails.org/association_basics.html
[5]: http://guides.rubyonrails.org/routing.html
[6]: https://developers.google.com/cloud-messaging/http
[7]: https://console.developers.google.com/
[8]: https://developers.google.com/cloud-messaging/android/client
[9]: https://en.wikipedia.org/wiki/Ifconfig
