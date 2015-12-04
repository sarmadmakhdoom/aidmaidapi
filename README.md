# AidMaid API


HOST: http://aidmaid.net/api/v3

# AidMaid

Here are the details of complete list of operations we need to integrade in our iOS and Android apps.
You must have to use this API in order to make the calls to the server to keep data consitant between two platforms.

# User Management

Here you will find operations to register, login or edit profile for a specific user in AidMaid system

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


## Register [/register]

In order to register a user in the system, you have to send POST call to the /register url
with the data for everything, on successfuly user creating, the server will result JSON 
with Stauts = OK, and Data field with required data for other functions. You have to specially
take care of `userid` and store it in
the app repository for later use, because if user want to
post any alert, then it should have to send this user id, for edit profile/change password, for everything
this `userid` is a must


+ Parameters
    + username (string)
    + password (string)
    + fullname (string)
    + phone (string)
    + email (string)


### Register a User [POST] 


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

