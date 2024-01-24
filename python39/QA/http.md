如果你正在使用`CloseableHttpClient`，你可以通过创建一个自定义的`HttpRequestInterceptor`来为所有的对外的HTTP请求增加统一的请求头。`HttpRequestInterceptor`可以在发送请求之前修改请求，例如添加或修改请求头。

以下是一个简单的示例：

```java
import org.apache.http.HttpException;
import org.apache.http.HttpRequest;
import org.apache.http.HttpRequestInterceptor;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.protocol.HttpContext;

import java.io.IOException;

public class CustomHttpClient {

    public static void main(String[] args) throws IOException {
        HttpRequestInterceptor requestInterceptor = new HttpRequestInterceptor() {
            public void process(final HttpRequest request, final HttpContext context) throws HttpException, IOException {
                request.addHeader("id", "your_id_value");
                request.addHeader("key", "your_key_value");
            }
        };

        CloseableHttpClient httpclient = HttpClients.custom()
                .addInterceptorFirst(requestInterceptor)
                .build();

        try {
            HttpGet httpget = new HttpGet("http://www.example.com/");
            CloseableHttpResponse response = httpclient.execute(httpget);
            try {
                // handle response here
            } finally {
                response.close();
            }
        } finally {
            httpclient.close();
        }
    }
}
```

在这个示例中，我们创建了一个`HttpRequestInterceptor`，在这个拦截器中，我们添加了"id"和"key"的请求头。然后，我们使用`HttpClients.custom().addInterceptorFirst(requestInterceptor).build()`来创建一个`CloseableHttpClient`，这个`CloseableHttpClient`会在每个请求发送之前运行我们的拦截器。

然后，你可以使用这个`CloseableHttpClient`来发送HTTP请求。这样，所有通过这个`CloseableHttpClient`发送的请求都会包含你添加的请求头。

请注意，你需要将"your_id_value"和"your_key_value"替换为你想要设置的实际值。
