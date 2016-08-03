# AidMaid API


HOST: http://aidmaid.io/api

# AidMaid

Before jumping into API endpoints, let me describe the architecture of the whole system. The server are based on Amazon EC2 instances which are running behind the Elastic Load Balancer. Database is hosted on Amazon RDS (MySql) and Apache/Slim is used to construct Restful APIs. The apps (iOS and Android) is using Amazon Cognito, that provide identity to all the app users. Currently we are not using authenticated identites, all identites are treated as guests.

All API endpoints with few exceptions (like register, login) should be called with authenticated headers. The header must have SHA1 value of user's uid (Universal Identified provided from Cognito). That SHA1(uid) would be checked before entertaining any request. If server did not find any `uid` present in the database, it will return 400 response code. That would indicate your request is not valid.

We are using Amazon SNS for push notification for both platforms. When an app run, the Amazon gives the app and uid (Cognito) and Push Arn (SNS). We would be using these two values in various API calls. 

# User Management


## User Availability [/checkuser/{username}]

To check for a username to be available in the system, you have to pass requested username 
to the {username} paramters in the URL, and the server will return 'available' boolean value.
If anything goes wrong during the process, then 'status' value of the response would be 'FAIL'
so you have to check for the valid available username before registering

Example: `/checkuser/sarmad`

+ Parameters
    + username (string)

## Check For a Username [GET] 

+ Response 200 (application/json)

        {
            "status":"OK",
            "available":false
        }


## Register [/register_email]

There are two types of registration in AidMaid, one is from the Facebook and one is in-app registration. Facebook registration is not implemented as it suppose to be. But we kind a get our system working with it.

In case of Facebook login button, user authenticate from facebook, on return we grab basic info permissions for user account. Then we just get `name` and `email` from their API. We use this information to populate our own registration page. Remember in case of Facebook login, user would not enter the `password` in registration page. So his email, name and password would be automatically populated and disabled for editing. On registration page, he just have to supply phone number.

In case of AidMaid regsitration, user will have to provide all of the required information on the screen along with his photo.

Then you can call the register_email endpoint.

For Normal user, everything is straightforward, last `isaidmaid` should be 1 in case of in-app registration.
For Facebook user, `isaidmaid` should be 0, password should be sha1(email)

In final call you should send sha1 of password

Example: In simple case user entered password as 'abcdef'
You would send in API call sha1(abcdef)
In Facebook case, lets say emai is 'abc@xyz.com'
The password you would send in the api call would be sha1(sha1(abc@xyz.com))


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
            "Status": "OK",
            "Data": {
                "status": "true",
                "points": "100"
            }
        }


## Edit Profile [/editprofile]

To update a user profile, `username` is the key for this call, the server will automatically
update the user profile on performing POST call. ON successfuly call, it will return a JSON
result which have `Status` value to be OK, otherwise it will be FAIL. 

`password' is an option paramter here, if provided then the server will change this user password
and if it is not provided, it will just update the rest of the values for the user profile.



+ Parameters
    + username (string)
    + password (optional,string)
    + fullname (string)
    + phone (string)
    + email (string)


### Edit Profile [POST] 



+ Response 200 (application/json)

        {
            "Status": "OK",
            "Data": {
                "userid": "7",
                "username": "sarmad11",
                "fullname": "sarmad makhdoom",
                "phone": "+923344",
                "email": "adil@adil.com"
            }
        }


## Login [/login]

To authenticate user for the system, this endpoint will entertain your request for login query.
There is only two required paramaters which are `username` and `password`. Username can be
actual username of the user or the email address he/she used while signup. It will return the same
JSON structure which was return after signup, `Status` will return either OK or FAIL
and `Data` field will have all the user information, like userid, fullname etc.

And yeah, previously we don't have user based system in AidMaid and users can post anonymously
in the system, so all the users who are currently not logged in and/or from the previous AidMaid version,
we have `userid` assigned 1 for them. So if user is not logged in and try to post an alert
in the system, those alerts SHOULD be posted with userid 1 not with NULL.




### Login [POST] 

+ Parameters
    + username (string)
    + password (string)



+ Response 200 (application/json)

        {
            "Status": "OK",
            "Data": {
                "userid": "7",
                "username": "sarmad11",
                "fullname": "sarmad makhdoom",
                "phone": "+923344",
                "email": "adil@adil.com"
            }
        }


## Get Posts Global [/posts_global]

To display global alerts on the Network page, this endpoint is used. It will return all global alerts which are 
posted in last 24 hours, the format of the result can be seen in the response section, mostly
it will return the `userid` who posted the alert, `fullename`, `message`, etc for the panic alert.
Use them in the application you want and format datetime accordingly.

Moreover, this takes two optional parameters for paging, like if you want to present total alerts in different pages
so you can instead of taking all the global alerts, just provide `page` and `pageSize` GET parameters
`page` starts from 0 and then you can set the `pageSize` like for 25, so you will get first page of 25 alerts,
similarly you can move further with `page` 1 and same pageSize variable.

+ Parameters
    + page (optional, integer)
    + pageSize (optional, integer)




### Global Posts [GET] 

+ Response 200 (application/json)

        [
            {
                "userid":"1",
                "fullname":"Anonymous",
                "country":"Saudi Arabia",
                "message":"I NEED HELP IMMEDIATELY PELASE",
                "lat":"21.645660000000000",
                "long":"39.122750000000000",
                "datetime":"09:36 03.12.2015"
            },
            ....
        ]
        
        
        

## Get Posts Locally [/posts_local] 


Similarly like global alerts, we can take local alerts from the `/posts_local` endpoint,
here we need to provide three required paramters in POST request, `lat`, `lon`, `rad`. Where 
lat and lon are straight-forward which represent user's latitude and longitude. `rad` is Radius here,
in local alerts we need to send a radius value and server will return all alerts from that radius starting from provided 
lat and lon values.

Moreover, paging works like global alert here as well, these are optional paramters here as well, but I suggest one should use them
in order to avoid a long list download from the server on the app.

+ Parameters
    + lat (float) - Latitude of the center poitn
    + lon (float) - Longitude of the center point
    + rad (float) - Radius of the area
    + page (optional, integer)
    + pageSize (optional, integer)




### Global Posts [GET] 

+ Response 200 (application/json)

        [
            {
                "userid":"1",
                "fullname":"Anonymous",
                "country":"Saudi Arabia",
                "message":"I NEED HELP IMMEDIATELY PELASE",
                "lat":"21.645660000000000",
                "long":"39.122750000000000",
                "datetime":"09:36 03.12.2015"
            },
            ....
        ]
        
        
        
## Post Alert [/add_alert]

In order to post an alert in AidMaid system, we have to use `/add_alert` endpoint, and all the parameters are 
required here, which are self-explainatory, on return a boolean value is return to confirm that
new alert has been posted or not.


### Login [POST] 

+ Parameters
    + userid (int)
    + lat (float)
    + lon (float)
    + country (string)
    + message (string)



+ Response 200 (application/json)

        {
            "status":"OK",
            "isPosted":false
        }


## Uplaod Profile Picture [/uploadphoto]

You have to POST to this url with image file and the user id, it works just like a simple HTML
multipart/form-data type request, make sure to resize the image before uploading, Any prefered
size that you think should be used in your app can be uploaded, there is no limit on the server side.


### Upload Image [POST] 

+ Parameters
    + userid (int)
    + imgfile (binary data)



+ Response 200 (text/html)

        {
        }


## Downlaod User Profile Image [/downloadphoto/{userid}]

To download uploaded user profile picture, you have to pass the user id to the GET url
in this request and user profile picture is returned. If user id is not present on the server
or the image is not yet uploaded, it will return default AidMaid anonymous image placeholder
image in red color.

### Download Image [GET] 

+ Response 200 (image/jpeg)

        {
        }


# Group Streaming

Let me explain how live video streaming works in v2.3 of the AidMaid, we take off the image/audio
uploading in that version and moved only to live video streaming. So currently no API is available to
do that, just functions are present to make possible the video streaming.

How this system works, first of all when a user start streaming on AidMaid, the app connects with the server
and take a `token` which is a random string of length 10. It can be letters or numbers, this token also
initialize the server for the streaming. I recommend saving that `token` in the user preference on the mobile
so that it can reused when a user stream. But if somehow it is lost, you can generate a new token
from the server. Because we are currently not removing data from the server of a particular token
that is why I am suggesting to keeping it at a safe palce. You will need this token for Red5Pro server as well,
because this is the token you will be sending to the receiver side as an SMS. And all the information
can be retrieved from the server from this token easily.

After getting the token from the server. The first thing to do is to sync user position from the 
app to the server, there is an endpoint for that, where you will post `userid`, `latitude` and `longitude`
on the server. You will keep sending this information after (either a regular interval or user changes its positions).
Select any method you feel comfortable with, I personally choosed position changed event in iOS app to sync with the server.


There is another endpoint from which you will get position, name of the user and time of last update on the server
by supplying token.


So here is the workflow of the live video stream.

1. User start streaming
2. New Token is generated from the server
3. Sender sync its position with user id with the server
4. Sender start live streaming with Red5Server and the stream name would be token
5. Sender send a SMS to the Receiver (a link with the token, explain later)
6. Receiver open that link and it will open the app
7. Receiver retrieve the information from the server with that token
8. Receiver show the information and start receving the live steream from Red5Server


**SMS Sending:** You can send SMS to the user with the link `http://aidmaid.net/live/{token}`
if the app is installed, it will open it using scheme `aidmaid://token` or open the webpage where 
it shows the live position of the user and option for tracking.

## Get Token [/stream/gettoken]

Get and init random token from the server

### Get Token [GET] 

+ Response 200 (application/json)

        {
            "Status":"Done",
            "Token":"vCkWctZEvJ"
        }

## Sync User Position [/stream/sync]

Upload information on the server

### Upload Info [POST] 

+ Parameters
    + token (string)
    + userid (int)
    + lat (float)
    + lon (float)



+ Response 200 (application/json)

        {
            "Status":"Done"
        }

## Get Stream Info [/stream/getinfo]

Get stream information like user id, name, position, last update time from the server.
Here time is UNIX timestamp, you can convert it to any format you like.

### Get Stream Information [POST] 

+ Parameters
    + token (string)



+ Response 200 (application/json)

        {
            "Status":"OK",
            "Time":123423434,
            "userid":2,
            "lat":37.234234,
            "lon":-71.234234,
            "name":"Sarmad Makhdoom",
        }
