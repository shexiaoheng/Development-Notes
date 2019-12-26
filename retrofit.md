### 使用retrofit2遇到的问题记录

用`LievData`接收接口返回的数据：

自定义LiveDataCallAdapterFactory：

```Java
import androidx.lifecycle.LiveData;

import java.lang.annotation.Annotation;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;

import retrofit2.CallAdapter;
import retrofit2.Retrofit;
import retrofit2.internal.EverythingIsNonNull;

public class LiveDataCallAdapterFactory extends CallAdapter.Factory {

    @EverythingIsNonNull
    @Override
    public CallAdapter<?, ?> get(Type returnType, Annotation[] annotations, Retrofit retrofit) {
        if (getRawType(returnType) != LiveData.class ||
                !(returnType instanceof ParameterizedType))
            return null;
        Type responseType = getParameterUpperBound(
                0, (ParameterizedType) returnType);
        return new LiveDataCallAdapter(responseType);
    }
}
```

自定义LiveDataCallAdapter：

```Java
import androidx.lifecycle.LiveData;

import java.lang.reflect.Type;
import java.util.concurrent.atomic.AtomicBoolean;

import retrofit2.Call;
import retrofit2.CallAdapter;
import retrofit2.Callback;
import retrofit2.Response;
import retrofit2.internal.EverythingIsNonNull;

@EverythingIsNonNull
public class LiveDataCallAdapter<R> implements CallAdapter<R, LiveData<R>> {

    private Type responseType;

    LiveDataCallAdapter(Type type) {
        this.responseType = type;
    }

    @Override
    public Type responseType() {
        return responseType;
    }

    @Override
    public LiveData<R> adapt(Call<R> call) {
        return new Result(call);
    }

    class Result extends LiveData<R> {

        private Call<R> call;

        Result(Call<R> call) {
            this.call = call;
        }

        private AtomicBoolean started = new AtomicBoolean(false);

        @Override
        protected void onActive() {
            super.onActive();
            if (started.compareAndSet(false, true)) {
                call.enqueue(new Callback<R>() {
                    @Override
                    public void onResponse(Call<R> call, Response<R> response) {
                        postValue(response.body());
                    }

                    @Override
                    public void onFailure(Call<R> call, Throwable t) {
                        postValue(null);  //接口请求失败我返回null在接收的地方进行了处理
                    }
                });
            }
        }
    }

}
```

因为同一个接口在成功和失败的时候返回的值不同

成功返回：

```JSON
{
    "code": 1,
    "message": "成功",
    "data": {
        "a": "aaa",
        "b": "bb",
    }
}
```

失败返回：

```JSON
{
    "code": 1002,
    "message": "错误信息",
    "data": []
}
```

所以增加一个解析处理`CustomJsonDeserializer`：

```Java
import com.google.gson.Gson;
import com.google.gson.JsonDeserializationContext;
import com.google.gson.JsonDeserializer;
import com.google.gson.JsonElement;
import com.google.gson.JsonObject;
import com.google.gson.JsonParseException;
import com.joy.kuailiao.network.http.response.common.BaseResponse;

import java.lang.reflect.Type;


public class CustomJsonDeserializer implements JsonDeserializer<BaseResponse<?>> {

    @Override
    public BaseResponse<?> deserialize(JsonElement json,
                                       Type typeOfT,
                                       JsonDeserializationContext context)
            throws JsonParseException {
        if (json.isJsonObject()) {
            BaseResponse response = new BaseResponse();
            JsonObject jsonObject = json.getAsJsonObject();
            int code = jsonObject.get("code").getAsInt();
            response.setCode(code);
            String message = jsonObject.get("message").getAsString();
            response.setMessage(message);
            if (code == 1) {
                // 服务器定义code=1为成功，此时直接用gson解析
                Gson gson = new Gson();
                return gson.fromJson(json, typeOfT);
            } else {
                // 失败，不再解析data（上面解析过了code和message，直接返回response）
                return response;
            }
        }
        return null;
    }
}
```

定义接收数据的基类`BaseResponse`：

```Java
import com.google.gson.annotations.SerializedName;

public class BaseResponse<T> {

    @SerializedName("code")
    private int code;

    @SerializedName("message")
    private String message;

    @SerializedName("data")
    private T data;

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

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
```

在retrofit中使用：
```Java
...
Gson gson = new GsonBuilder().registerTypeHierarchyAdapter(
        BaseResponse.class, new CustomJsonDeserializer()).create();//添加自定义解析
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl(url)
        .addCallAdapterFactory(new LiveDataCallAdapterFactory()) //定义返回LiveData
        .addConverterFactory(GsonConverterFactory.create(gson))
        .client(client)
        .build();                
...
```

