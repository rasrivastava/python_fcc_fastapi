# JWT Token Authentication

![Screenshot 2021-11-16 at 23 43 00](https://user-images.githubusercontent.com/11652564/142041963-cd1b5578-4aeb-44c6-9ae2-230975cccdce.png)

(1) (`Client`): `Client` will Login with username and password to the `server` using the API created

(2) (`API`): if credentials are valid then the API will sigh a JWT token and send back the token to the client

(3) (`Client`): Client request for a post, so, the post id along with token will be sent to API, API will first verify the token

(4) (`API`) if the toke matched, then the related data will be sent to the client

![Screenshot 2021-11-16 at 23 47 31](https://user-images.githubusercontent.com/11652564/142042596-9fc68b08-41bc-4899-8690-3f2739a013a7.png)
