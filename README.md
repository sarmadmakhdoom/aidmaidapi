# AidMaid API


HOST: http://aidmaid.io/api

# AidMaid

Before jumping into API endpoints, let me describe the architecture of the whole system. The server are based on Amazon EC2 instances which are running behind the Elastic Load Balancer. Database is hosted on Amazon RDS (MySql) and Apache/Slim is used to construct Restful APIs. The apps (iOS and Android) is using Amazon Cognito, that provide identity to all the app users. Currently we are not using authenticated identites, all identites are treated as guests.

All API endpoints with few exceptions (like register, login) should be called with authenticated headers. The header must have SHA1 value of user's uid (Universal Identified provided from Cognito). That SHA1(uid) would be checked before entertaining any request. If server did not find any `uid` present in the database, it will return 400 response code. That would indicate your request is not valid.

We are using Amazon SNS for push notification for both platforms. When an app run, the Amazon gives the app and uid (Cognito) and Push Arn (SNS). We would be using these two values in various API calls. 

# User Management


## Register [/register_email]

There are two types of registration in AidMaid, one is from the Facebook and one is in-app registration. Facebook registration is not implemented as it suppose to be. But we kind a get our system working with it.

In case of Facebook login button, user authenticate from facebook, on return we grab basic info permissions for user account. Then we just get `name` and `email` from their API. We use this information to populate our own registration page. Remember in case of Facebook login, user would not enter the `password` in registration page. So his email, name and password would be automatically populated and disabled for editing. On registration page, he just have to supply phone number.

In case of AidMaid regsitration, user will have to provide all of the required information on the screen along with his photo. Cognito `uid` should be sent in the headers as `uid` field

Then you can call the register_email endpoint.

For Normal user, everything is straightforward, last `isaidmaid` should be 1 in case of in-app registration.
For Facebook user, `isaidmaid` should be 0, password should be sha1(email)

In final call you should send sha1 of password

Example: In simple case user entered password as 'abcdef'
You would send in API call sha1(abcdef)
In Facebook case, lets say emai is 'abc@xyz.com'
The password you would send in the api call would be sha1(sha1(abc@xyz.com))

Save all the information in your local storage as well, other than password, you will be using this information in other screens of the app.

+ Parameters
    + name (string)
    + phone (string)
    + pusharn (string)
    + email (string)
    + password (string)
    + isaidmaid


### Register a User [POST] 


+ Response 200 (application/json)

        {
            "Status": "true",
            "points": "100"
        }



## Picture Upload [/picture_upload]

After registering the user on the server, you should upload his/her profile picture using this endpoint. It uses multipart request to upload the image. This would be first authenticated request you would be making. Add sha1(uid) in the request header for `uid` field. And `image` in the form data and post the request. This would be stored on the server, don't worry about he response at this moment. If you have constructed the right request, you can see the actualy image on `http://aidmaid.io/api/picture/sha1(uid)` 
You can see my picture at ``


## Login [/login]

In Login, make the POST request to the endpoint, put `uid` in the request header and call the API endpoint with `email` and `password` fields.

Make sure you send the sha1 value of the password in the API call, plain password won't authenticate the user from the server. In response please store all the values in your local storage for further call. Here store the returned `uid` from the response and use this `uid` in later requests for authentication. Not the one which is provided by Cognito. That will be only used once for registering the user. Now at this points you might have different cognito identity and server returned identity. But you carry on server returned `uid`

+ Parameters
    + email (string)
    + password (string)

+ Response 200 (application/json)

        {
            "status": "true",
            "user": {
                "name": "Sarmad Makhdoom",
                "phone": "+92334444444",
                "uid":  "us-east-1:123xxx....",
                "points": "points"
            }
        }



## Reset Password [/reset_password]

In order to reset the user password, take its valid email address from the app and sent it to the `/reset_password` endpoint. Don't worry about the response here if you have valid request. Server will automatically call the reset password routine, send email, allow user to change the password from the link sent in the email. You don't have to worry about that at this moment.

+ Parameters
    + email (string)


