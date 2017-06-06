# 1. Introduction

**NOTE: This integration is currently in beta stage, usage is at your own risk. The REST API is currently not secured so ensure you run this application sandboxed (behind a firewall).**

The Elvis image recognition server is a nodejs server application that integrates Elvis with AI image recognition services from Google, Amazon and Clarifai. It uses these services to detect labels, landmarks and emotions. Labels can also be automatically translated to other languages (Google Translate). The recognition server supports tagging of images right after they are imported in Elvis but it also features a built-in REST API to detect images that are already imported in Elvis.

This readme describes how to setup the integration. Please read this [blog article](https://www.woodwing.com/blog/post/157564395070/ai-dam-five-ways-ai-can-make-life-easier-for) if you want to know more about Elvis and AI.

# 2. Prerequisites

- Fully installed and licensed Elvis server (5.24 or higher). You can obtain Elvis via: https://www.woodwing.com/en/digital-asset-management-system
- Server where the image recognition server can run (can be on the same machine where Elvis runs)
- API user license 

# 3. Installation steps

## 3.1 Configure Elvis metadata fields

Depending on your configuration and used services, you may need to add custom metadata fields to your Elvis installation. These custom metadata fields need to be configured in the `<Elvis Config>/custom-assetinfo.xml` file. Sample configuration files are provided in the `elvis-config` folder.

## 3.2 Optional: configure the Elvis Webhook

An Elvis webhook needs to be configured if you want to detect incoming images in Elvis. You can skip this step if you only want to use the REST API.

- Log-in to the Elvis web client as admin user
- Go to the management console, webhooks secion and add a new webhook
- Name: For example, "Image Recognition"
- URL: Point it to the URL where the image recognition server is running, if it's running on the same machine as Elvis, this will be: http://localhost:9090/
- Event types: `asset_create`
- Metadata to include `assetDomain`
- Save the webhook
- The generated secret token needs to be specified in the image recognition configuration later on.

Detailed information on setting up and using webhooks can be found on [Help Center](https://helpcenter.woodwing.com/hc/en-us/articles/115001884346).

## 3.3 Install the image recognition server

- Clone or download this package.
- Open src/config.ts and configure the settings (Port where this server runs, Elvis Server settings, which Image AI services to use, etc). You can either configure the settings in this config file or by setting environment variables. Note: the configured Elvis user needs to be an API user that can process all incoming webhook events.
- Install nodejs (6.9 or higher) from https://nodejs.org
- Open a terminal and go to the package folder.
- Install TypeScript via npm: `npm install -g typescript`
- Install node modules: `npm install`
- Start the server: `npm start`

# 4. Detect images during import

- Open the Elvis web client.
- Upload a few images.
- Enable the relevant metadata fields in the metadata panel.
- Watch the magic :)
- When there's no magic... check the terminal where the image recognition server is running for errors and make sure your configuration is correct.

# 5. Detect images using the REST API

## 5.1 POST /api/recognize

Starts the image recognition for a given query, immediatly returns a process id that can be used to track progress or cancel the operation.

In this example we detect all images in the `/Demo Zone` folder.

Request
```bash
$ curl -H "Content-Type: application/json" -X POST -d '{"q":"ancestorPaths:\"/Demo Zone\""}' http://localhost:9090/api/recognize
```

Response (202 ACCEPTED)
```json
{
  "processId": "5e5949d8-3c58-4074-84a4-a63fa10286f8"
}
```

## 5.2 GET /api/recognize/:id:

Retrieve progress information for a given recognition process.

Request
```bash
$ curl -X GET http://localhost:9090/api/recognize/5e5949d8-3c58-4074-84a4-a63fa10286f8
```

Response (200 OK)
```json
{
  "cancelled": false,
  "failedCount": 0,
  "successCount": 335,
  "totalCount": 3246,
  "id": "5e5949d8-3c58-4074-84a4-a63fa10286f8",
  "request": {
    "q": "(ancestorPaths:\"/Demo Zone\") AND assetDomain:image"
  }
}
```

## 5.3 DELETE /api/recognize/:id:

Cancel a recognition process.

Request
```bash
$ curl -X DELETE http://localhost:9090/api/recognize/5e5949d8-3c58-4074-84a4-a63fa10286f8
```

Response (200 OK)
```
Process with id "5e5949d8-3c58-4074-84a4-a63fa10286f8" is being cancelled.
```

# 6. Version history

## v2.0.0
- Added support for translating tags into different languages (using Google Translate)
- New recognize REST API to detect existing assets in Elvis
- Refactored package structure

## v1.1.0
- Elvis 6 support
- Support for API users (requires Elvis 5.24 or higher)
- Improved logging: log lines with date and time

## v1.0.0
- Clarifai: label detection
- Google Vision: label & landmark detection
- AWS Rekognition: label & emotion detection
