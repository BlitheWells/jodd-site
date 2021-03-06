# HTTP ![HTTP](/gfx/http.png)

*HTTP* is tiny, raw HTTP client - and yet simple and convenient. It
offers a simple way to send requests and read responses. The goal is to
make things simple for everyday use; it probably can't replace
[HttpCommons][1], but can make developers happy :)

The best way to learn about *HTTP* is through examples.

## Simple GET

~~~~~ java
    HttpRequest httpRequest = HttpRequest.get("http://jodd.org");
    HttpResponse response = httpRequest.send();

    System.out.println(response);
~~~~~

All *HTTP* classes offers fluent interface, so you can write:

~~~~~ java
    HttpRequest httpRequest = HttpRequest.get("http://jodd.org").send();

    System.out.println(response);
~~~~~

You can build the request step by step:

~~~~~ java
    HttpRequest request = new HttpRequest();
    request
		.method("GET")
		.protocol("http")
		.host("srv")
		.port(8080)
		.path("/api/jsonws/user/get-user-by-id");
~~~~~

## Query parameters

Query parameters may be specified in the URL line (but then they have to
be correctly encoded).

~~~~~ java
    HttpResponse response = HttpRequest
		.get("http://srv:8080/api/jsonws/user/get-user-by-id?userId=10194")
		.send();
~~~~~

Other way is by using `query()` method:

~~~~~ java
    HttpResponse response = HttpRequest
		.get("http://srv:8080/api/jsonws/user/get-user-by-id")
		.query("userId", "10194")
		.send();
~~~~~

You can use `query()` for each parameter, or pass many arguments in one
call (varargs). You can also provide `Map<String, String>` as a
parameter too.

Note: if one query parameter is set more then once, it will not be
replaced, but converted into an `String[]` array. Use method
`removeQuery` to remove some parameter.
{: .attn}

Finally, you can reach internal query map, that actually holds all
parameters:

~~~~~ java
    Map<String, Object> httpParams = request.query();
    httpParams.put("userId", "10194");
~~~~~

## Basic Authentication

Basic authentication is made easy:

~~~~~ java
    request.basicAuthentication("test", "test");
~~~~~

## POST and form parameters

Looks very similar:

~~~~~ java
    HttpResponse response = HttpRequest
		.post("http://srv:8080/api/jsonws/user/get-user-by-id")
		.form("userId", "10194")
		.send();
~~~~~

As you can see, use `form()` in the same way to specify form parameters.
Everything what is said for `query()` applies to the `form()`.

## Upload files

Again, it's easy: just add file form parameter. Here is one real-world
example:

~~~~ java
    HttpRequest httpRequest = HttpRequest
		.post("http://srv:8080/api/jsonws/dlapp/add-file-entry")
		.form(
			"repositoryId", "10178",
			"folderId", "11219",
			"sourceFileName", "a.zip",
			"mimeType", "application/zip",
			"title", "test",
			"description", "Upload test",
			"changeLog", "testing...",
			"file", new File("d:\\a.jpg.zip")
		);

	HttpResponse httpResponse = httpRequest.send();
~~~~~

And that's really all!

## Headers

Add or reach header parameters with method `header()`. Some common
header parameters are already defined as methods, so you will find
`contentType()` etc.

## GZipped content

Just `unzip()` the response.

~~~~~ java
    HttpResponse response = HttpRequest
		.get("http://www.liferay.com")
		.acceptEncoding("gzip")
		.send();

    System.out.println(response.unzip());
~~~~~

## Use body

You can set request body manually - sometimes some APIs allow to specify
commands in it:

~~~~~ java
    HttpResponse response = HttpRequest
		.get("http://srv:8080/api/jsonws/invoke")
		.body("{'$user[userId, screenName] = /user/get-user-by-id' : {'userId':'10194'}}")
		.basicAuthentication("test", "test")
		.send();
~~~~~

Setting the body discards all previously set `form()` parameters.
{: .attn}

However, using `body()` have more sense on `HttpResponse` object, to see
the received content.

## Charsets and Encodings

By default, query and form parameters are encoded in UTF-8. This can be
changed globally in `JoddHttp`, or per instance:

~~~~~ java
    HttpResponse response = HttpRequest
		.get("http://server/index.html")
		.queryEncoding("CP1251")
		.query("param", "value")
		.send();
~~~~~

You can set form encoding similarly. Moreover, form posting detects
value of **charset** in "Content-Type" header, and if present,
it will be used.

With received content, `body()` method always returns the **raw** string
(encoded as ISO-8859-1). To get string in usable form, use method
`bodyText()`. This method uses provided **charset** from
"Content-Type" header and encodes the body string.

## Socket

As said, all communication goes through the plain `Socket`. Sometimes
you will need to configure socket before sending data. Once
`HttpRequest` is set, call `open()` first, just before `send()`:

~~~~~ java
    HttpRequest request = HttpRequest.get()...;
    Socket socket = request.open().getSocket();
    socket.setSoTimeout(1000);
    HttpResponse response = request.send();
~~~~~

## Parse from InputStreams

Both `HttpRequest` and `HttpResponse` have a method
`readFrom(InputStream)`. Basically, you can parse input stream with
these methods. This is, for example, how you can read request on server
side.

## HttpTunnel

*HTTP* is so flexible that you can easily build a HTTP tunnel with it -
small proxy between you and destination. We even give you a base class:
`HttpTunnel` class, that provides easy HTTP tunneling. It opens server
socket on one port and tunnels the whole HTTP traffic to some target
address.

[TinyTunnel][2] is one implementation that simply prints
out the whole communication to the console.


[1]: http://hc.apache.org/
[2]: https://github.com/oblac/tools/blob/master/src/jodd/tools/http/TinyTunnel.java
