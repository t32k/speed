原文: https://developers.google.com/speed/docs/best-practices/request

# リクエストオーバーヘッドを最小化する

Every time a client sends an HTTP request, it has to send all associated cookies that have been set for that domain and path along with it. Most users have asymmetric Internet connections: upload-to-download bandwidth ratios are commonly in the range of 1:4 to 1:20. This means that a 500-byte HTTP header request could take the equivalent time to upload as 10 KB of HTTP response data takes to download. The factor is actually even higher because HTTP request headers are sent uncompressed. In other words, for requests for small objects (say, less than 10 KB, the typical size of a compressed image), the data sent in a request header can account for the majority of the response time.

The latency can be higher at the beginning of a new browser session. To minimize net congestion, TCP employs the "slow start" algorithm for new connections. This limits the amount of data that can be sent by the initiator of the connection before the data must be acknowledged by the recipient. If the initiator sends more than that amount of data over a new connection, an additional RTT is incurred. 

The best way to cut down on client request time is to reduce the number of bytes uploaded as request header data.

+ Serve static content from a cookieless domain
+ Minimize request size


## リクエストサイズを最小化する


### 概要

Keeping cookies and request headers as small as possible ensures that an HTTP request can fit into a single packet.

### 詳細

Ideally, an HTTP request should not go beyond 1 packet. The most widely used networks limit packets to approximately 1500 bytes, so if you can constrain each request to fewer than 1500 bytes, you can reduce the overhead of the request stream. HTTP request headers include:

Cookies: For resources that must be sent with cookies, keep the cookie sizes to a bare minimum.  To keep the request size within this limit, no one cookie served off any domain should be more than 1000 bytes. We recommend that the average size of cookies served off any domain be less than 400 bytes.
Browser-set fields: Many of the header fields are automatically set by the user agent, so you have no control over them. 
Requested resource URL (GET and Host fields). URLs with multiple parameters can run into the thousands of bytes. Try to limit URL lengths to a few hundred bytes at most.
Referrer URL.

### 推奨

Use server-side storage for most of the cookie payload.
Store only a unique identifier in the cookie, and key the ID to data stored at the server end. You can use server-side cookies for both session and persistent cookies by specifying the expiry date/time on the cookie.
Remove unused or duplicated cookie fields.
The fields set by a cookie at the top-level path of a domain (i.e. /) are inherited by the resources served off all paths below that domain. Therefore, if you are serving different applications on different URL paths, and you have a field that applies globally to all applications on a domain — for example, a user's language preference — include that field in the cookie set at the top-level domain; don't duplicate the field in cookies set for subpaths. Conversely, if a field only applies to an application served from a subpath — for example, a UI setting — don't include that field in the top-level cookie and force the unused data to be passed needlessly for other applications.

Back to top

Serve static content from a cookieless domain

### 概要

Serving static resources from a cookieless domain reduces the total size of requests made for a page.

### 詳細

Static content, such as images, JS and CSS files, don't need to be accompanied by cookies, as there is no user interaction with these resources. You can decrease request latency by serving static resources from a domain that doesn't serve cookies. This technique is especially useful for pages referencing large volumes of rarely cached static content, such as frequently changing image thumbnails, or infrequently accessed image archives. We recommend this technique for any page that serves more than 5 static resources. (For pages that serve fewer resources than this, it's not worth the cost of setting up an extra domain.)  
To reserve a cookieless domain for serving static content, register a new domain name and configure your DNS database with a CNAME record that points the new domain to your existing domain A record. Configure your web server to serve static resources from the new domain, and do not allow any cookies to be set anywhere on this domain. In your web pages, reference the domain name in the URLs for the static resources.

If you host your static files using a CDN, your CDN may support serving these resources from another domain. Contact your CDN to find out.


### 推奨

Enable proxy caching.
For resources that rarely change, set caching headers for browsers and proxies. Because cookies will not be sent for these resources, there is no risk that proxy caches will cache user-specific content.
Don't serve early loaded external JS files from the cookieless domain.
For JavaScript referenced in the head of the document and needed for page startup, it should be served from the same hostname as the main document. Because most browsers block other downloads and rendering until all JavaScript files have been downloaded, parsed and executed, it's better to avoid the risk of an additional DNS lookup at this point of processing.
Example

Many Google properties, including News and Code (this site), serve static resources, such as JS files and images, from a separate domain, www.gstatic.com. No cookies can be set on this domain. For the News homepage at news.google.com, you can see the cookie in the request header in this screen shot:



But for www.gstatic.com, which is used to serve the News logo .gif file, no cookie headers appear in the request or response:



