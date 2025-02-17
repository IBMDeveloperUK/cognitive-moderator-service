# Create a cognitive moderator chatbot for anger detection, natural language understanding and explicit images removal
In this code pattern, we will create a chatbot using IBM functions and Watson services. The chatbot flow will be enhanced by using Visual Recognition and Natural Language Understanding to identify and remove explicit images and or detect anger and ugly messages

When the reader has completed this journey, they will understand how to:

* Create a chatbot that integrates with Slack via IBM Functions
* Use Watson Visual Recognition to detect explicit images (in beta)
* Use Watson Natural Understanding to detect emotions in a conversation
* Identify entities with Watson Natural Language Understanding

![](doc/source/images/architecture.png)

## Flow
1. The user interacts from the Slack app and either sends a text or uploads an image.
2. The text or image that is used in the Slack for conversation is then passed to an IBM function API by a bot. The API is a call to an IBM Function that categorizes the text or images based on the response of Watson Visual Recognition or Watson Natural Language Processing.
3. Watson Visual Recognition categorizes the uploaded image using default and explicit classifier.
4. Watson Natural Language Processing categorizes the text, if text is send as part of slack communication.
5. IBM function then gets the response and if the text is not polite, a message is sent by the bot to the Slack user to be more polite using [Slack post message API](https://api.slack.com/methods/chat.postMessage). If the image used is explicit, the image will be deleted by the IBM function using [Slack files delete API](https://api.slack.com/methods/files.delete).


## Included components

* [IBM Functions](https://cloud.ibm.com/openwhisk): IBM Cloud Functions (based on Apache OpenWhisk) is a Function-as-a-Service (FaaS) platform which executes functions in response to incoming events and costs nothing when not in use.
* [IBM Watson Visual Recognition](https://www.ibm.com/watson/services/visual-recognition/): Quickly and accurately tag, classify and train visual content using machine learning.
* [IBM Watson Natural Language Understanding](https://www.ibm.com/watson/developercloud/natural-language-understanding.html): Analyze text to extract meta-data from content such as concepts, entities, keywords, categories, sentiment, emotion, relations, semantic roles, using natural language understanding.

## Featured technologies

* [Slack Apps](https://api.slack.com/slack-apps) Customize functionality for your own workspace or build a beautiful bot to share with the world.
* [Slack Bots](https://api.slack.com/bot-users) Enable conversations between users and apps in Slack by building bots.

# Watch the Video

[![](https://img.youtube.com/vi/9c7NuamK8JA/0.jpg)](https://youtu.be/9c7NuamK8JA)

# Pre-requisites

To go through this workshop smoothly, you'll need the following:
- an [IBM Cloud account](https://ibm.biz/Bdzfzg)
- [GIT CLI](https://git-scm.com/downloads) installed on your machine
- [IBM Cloud CLI](https://github.com/IBM-Cloud/ibm-cloud-cli-release/releases/) installed on your machine

## Steps

1. [Clone the repo](#1-clone-the-repo)
2. [Create Watson Visual Recognition and Natural Language Understanding service with IBM Cloud](#2-create-watson-visual-recognition-and-natural-language-understanding-service-with-ibm-cloud)
3. [Create Slack App and Bot for a workspace](#3-create-slack-app-and-bot-for-a-workspace)
4. [Deploy the function to IBM Cloud](#4-deploy-the-function-to-ibm-cloud)
5. [Test using slack](#5-test-using-slack)


## 1. Clone the repo

Clone the `cognitive-moderator-service` repository locally. In a terminal (we recommend using PowerShell on Windows), run:

```
$ git clone https://github.com/IBMDeveloperUK/cognitive-moderator-service.git
$ cd cognitive-moderator-service
```

## 2. Create Watson Visual Recognition and Natural Language Understanding service on IBM Cloud

Create the following services, you can leave all the values to default:

* [**Watson Visual Recognition**](https://cloud.ibm.com/catalog/services/visual-recognition)
* [**Watson Natural Language Understanding**](https://cloud.ibm.com/catalog/services/natural-language-understanding)

> Make note of the service credentials - you can access them by clicking `Service Credentials` on the left hand side panel once the service is created - as they will be later used when creating a function.

## 3. Create Slack App and Bot for a workspace

* Create [your own Slack workspace here](https://slack.com/create) 
* Once created, access [Your Apps](https://api.slack.com/apps), click `Create New App`. Fill out the `App Name` and the `Workspace` fields and click `Create App`.

![](doc/source/images/create-new-app.png)

* After the app is created, you will be taken to a page where you can configure the Slack app. Make note of **`Verification Token`**  which will be used later in the function.
![](doc/source/images/app-credentials.png)

* Create bot user from `Bot Users`. Provide a `Display Name` and `Default Username` for the bot and click `Save Changes`.
![](doc/source/images/bot-users.png)

* From `OAuth and Permissions`, save the `OAuth Access Token` for later use by the IBM cloud function.
![](doc/source/images/oauth-access-token.png)

Select `Permissions Scopes` that will be used by the bot that we will be creating next.

![](doc/source/images/permission-scopes.png)

* Once this is done, click Save and `reinstall the app` as prompted at the top of the page. 

## 4. Deploy the function to IBM Cloud
Once the credentials for both IBM Cloud and Slack are noted, we can now deploy the function to IBM Cloud.

Copy `params.sample.json` to `params.json` using `cp params.sample.json params.json` and replace the values with the credentials you have noted in proper place holders.

```
{
    "NLU_APIKEY": "<NLU apikey>",
    "NLU_URL": "<NLU url>",

    "VISUAL_RECOGNITION_IAM_APIKEY": "<Visual Recognition IAM API Key>",

    "SLACK_VERIFICATION_TOKEN": "<Slack Verification Token>",
    "SLACK_ACCESS_TOKEN": "<Slack OAuth Access Token>",

    "SLACK_MESSAGE_POST_URL": "https://slack.com/api/chat.postMessage",
    "SLACK_MESSAGE_DELETE_URL": "https://slack.com/api/files.delete"
}

```

To deploy the function you'll need to install the Cloud Functions plugin, you can do so by running the following command in your terminal:
```
ibmcloud plugin install cloud-functions
```

Once this is done, run the following commands in your terminal:

* Login to IBM Cloud
```
ibmcloud login
```

* Target the right `Org` and `Space`
```
ibmcloud target --cf
```

* Deploy the function to IBM cloud. Make sure you are in the project directory then run:

```
ibmcloud wsk action create WatsonModerator functions/bot_moderator_function.py --param-file params.json --kind python:3.7 --web true

```

* After the function is created, you can run following command to update the function if there are any changes:

```
ibmcloud wsk action update WatsonModerator functions/bot_moderator_function.py --param-file params.json --kind python:3.7
```

Now you can login to IBM Cloud and see the function by going to `IBM functions` in the left side-panel (if it doesn't show you can click the hamberger button at the top-left), click `Actions`. You should see the function in the list of functions in this page.
![](doc/source/images/ibm-cloud-functions.png)


* Expose the function so that it can be accessed using an API

1. Click the `APIs` from the IBM Cloud Functions page and click  `Create Cloud Functions API`.
![](doc/source/images/create-api.png)

2. Provide `API Name`, `Base path for API`
![](doc/source/images/create-api-1.png)

3. Click `Create Operation` and in the dialog box that appears, provide the following like in the figure below and click `create`. Then click `Save & Expose` at the bottom right.
![](doc/source/images/create-api-2.png)

4. From the `API Explorer` select the `API URL` and save it for later use.
![](doc/source/images/create-api-3.png)

* Now, go back to your [Slack administration page](https://api.slack.com/apps/)

* Add `Event Subscriptions` for your app by clicking on to `Add features and functionality` from the main page of the app.
![](doc/source/images/event-subscriptions.png)

Add the `API URL` of the function from IBM Cloud to the `Request URL` in Slack App

![](doc/source/images/request-url.png)

> Make sure the URL is verified. To be verified the API needs to return a response to the request that Slack sends to this `Request URL` with `challenge` parameter.

In the `Event Subscriptions` page, enable it by turning `on` using the `on/off` toggle button and `Add Workspace and Bot Events` in subsequent sections. Add Following events for both:

1. message.channels
2. message.group
3. message.im

![](doc/source/images/events.png)

* Click Save Changes and reinstall the app as prompted at the top of the page.

## 5. Test using slack

* **Test Case 1: Usage of rude messages**

Now you can use slack to test. If rude message are sent which NLU categorized as `anger` or `disgust` you will see a message from the `bot` that you created from Slack.

![](doc/source/images/slack-test-1.png)

* **Test Case 2: Usage of explicit image**

Upload an image that's explicit.

> An explicit image is an image that contains objectionable or adult content that may be unsuitable for general audiences.

When you upload an **explicit** image, the image will be deleted from the Slack conversation.

![](doc/source/images/explicit-photo-upload.png)

![](doc/source/images/explicit-photo-upload-2.png)

# Sample Output

![](video/Bot4Code.gif)

## Troubleshooting

* You can monitor logs and see the output of the each request and/or call to function from the `monitor` page of the function.
![](doc/source/images/monitor-function.png)

* You can change the code at runtime and run the function, by clicking `Code` from left navigation menu of the function page. It will open up an editor with the code which you can change and save. To run click `Invoke`.
![](doc/source/images/code-editor.png)

* You can also change parameters at runtime by clicking `Parameters` from left navigation menu of the function page.
![](doc/source/images/parameters.png)

# Links

* [IBM Cloud Functions](https://cloud.ibm.com/docs/openwhisk/index.html#getting-started-with-cloud-functions) - Getting started with IBM Cloud functions
* [Watson Node.js SDK](https://github.com/watson-developer-cloud/node-sdk): Visit the Node.js library to access IBM Watson services.
* [Sample Node.js application for Language Translator](https://github.com/watson-developer-cloud/language-translator-nodejs): Sample Node.JS application for Watson Language Translator service


# Learn more

* **Artificial Intelligence Code Patterns**: Enjoyed this Code Pattern? Check out our other [AI Code Patterns](https://developer.ibm.com/technologies/artificial-intelligence/).
* **AI and Data Code Pattern Playlist**: Bookmark our [playlist](https://www.youtube.com/playlist?list=PLzUbsvIyrNfknNewObx5N7uGZ5FKH0Fde) with all of our Code Pattern videos
* **With Watson**: Want to take your Watson app to the next level? Looking to utilize Watson Brand assets? [Join the With Watson program](https://www.ibm.com/watson/with-watson/) to leverage exclusive brand, marketing, and tech resources to amplify and accelerate your Watson embedded commercial solution.

# License
This code pattern is licensed under the Apache Software License, Version 2.  Separate third party code objects invoked within this code pattern are licensed by their respective providers pursuant to their own separate licenses. Contributions are subject to the [Developer Certificate of Origin, Version 1.1 (DCO)](https://developercertificate.org/) and the [Apache Software License, Version 2](https://www.apache.org/licenses/LICENSE-2.0.txt).

[Apache Software License (ASL) FAQ](https://www.apache.org/foundation/license-faq.html#WhatDoesItMEAN)
