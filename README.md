# Meteor android-ddp

今天给大家讲讲使用meteor写服务器，前端用android实现

首先我们需要下载一个工程，下载地址：[https://github.com/delight-im/Android-DDP](http://)

好了把项目克隆下来了之后，里面就有server端代码（meteor写的）和服务端代码（java写的）

#### 启动server：
	
	进入你的server文件夹运行命令      meteor deploy guoyoujin.meteor.com
	当提示你部署到meteor成功之后即可，
	
	
	// 这个方法就是我们服务器的发布了，你可以在android代码里面直接订阅,注意请把数据变成json格式
	//	mSubscriptionId = mMeteor.subscribe("publicMessages");
	Meteor.publish("database", function () {
       return EJSON.stringify(Database._collection._docs._map, null,4);
    });
	//下面这些方法就是你在android代码里面Meteor.call(methodName,listener);的回调，你可以设置带参数，带返	//	回结果，等等一些条件
	Meteor.methods({
		"getServerTime": function () {
			return Date.now();
		},
		"reset": function () {
			return Database.remove({});
		},
		"Firebase.upsert": function (query, update) {
			return Firebase.upsert(query, update);
		},
        "DatabaseCount": function(){
            //return EJSON.
            //
            // stringify({ when: Database._collection._docs._map}).toString();
            return (EJSON.stringify(Database.find().fetch()));
        }
	});
	
把android工程导入到eclipse编译器中，可以看到有三个工程，一个是source，他是一个架包，一个是androidDDP他就是我们的应用程序


###### source工程里面主要的一个类就是Meteor.java类了，里面封装好了我们需要用到的一些回调方法，
###### androidDDP工程里面只有一个activity，我们直接运行即可跑起来，和meteor一样我们现在meteor里面发布我们需要的数据，需要的回调方法，然后我们在java里面与服务器建立连接，并进行订阅，在需要的时候调用服务器的回调方法，注意方法名尽量取长一点，不让很容易报一些奇怪的错误，大家可以看下面的详细配置信息。


	public class MainActivity extends Activity implements MeteorCallback {

    private Meteor mMeteor;

    String mSubscriptionId;

    Map<String, Object> mInsertValues = new HashMap<String, Object>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 与你的服务器建立连接把下面的kickit.meteor.com替换成你自己的ip地址
        mMeteor = new Meteor("ws://kickit.meteor.com/websocket");

        // 注册回调事件
        mMeteor.setCallback(this);
    }
    
    
    
    
	//建立连接
    @Override
    public void onConnect() {
        Log.d("Poop","Connected");
    }
	//取消连接
    @Override
    public void onDisconnect(int code, String reason) {
        Log.d("Poop", "Disconnected: " + reason);
    }
    
    
    
	//当服务器数据库有你订阅的数据添加了数据，就会通过此方法告诉你添加的一些参数，你可以在这里写监听事件，通知你		的ui更新数据
    @Override
    public void onDataAdded(String collectionName, String documentID, String fieldsJson) {
        Log.d("Poop", "Data added to <"+collectionName+"> in document <"+documentID+">");
        Log.d("Poop", "    Added: "+fieldsJson);
        TextView tv = (TextView)findViewById(R.id.hello_world);
        try {
            JSONObject jo = new JSONObject(fieldsJson);
            if (jo.has("name")) {
                tv.append(jo.getString("name") + " at " + jo.getString("location") + "\n");
            }
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
    
    
	//刚建立连接时他会给你返回你所订阅的一些数据
    @Override
    public void onDataChanged(String collectionName, String documentID, String updatedValuesJson, String removedValuesJson) {
        Log.d("Poop", "Data changed in <"+collectionName+"> in document <"+documentID+">");
        Log.d("Poop", "    Updated: "+updatedValuesJson);
        Log.d("Poop", "    Removed: "+removedValuesJson);
        // TODO Update value
    }
    
    
    
    
	//当服务器数据库有你订阅的数据删除了数据，就会通过此方法告诉你删除的一些参数，你可以在这里写监听事件，通知你		的ui更新数据，告诉他删除了什么
    @Override
    public void onDataRemoved(String collectionName, String documentID) {
        Log.d("Poop", "Data removed from <" + collectionName + "> in document <" + documentID + ">");
        // TODO Remove value
    }
	//
	
	
	
    @Override
    public void onException(Exception e) {
        Log.d("Poop", "Exception");
        if (e != null) {
            e.printStackTrace();
        }
    }
    
    
    
	//在onDestroy()方法里面调用disconnect()方法，取消与服务器的连接
    @Override
    public void onDestroy() {
        super.onDestroy();
        mMeteor.disconnect();
    }
    
    
    
	//插入数据
    public void insertData(View view) {
        // insert data into a collection
        EditText et = (EditText)findViewById(R.id.event_name);
        mInsertValues.put("name", et.getText().toString());
        et = (EditText)findViewById(R.id.event_location);
        mInsertValues.put("location", et.getText().toString());
        mMeteor.insert("hangouts", mInsertValues);
    }
    
    
    
	//修改数据
    public void updateData(View view) {
        // update data in a collection
        Map<String, Object> updateQuery = new HashMap<String, Object>();
        updateQuery.put("_id", "my-key");
        Map<String, Object> updateValues = new HashMap<String, Object>();
        mInsertValues.put("_id", "my-key");
        mInsertValues.put("some-number", 5);
        mMeteor.update("tasks", updateQuery, updateValues);
    }
    
    
    
	//删除数据
    public void removeData(View view) {
        // remove data from a collection
        mMeteor.remove("tasks", "my-key");
    }
    
    
    
	//订阅数据
    public void subscribe(View view) {
        // subscribe to data from the server
        mSubscriptionId = mMeteor.subscribe("publicMessages");
    }
    
    
	//取消订阅
    public void unsubscribe(View view) {
        // unsubscribe from data again (usually done later or not at all)
        mMeteor.unsubscribe(mSubscriptionId);
    }
    
    
	//调用服务器的回调方法，相当于普通的接口,她可以有返回值也可以没有返回值，取决于你自己，我们只需要调用他的方	法即可
    public void callMethod(View view) {
        // call an arbitrary method
        mMeteor.call("/my-collection/count");
        mMeteor.call(methodName, params, new ResultListener() {
					
					@Override
					public void onSuccess(String result) {
						// TODO Auto-generated method stub
						//result就是服务器给你返回的json数据
						
						
					}
					
					@Override
					public void onError(String error, String reason, String details) {
						// TODO Auto-generated method stub
						
					}
				});
    }
    //登录方法，也是一个回调，你可以在服务器重写登录方法，返回你自己需要的信息
    public void login(final String username,final String password){
		mMeteor.loginWithUsername(username, password, new ResultListener() {		
			@Override
			public void onSuccess(String result) {
				Log.e("TAG", "login========="+result);
				doctoUser = JSON.parseObject(result, DoctorUser.class);
				doctoUser.setName(username);
				doctoUser.setPassword(password);
				Log.e("TAG", "user===="+doctoUser.toString());
				Toast.makeText(getApplicationContext(), "登录成功", Toast.LENGTH_SHORT).show();
			}
			public void onError(String error, String reason, String details) {
				Toast.makeText(getApplicationContext(), "登录失败", Toast.LENGTH_SHORT).show();
				Log.e("TAG", "error=="+error+"====reason==="+reason+"===details===="+details);
			}
		});
	}
	
	//不带参数的注册方法
	 mMeteor.registerAndLogin(username, null, password, new ResultListener() {		
		@Override
			public void onSuccess(String result) {
				// TODO Auto-generated method stub
				Log.e("TAG", "result====="+result);
				Toast.makeText(getApplicationContext(), "这册成功", Toast.LENGTH_SHORT).show();

			}			
			@Override
			public void onError(String error, String reason, String details) {
				// TODO Auto-generated method stub
				Log.e("TAG", "error=="+error+"====reason==="+reason+"===details===="+details);
				Toast.makeText(getApplicationContext(), "注册失败", Toast.LENGTH_SHORT).show();
			}
		});
		
		
		//带参数的注册方法
		HashMap<String,Object> profile=new HashMap<String, Object>();
		profile.put("name", username);
		profile.put("mobile", username);
		profile.put("name", username);
		//profile.put("username", username);
		mMeteor.registerAndLogin(username, null, password, profile, new ResultListener() {
			@Override
			public void onSuccess(String result) {
				// TODO Auto-generated method stub
				Log.e("TAG", "result===="+result.toString());
				if(result!=null){
					Toast.makeText(getApplicationContext(), "注册成功", Toast.LENGTH_SHORT).show();
				}
			}
			@Override
			public void onError(String error, String reason, String details) {
				// TODO Auto-generated method stub
				Log.e("TAG", "error=="+error+"====reason==="+reason+"===details===="+details);
				Toast.makeText(getApplicationContext(), "注册失败", Toast.LENGTH_SHORT).show();
			}
		});
    }


