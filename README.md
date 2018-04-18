```
   _____ _                 _        ______ _ _         _____ _
  / ____(_)               | |      |  ____(_) |       / ____| |
 | (___  _ _ __ ___  _ __ | | ___  | |__   _| | ___  | (___ | |_ ___  _ __ __ _  __ _  ___
  \___ \| | '_ ` _ \| '_ \| |/ _ \ |  __| | | |/ _ \  \___ \| __/ _ \| '__/ _` |/ _` |/ _ \
  ____) | | | | | | | |_) | |  __/ | |    | | |  __/  ____) | || (_) | | | (_| | (_| |  __/
 |_____/|_|_| |_| |_| .__/|_|\___| |_|    |_|_|\___| |_____/ \__\___/|_|  \__,_|\__, |\___|
                    | |                                                          __/ |
                    |_|                                                         |___/
```

This is a basic implementation of a RESTful API for simple file storage. It allows users to
register for an account and sign in to get a Bearer authorization token which is required
for uploading, fetching, and deleting files. The files are all stored in an Amazon AWS S3
bucket, so it is necessary for users to have an AWS account with correct credentials and a
remote bucket to store files in.

Users
- POST - `/register` - Create an account with username, password, and email address
- POST - `/login` - Sign into account to receive back a token

Files
- PUT - `/files/<filename>` - Upload a file to the users personal storage
- GET - `/files/:<filename>?` - Get a single file or a list of all files owned by the user
- DELETE - `/files/<filename>` - Delete a single file

## Setup

A number of steps are necessary to setup this RESTful API.

- Clone the git repository from [stevegcarpenter/simple-storage-server](https://github.com/stevegcarpenter/simple-storage-server)
- Navigate to the repository and run `npm install` inside to install all necessary node packages
  - Directions to install the `npm` package manager can be found [here](https://www.npmjs.com/get-npm) for those that do not already have this installed
- Install a tool to make HTTP requests. [HTTPie](https://httpie.org/) is the tool used for this guide.
  - Another popular alternative is [Postman](https://www.getpostman.com/)
- Sign up for an AWS account and create a user with read/write permissions to an S3 bucket you create
  - Safely store the access key id and secret access key as these will be needed
- Setup both a `__test__/lib/.test.env` and `.env` file as follows
  - It is recommended to use another port for testing such as 4000 and possibly a test S3 bucket as well
```
PORT=3000
MONGODB_URI='mongodb://localhost/sfsdb'
APP_SECRET='<a-long-random-string>'

AWS_BUCKET='<your-s3-bucket-name>'
AWS_ACCESS_KEY_ID='<your-access-key-id>'
AWS_SECRET_ACCESS_KEY='<your-secret-access-key>'
```

## Using the API

Once all the setup is complete, it is possible to run all the unit and route tests by executing
`npm run test` in the terminal within the project directory. This will report on any bugs and
display overall coverage statistics for tests. The remaining portion of this section details how
to use [HTTPie](https://httpie.org/) to interact with the RESTful API.

### Create an account
```
http POST :3000/register username=<desired-username> password=<desired-password> email=<your-email@domain.com>

Example:
========
http POST :3000/register username=steve password=password email=s@s.com

HTTP/1.1 204 No Content
Access-Control-Allow-Origin: *
Connection: keep-alive
Date: Wed, 18 Apr 2018 07:51:31 GMT
ETag: W/"a-bAsFyilMr4Ra1hIU5PyoyFRunpI"
X-Powered-By: Express
```

### Login to get token
```
http -a <username>:<password> POST :3000/login

Example:
========
http -a steve:password POST :3000/login

HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Length: 293
Content-Type: application/json; charset=utf-8
Date: Wed, 18 Apr 2018 05:42:44 GMT
ETag: W/"125-oXk6pPGfke6GsghTVGz6zZ5O7Jc"
X-Powered-By: Express

"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbiI6IjkyMmFiYzZiYjAyYzI3NjE5YTA1YjA3NmM0OGJiNTk4MWUwODM3Y2UwOTQ0OGIxOGQ0ODVjNDQ2YzIyZmI5OTc5YjMwN2NmZjUwYTZkMzYwZWZhMjM3NmNlZmFlNWQ1ZWQ0MTUxODEwZTI2ODhlMzkwYjA3NjYwNDdmNTIwNTMxIiwiaWF0IjoxNTI0MDMwMTY0fQ.RpYsuVoH_JHLyls0D9c95C1h6LlBqDnJgU72alY1Ma0"
```

### Upload a file
```
http -f PUT :3000/files/<desired-filenamet> file@</path/to/file.ext> 'Authorization:Bearer
<token>'

Example:
========
http -f PUT :3000/files/baz.jpg file@~/Downloads/2.jpg 'Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbiI6IjkyMmFiYzZiYjAyYzI3NjE5YTA1YjA3NmM0OGJiNTk4MWUwODM3Y2UwOTQ0OGIxOGQ0ODVjNDQ2YzIyZmI5OTc5YjMwN2NmZjUwYTZkMzYwZWZhMjM3NmNlZmFlNWQ1ZWQ0MTUxODEwZTI2ODhlMzkwYjA3NjYwNDdmNTIwNTMxIiwiaWF0IjoxNTI0MDMwMTY0fQ.RpYsuVoH_JHLyls0D9c95C1h6LlBqDnJgU72alY1Ma0'

HTTP/1.1 201 Created
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Length: 29
Content-Type: application/json; charset=utf-8
Date: Wed, 18 Apr 2018 06:56:51 GMT
ETag: W/"1d-y8BomfIcNC7oA5hAn9SCxxmrC48"
X-Powered-By: Express

{
    "Location": "/files/baz.jpg"
}
```

### Get list of all uploaded filenames
```
http GET :3000/files 'Authorization:Bearer <token>'

Example:
========
http GET :3000/files 'Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbiI6IjkyMmFiYzZiYjAyYzI3NjE5YTA1YjA3NmM0OGJiNTk4MWUwODM3Y2UwOTQ0OGIxOGQ0ODVjNDQ2YzIyZmI5OTc5YjMwN2NmZjUwYTZkMzYwZWZhMjM3NmNlZmFlNWQ1ZWQ0MTUxODEwZTI2ODhlMzkwYjA3NjYwNDdmNTIwNTMxIiwiaWF0IjoxNTI0MDMwMTY0fQ.RpYsuVoH_JHLyls0D9c95C1h6LlBqDnJgU72alY1Ma0'

HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Length: 31
Content-Type: application/json; charset=utf-8
Date: Wed, 18 Apr 2018 07:04:13 GMT
ETag: W/"1f-1AgUjnH/5sIvcsHcPjUSHTcDRBw"
X-Powered-By: Express

[
    "foo.jpg",
    "bar.jpg",
    "baz.jpg"
]
```

### Get a single file
```
http GET :3000/files/<filename> 'Authorization:Bearer <token>'

Example:
========
http GET :3000/files/foo.jpg 'Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbiI6IjkyMmFiYzZiYjAyYzI3NjE5YTA1YjA3NmM0OGJiNTk4MWUwODM3Y2UwOTQ0OGIxOGQ0ODVjNDQ2YzIyZmI5OTc5YjMwN2NmZjUwYTZkMzYwZWZhMjM3NmNlZmFlNWQ1ZWQ0MTUxODEwZTI2ODhlMzkwYjA3NjYwNDdmNTIwNTMxIiwiaWF0IjoxNTI0MDMwMTY0fQ.RpYsuVoH_JHLyls0D9c95C1h6LlBqDnJgU72alY1Ma0'

HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Length: 256
Content-Type: application/json; charset=utf-8
Date: Wed, 18 Apr 2018 07:10:53 GMT
ETag: W/"100-dvRDAcv/tW6yEs7tQw4p2zcb8/U"
X-Powered-By: Express

[
    {
        "__v": 0,
        "_id": "5ad6ec1b352970419214e0e8",
        "fileURI": "https://simple-file-storage.s3.amazonaws.com//files/1a05a54203f36ab860f2ad83ea224080.jpg",
        "name": "foo.jpg",
        "objectKey": "/files/1a05a54203f36ab860f2ad83ea224080.jpg",
        "userId": "5ad6c848475c0e138d3df20e"
    }
]
```

### Delete a file
```
http DELETE :3000/files/<filename> 'Authorization:Bearer <token>'

Example:
========
http DELETE :3000/files/foo.jpg 'Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbiI6IjkyMmFiYzZiYjAyYzI3NjE5YTA1YjA3NmM0OGJiNTk4MWUwODM3Y2UwOTQ0OGIxOGQ0ODVjNDQ2YzIyZmI5OTc5YjMwN2NmZjUwYTZkMzYwZWZhMjM3NmNlZmFlNWQ1ZWQ0MTUxODEwZTI2ODhlMzkwYjA3NjYwNDdmNTIwNTMxIiwiaWF0IjoxNTI0MDMwMTY0fQ.RpYsuVoH_JHLyls0D9c95C1h6LlBqDnJgU72alY1Ma0'

HTTP/1.1 204 No Content
Access-Control-Allow-Origin: *
Connection: keep-alive
Date: Wed, 18 Apr 2018 07:44:28 GMT
ETag: W/"a-bAsFyilMr4Ra1hIU5PyoyFRunpI"
X-Powered-By: Express
```

## Authors
- Steve Carpenter