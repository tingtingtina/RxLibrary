[![API](https://img.shields.io/badge/API-15%2B-brightgreen.svg)](https://android-arsenal.com/api?level=15) [![License](https://img.shields.io/badge/license-Apache%202-green.svg)](https://www.apache.org/licenses/LICENSE-2.0)[ ![Download](https://api.bintray.com/packages/bikie/bhm-sdk/RxLibrary/images/download.svg) ](https://bintray.com/bikie/bhm-sdk/RxLibrary/_latestVersion)
-----

RxLibrary工程：<br>1.rxjava2 + retrofit2的封装，常用的请求（Get,Post,文件上传，文件下载），防止内存泄漏，简单便捷，支持自定义loading等属性。</br>2.RxBus的使用，用法完全与EvenBus一样。
=
效果图
------
>![image](https://github.com/buhuiming/RxLibrary/blob/master/screenShots/2.jpg)</br>

<br>

集成
-------
    maven { url "https://dl.bintray.com/bikie/bhm-sdk" } //根目录下build.gradle的allprojects-repositories节点下添加

    compile 'com.bhm.sdk.rxlibrary:RxLibrary:2.4.4'
<br>或者

    <dependency>
      <groupId>com.bhm.sdk.rxlibrary</groupId>
      <artifactId>RxLibrary</artifactId>
      <version>2.4.3</version>
      <type>pom</type>
    </dependency>


一、rxjava2 + retrofit2的使用
-------  
### 强烈建议参考demo，MianActivity包含了常用的用法及用法介绍。
### 第一步，继承RxBaseActivity或者RxBaseFragment（不继承的话，就必须父类是RxJava包下的RxAppCompatActivity）,添加内存管理的机制，同时获取或生成rxManager对象，rxManager是管理观察者的类，当取消请求、中断请求等可调用对应的方法。注意：如果项目BaseActivity继承了别的Activity，则需在BaseActivity中添加别的Activity代码再继承RxAppCompatActivity类，并且生成RxManager对象，解析的实体类必须继承BaseResponse类。Fragment同理哦。
### 第二步，使用。

        RxBuilder builder = RxBuilder.newBuilder(this)
                //.setLoadingDialog(RxLoadingDialog.getDefaultDialog())
                .setLoadingDialog(new MyLoadingDialog())
                .setDialogAttribute(true, false, false)
                //.setHttpTimeOut()
                .setIsLogOutPut(true)//默认是false
                .setIsDefaultToast(true, rxManager)
                .bindRx();
        Observable<DoGetEntity> observable = builder
                .createApi(HttpApi.class, "http://news-at.zhihu.com")
                .getData("Bearer aedfc1246d0b4c3f046be2d50b34d6ff", "1");
        builder.setCallBack(observable, new CallBack<DoGetEntity>() {
            @Override
            public void onStart(Disposable disposable) {

            }

            @Override
            public void onSuccess(DoGetEntity response) {
                Log.i("MainActivity--> ", response.getDate());
                Toast.makeText(MainActivity.this, response.getDate(), Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onFail(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
<br>

### HttpApi是一个接口，getData是HttpApi中的方法。</br>
### Host 为域名。</br>                         
### 以下介绍下HttpApi中常用注解的使用：</br>
        1.get请求：方法前添加@GET("url"),参数 @Query、@QueryMap
        <br>例（url和域名拼接好后组成完整链接，当然我们不需要自己拼接，Retrofit会处理）</br>
        @GET("福利/10/1")
        Observable<Object> getData(@Header("token") String token, @Query("type")  String type);     
        2.post请求：方法前添加@FormUrlEncoded和@POST("url")
          Observable<Object> getDataPost(@Field("once") boolean once_no);
        3.文件上传：方法前添加@Multipart和@POST("url")
            Observable<Object> upload(
            @Header("token") String token,
            @Part("filename") RequestBody description,
            @Part("id") RequestBody id,
            @Part MultipartBody.Part file);
           其中RequestBody的生成：
           RequestBody.create(MediaType.parse("text/plain"),"id"
           MultipartBody.Part的生成：
           RequestBody requestBody = RequestBody.create(MediaType.parse("image/jpeg; charset=UTF-8"),file);
           MultipartBody.Part part= MultipartBody.Part.createFormData("file", file.getName(), requestBody);
        4.文件下载：方法前添加@Streaming和@GET
        @Streaming
        @GET
        Observable<ResponseBody> downLoad(@Url String url);
        其中@GET后不需指定url，参数@Url指定了完整的url，所以也不需域名Host的指定。
        
二、RxBus的使用
-------  

### 1.订阅RxBus.get().register(this);
    一般在activity/fragment的oncreate方法中添加
### 2.取消订阅RxBus.get().unRegister(this);
     一般在activity/fragment的的ondestory方法添加
### 3.接收event，处理
    在activity/fragment中添加如下方法
      
    @Subscribe(code = 1111,threadMode = ThreadMode.MAIN)
    public void rxBusEvent(Entity entity){
        if(null != entity){
        Toast.makeText(this, "RxBus改变了MainActivity的标题", Toast.LENGTH_SHORT).show();      
        }
    }
       
    threadMode：MAIN，NEW_THREAD，CURRENT_THREAD（默认）.
### 4.在任意一个地方发送
      RxBus.get().send(1111, entity);
