## Android 

### 1. Gradle Example 
此处提供一个自用得Gradle Dependencies，由于是多年前的Dependencies，版本一定不是最新的，请参考使用
```
dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    implementation 'com.android.support:appcompat-v7:26.1.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.2'
    implementation 'com.android.support:support-v4:26.1.0'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
    implementation 'com.jakewharton:butterknife:8.8.1'
    annotationProcessor 'com.jakewharton:butterknife-compiler:8.8.1'
    implementation 'commons-io:commons-io:2.6'
    implementation 'commons-collections:commons-collections:3.2.2'

    //网络请求
    implementation 'com.squareup.retrofit2:retrofit:2.4.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.4.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:3.10.0'

    //图片加载
    implementation 'com.github.bumptech.glide:glide:4.7.1'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.7.1'

    //弹出框
    implementation 'com.orhanobut:dialogplus:1.11@aar'

    //flexBox
    implementation 'com.google.android:flexbox:1.0.0'

    //tab
    implementation 'devlight.io:navigationtabbar:1.2.5'
    implementation 'com.ogaclejapan.smarttablayout:library:1.6.1@aar'
    implementation 'com.ogaclejapan.smarttablayout:utils-v4:1.6.1@aar'

    //时间选择
    implementation 'com.wdullaer:materialdatetimepicker:3.6.2'

    //状态栏
    implementation 'com.jaeger.statusbarutil:library:1.5.1'

    //通知
    implementation 'com.github.GrenderG:Toasty:1.3.0'

    //角标
    implementation 'me.leolin:ShortcutBadger:1.1.21@aar'

    //路由
    implementation 'com.chenenyu.router:router:1.5.2'

    // 每个使用了Router注解的module都要添加该注解处理器
    annotationProcessor 'com.chenenyu.router:compiler:1.5.0'

    //JSON
    implementation 'com.squareup.moshi:moshi:1.6.0'

    //Ptr & Load More
    implementation 'com.scwang.smartrefresh:SmartRefreshLayout:1.1.0-alpha-14'
    implementation 'com.timehop.stickyheadersrecyclerview:library:0.4.3@aar'

    //Localization
    implementation 'com.akexorcist:localizationactivity:1.2.2'

    //主题
    implementation 'skin.support:skin-support:3.1.0-beta1'

    //大图查看
    implementation 'com.github.piasy:BigImageViewer:1.4.6'
    implementation 'com.github.piasy:GlideImageLoader:1.4.6'

    //权限
    implementation('com.github.hotchemi:permissionsdispatcher:3.3.1') {
        exclude module: "support-v13"
    }
    annotationProcessor "com.github.hotchemi:permissionsdispatcher-processor:3.3.1"

    //QR
    implementation 'com.journeyapps:zxing-android-embedded:3.6.0'
    implementation 'com.github.prolificinteractive:material-calendarview:1.6.0'

    //图库
    implementation 'com.github.lovetuzitong:MultiImageSelector:1.2'

    //下拉框
    implementation 'com.github.arcadefire:nice-spinner:1.3.4'

    //expand
    implementation 'com.github.traex.expandablelayout:library:1.2.2'

    //toolbar
    implementation 'com.wuhenzhizao:titlebar:1.1.3'
    implementation 'com.jaeger.statusbarutil:library:1.5.1'

    //drawer
    implementation 'com.mxn.soul:flowingdrawer-core:2.0.0'
    implementation 'com.nineoldandroids:library:2.4.0'

    //右滑返回
    implementation 'me.imid.swipebacklayout.lib:library:1.1.0'

    //右滑删除
    implementation "com.daimajia.swipelayout:library:1.2.0@aar"

    //alphabet
    implementation 'com.github.viethoa:fastscroller:1.2.0'

    //保活
    implementation project(':LibMarsdaemon')
    implementation 'org.apache.commons:commons-lang3:3.8'

    //RxJava
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.4.0'
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
    implementation 'io.reactivex.rxjava2:rxjava:2.2.2'

    implementation 'com.google.code.gson:gson:2.8.5'
    implementation 'log4j:log4j:1.2.17'
    implementation 'com.squareup.okhttp3:logging-interceptor:3.11.0'

    //pdf
    implementation 'com.github.barteksc:android-pdf-viewer:3.1.0-beta.1'
}
```

同时推几个网址
- [awesome-android-ui](https://github.com/wasabeef/awesome-android-ui)
- [awesome-android](https://github.com/JStumpp/awesome-android)
- [awesome-android-libraries](https://github.com/wasabeef/awesome-android-libraries)

### 2. Retrofit
据我所知，retrofit多用于Android上，Server之间通信一般用Eureka或者nacos等注册中心 或者是 Dubbo、Feign等RPC
```
public class ServerApi{
    private static ServerApi api;
    private Retrofit retrofit;

    private ServerApi() {
        initRetrofit(new TokenInterceptor());
    }

    private ServerApi(Interceptor interceptor) {
        initRetrofit(interceptor);
    }

    private void initRetrofit(Interceptor interceptor) {
        OkHttpClient client = new OkHttpClient.Builder()
                .connectTimeout(Constant.Http.HTTP_CONNECT_TIMEOUT, TimeUnit.SECONDS)
                .readTimeout(Constant.Http.HTTP_READ_TIMEOUT, TimeUnit.SECONDS)
                .writeTimeout(Constant.Http.HTTP_WRITE_TIMEOUT, TimeUnit.SECONDS)
                .addInterceptor(interceptor)
                .addInterceptor(new HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BODY))
                .build();

        retrofit = new Retrofit.Builder()
                .baseUrl(Env.SERVER_URL)
                .addConverterFactory(ConverterFactory.create())
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .client(client)
                .build();

    }

    public static ServerApi defaultInstance(){
        if(api == null) {
            api = new ServerApi();
        }

        return api;
    }

    public static ServerApi getInstance(HttpHeaderInterceptor interceptor) {
        api = null;
        return new ServerApi(interceptor);
    }


    public <T> T create(Class<T> service) {
        return retrofit.create(service);
    }

}
```
```
public class TokenInterceptor implements Interceptor {

    private Map<String,String> headerParams = new HashMap<>();

    public TokenInterceptor() {}

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request oldRequest = chain.request();
        Request.Builder newRequestBuilder = oldRequest.newBuilder()
                .method(oldRequest.method(), oldRequest.body())
                .addHeader("Authorization", "Bearer " + MainApplication.getInstance().getAccessToken())
                .addHeader("Content-Type", "application/json");

        return chain.proceed(newRequestBuilder.build());
    }

}
```
Common Header Interceptor
```
public class HttpHeaderInterceptor implements Interceptor {
    private Map<String,String> headerParams = new HashMap<>();

    public HttpHeaderInterceptor() {}

    @Override
    public Response intercept(Chain chain) throws IOException{
        Request oldRequest = chain.request();
        Request.Builder newRequestBuilder = oldRequest.newBuilder()
                .method(oldRequest.method(), oldRequest.body());

        if(headerParams.size() > 0){
            for(Map.Entry<String, String> params : headerParams.entrySet()){
                newRequestBuilder.header(params.getKey(), params.getValue());
            }
        }

        return chain.proceed(newRequestBuilder.build());
    }

    public static class Builder {
        HttpHeaderInterceptor httpHeaderInterceptor;

        public Builder () {
            httpHeaderInterceptor = new HttpHeaderInterceptor();
        }

        public Builder addHeaderParams(String key, int value){
            return addHeaderParams(key, String.valueOf(value));
        }

        public Builder addHeaderParams(String key, float value){
            return addHeaderParams(key, String.valueOf(value));
        }

        public Builder addHeaderParams(String key, double value){
            return addHeaderParams(key, String.valueOf(value));
        }

        public Builder addHeaderParams(String key, long value){
            return addHeaderParams(key, String.valueOf(value));
        }

        public Builder addHeaderParams(String key, String value){
            httpHeaderInterceptor.headerParams.put(key, value);
            return this;
        }

        public HttpHeaderInterceptor build() {
            return httpHeaderInterceptor;
        }
    }
}
```
ConvertFactory
```
public class ConverterFactory extends Converter.Factory {

    /**
     * Create an instance using a default {@link Gson} instance for conversion. Encoding to JSON and
     * decoding from JSON (when no charset is specified by a header) will use UTF-8.
     */
    public static ConverterFactory create() {
        return create(new Gson());
    }

    /**
     * Create an instance using {@code gson} for conversion. Encoding to JSON and
     * decoding from JSON (when no charset is specified by a header) will use UTF-8.
     */
    @SuppressWarnings("ConstantConditions") // Guarding public API nullability.
    public static ConverterFactory create(Gson gson) {
        if (gson == null) throw new NullPointerException("gson == null");
        return new ConverterFactory(gson);
    }

    private final Gson gson;

    private ConverterFactory(Gson gson) {
        this.gson = gson;
    }

    @Override
    public Converter<ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations,
                                                            Retrofit retrofit) {
        TypeAdapter<?> adapter = gson.getAdapter(TypeToken.get(type));
        return new ResponseBodyConverter<>(gson, adapter);
    }

    @Override
    public Converter<?, RequestBody> requestBodyConverter(Type type,
                                                          Annotation[] parameterAnnotations, Annotation[] methodAnnotations, Retrofit retrofit) {
        TypeAdapter<?> adapter = gson.getAdapter(TypeToken.get(type));
        return new RequestBodyConverter<>(gson, adapter);
    }
}
```
Request/Response Body Converter
```
public class RequestBodyConverter<T> implements Converter<T, RequestBody> {

    private static final MediaType MEDIA_TYPE = MediaType.parse("application/json; charset=UTF-8");
    private static final Charset UTF_8 = Charset.forName("UTF-8");

    private final Gson gson;
    private final TypeAdapter<T> adapter;

    RequestBodyConverter(Gson gson, TypeAdapter<T> adapter) {
        this.gson = gson;
        this.adapter = adapter;
    }

    @Override
    public RequestBody convert(T value) throws IOException {
        ((Request)value).setBody(trimRequest(((Request) value).getBody()));
        Buffer buffer = new Buffer();
        Writer writer = new OutputStreamWriter(buffer.outputStream(), UTF_8);
        JsonWriter jsonWriter = gson.newJsonWriter(writer);
        adapter.write(jsonWriter, value);
        jsonWriter.close();
        return RequestBody.create(MEDIA_TYPE, buffer.readByteString());
    }

    private Object trimRequest (Object object) {
        if(object == null) return null;
        Field[] fields = object.getClass().getDeclaredFields();
        for(Field field : fields) {
            Class<?> type = field.getType();
            if("class java.lang.String".equals(type.toString())) {
                String fieldName = field.getName();
                Object value = getFieldValueByName(fieldName, object);
                if(value != null) {
                    setTrimedValue(object, type, fieldName, value.toString().trim());
                }
            }
        }

        return object;
    }

    private Object getFieldValueByName(String fieldName, Object o) {
        try {
            String firstLetter = fieldName.substring(0, 1).toUpperCase();
            String getter = "get" + firstLetter + fieldName.substring(1);
            Method method = o.getClass().getMethod(getter, new Class[]{});
            Object value = method.invoke(o, new Object[]{});
            return value;
        } catch (Exception e) {
            return null;
        }
    }

    private static void setTrimedValue(Object obj,Class<?> classType,String fieldName,String value){
        String firstLetter=fieldName.substring(0,1).toUpperCase();
        String setter = "set"+firstLetter+fieldName.substring(1);
        try{
            Method method = obj.getClass().getMethod(setter, new Class[]{classType});
            method.invoke(obj, new Object[] {value});
        }catch (Exception e){
            e.printStackTrace();
        }
    }

}

public class ResponseBodyConverter<T> implements Converter<ResponseBody, T> {

    private final Gson gson;
    private final TypeAdapter<T> adapter;

    ResponseBodyConverter(Gson gson, TypeAdapter<T> adapter) {
        this.gson = gson;
        this.adapter = adapter;
    }

    @Override
    public T convert(ResponseBody value) throws IOException {

        String responseString = value.string();
        try {
            Response response = gson.fromJson(responseString, Response.class);
            switch (response.getCode()) {
                case 200:
                    return adapter.read(gson.newJsonReader(new StringReader(gson.toJson(response.getBody()))));
                default:
                    throw new ServerException(response.getCode(), response.getMessage());
            }

        } finally {
            value.close();
        }
    }
}
```
Request/Response Entity
```
public class Request<T> {

    public Request(T body, HashMap<String, String> secure) {
        this.body = body;
        this.secure = secure;
    }

    public Request(T body) {
        this.body = body;
    }

    public Request() {
    }

    private T body;
    private HashMap<String, String> secure = null;

    public void setSecure(HashMap<String, String> secure) {
        this.secure = secure;
    }

    public void setBody(T body) {
        this.body = body;
    }

    public T getBody() {
        return body;
    }
}


public class Response<T> {
    private int code;
    private String message;
    private T body;

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public T getBody() {
        return body;
    }

    public void setBody(T body) {
        this.body = body;
    }
}
```