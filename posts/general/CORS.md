### Dealing With Cross Site Origin Restrictions

<img src="https://images.unsplash.com/photo-1605322054569-567f7c6a1713?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1789&q=80" alt="unsplash_img" style="zoom:70%;" />

I started out with testing an issue which I was facing on my setup. My setup included an API server written in Golang and a HTML client code which is to be run on browser. This backend server hosted HTTP and Websocket API endpoints:

* `/api/v1/login`: To authenticate the user and get a login token. This login token is sent to client as part of HTTP Cookies in response headers
* `/api/v1/auth/showdata`: All subsequent requests will have above cookie set and hence server can authenticate such requests based on associated cookie
* `/ws/api/v1/auth/streamdata`: Websocket API endpoint is used to stream data. Since this is also an authenticated request, browser should set cookie for every request to Websocket API

To achieve above cookie based authentication, I planned to test it out locally with minimal code before applying my changes on production code. So two main things to be achieved once Cookie is sent as part of Login response:

1. Ensure that the cookie is set for all the authenticated HTTP API requests by the Browser
2. Ensure that the cookie is set for all the authenticated Websocket API requests by the Browser

#### Local Testing with cross origin requests

I performed my testing on Chrome browser (Version 90) and below steps are only tested on the mentioned browser. But before we look at the cross-origin requests, we must understand CORS restriction. Here's a nice [article](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) to understand this.

* My local setup uses a simple HTTP server by python to host sample HTML side code:
  `python -m SimpleHTTPServer 8800`
* Started Golang server to host API endpoints on port 1323

Hence we deal with two different origins:

1. `http://localhost:8800` : Python Webserver
2. `http://localhost:1323`: Golang Backend Server

When we open URL #1, it makes requests to URL #2. This is cross origin requests and it is blocked by browsers for security reasons

##### Sample code:

* Golang API server (test_server.go):

  ```go
  package main
  
  import (
          "net/http"
          "time"
  
          jwt "github.com/dgrijalva/jwt-go"
          "github.com/labstack/echo"
          "github.com/labstack/echo/middleware"
  )
  
  func login(c echo.Context) error {
          claims := jwt.StandardClaims{
                  IssuedAt: time.Now().Unix(),
                  // 1 day expiration for now
                  ExpiresAt: time.Now().AddDate(0, 0, 1).Unix(),
          }
          token := jwt.NewWithClaims(jwt.SigningMethodHS512, &claims)
          // Set ID just for local testing
          jwtToken, err := token.SignedString([]byte("testkey"))
          if err != nil {
                  return err
          }
          httpCookie := http.Cookie{
                  Name:    "token",
                  Value:   jwtToken,
                  Expires: time.Unix(claims.ExpiresAt, 0),
          }
          c.SetCookie(&httpCookie)
          return c.JSON(http.StatusOK, "Logged in successfully")
  }
  
  func show_data(c echo.Context) error {
          jwtToken := ""
          for _, httpCookie := range c.Request().Cookies() {
                  if httpCookie.Name == "token" {
                          jwtToken = httpCookie.Value
                          break
                  }
          }
          if jwtToken == "" {
                  return c.JSON(http.StatusUnauthorized, "No token found")
          }
          token, err := jwt.Parse(jwtToken, func(token *jwt.Token) (interface{}, error) {
                  return []byte("testkey"), nil
          })
          if !token.Valid || err != nil {
                  return c.JSON(http.StatusUnauthorized, "token is invalid")
          }
          sampleData := []string{
                  "test1",
                  "test2",
          }
          return c.JSON(http.StatusOK, sampleData)
  }
  
  func main() {
          e := echo.New()
          e.Use(middleware.Logger())
          e.Use(middleware.Recover())
  
          e.POST("/api/v1/login", login)
          e.POST("/api/v1/auth/showdata", show_data)
          e.Logger.Fatal(e.Start(":1323"))
  }
  ```

  Started from terminal using the following command:

  ```
  ❯ go run test_server.go
  
     ____    __
    / __/___/ /  ___
   / _// __/ _ \/ _ \
  /___/\__/_//_/\___/ v3.3.6
  High performance, minimalist Go web framework
  https://echo.labstack.com
  ____________________________________O/_______
                                      O\
  ⇨ http server started on [::]:1323
  ```

* HTML client (test_client.html):

  ```html
  <!DOCTYPE HTML>
  <html>
     <head>
        <script type = "text/javascript">
           function Login() {
            const Http = new XMLHttpRequest();
            const url='http://localhost:1323/api/v1/login';
            Http.open("POST", url, true);
            Http.send();
  
            Http.onreadystatechange = (e) => {
              console.log("Login", Http.responseText)
            }
           }
  
           function ShowData() {
            const Http = new XMLHttpRequest();
            const url='http://localhost:1323/api/v1/auth/showdata';
            Http.open("POST", url, true);
            Http.send();
  
            Http.onreadystatechange = (e) => {
              console.log("Data", Http.responseText)
            }
           }
        </script>
     </head>
     <body>
        <div id = "login">
           <a href = "javascript:Login()">Login</a>
        </div>
        <div id = "showdata">
           <a href = "javascript:ShowData()">ShowData</a>
        </div>
     </body>
  </html>
  ```

Now because we are dealing with different origins, requests to backend server from browser via webserver will not go through. Here's browser console log on perfoming the following actions:

1. Open `http://localhost:1323/test_client.html`
2. Click `Login` text

```
Access to XMLHttpRequest at 'http://localhost:1323/api/v1/login' from origin 'http://localhost:8800' has been blocked by CORS policy: The 'Access-Control-Allow-Origin' header contains the invalid value ''.
```

To allow access to such cross origins, we use the following CORS policy in our API server code:

```diff
@@ -65,6 +65,13 @@ func main() {
        e.Use(middleware.Logger())
        e.Use(middleware.Recover())

+       corsConfig := middleware.DefaultCORSConfig
+       corsConfig.AllowOrigins = []string{"http://localhost:8800"}
+       e.Use(middleware.CORSWithConfig(corsConfig))
+
        e.POST("/api/v1/login", login)
        e.POST("/api/v1/auth/showdata", show_data)
        e.Logger.Fatal(e.Start(":1323"))
```

Now when we open `http://localhost:8800/test_client.html` and click `Login` text, it will go through

But then when we click `ShowData` text, we see following error in Browser's console:

```
test_client.html:22 POST http://localhost:1323/api/v1/auth/showdata 401 (Unauthorized)
```

This is because we didn't set [XMLHttpRequest.withCredentials](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/withCredentials) it to true. This is required for browser to store cookie or any other credentials like authorization headers or TLS client certificates for cross-site requests

Hence we'll need the following changes:

1. In HTML client code, set `withCredentials` to true:

   ```diff
   @@ -6,6 +6,7 @@
              const Http = new XMLHttpRequest();
             const url='http://localhost:1323/api/v1/login';
             Http.open("POST", url, true);
   +         Http.withCredentials = true;
             Http.send();
   
             Http.onreadystatechange = (e) => {
   @@ -17,6 +18,7 @@
              const Http = new XMLHttpRequest();
             const url='http://localhost:1323/api/v1/auth/showdata';
             Http.open("POST", url, true);
   +         Http.withCredentials = true;
             Http.send();
   ```

2. In API server code, update CORS policy:

   ```diff
   @@ -67,6 +67,7 @@ func main() {
   
           corsConfig := middleware.DefaultCORSConfig
           corsConfig.AllowOrigins = []string{"http://localhost:8800"}
   +       corsConfig.AllowCredentials = true
           e.Use(middleware.CORSWithConfig(corsConfig))
   
           e.POST("/api/v1/login", login)
   ```

This should now fix the cookie issue.

##### Summary

* Cross site requests are restricted by Browser due to security reasons
* Server should explicitly list which all origins are allowed
* For browsers to deal with credentials from cross sites (different origin), it must ensure that server allows those credentails to be stored. This requires changes from both client and server code, they should explicitly make the intent clear
* I haven't mentioned about websockets here, will be doing that as part of next blog