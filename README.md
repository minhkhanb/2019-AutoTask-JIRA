# Requirements:
- Your favorite IDE or text editor.
- A basic knowledge of JavaScript and web development.
- A basic understanding of Jira.
- Node.js (v4.5.0 or later). You’ll also need npm, which is included with Node.js.
- A tool to tunnel your local development environment to the internet. We recommend ngrok, which is free and easy to use.

# Set up your development environment
You’ll need the following things to develop for Jira Cloud:

- A developer instance of Atlassian Cloud for testing and validating apps.
- A local development environment for creating apps.

## Step 1. Get an Atlassian Cloud instance
Let’s start by getting a free developer instance of Atlassian Cloud that you can use to test your app. The Atlassian Cloud instance has Confluence and all of the Jira applications installed, but be aware that there are limits on the number of users you can create.

Go to http://go.atlassian.com/cloud-dev and register your instance.
Once your instance is ready, sign in, and complete the setup wizard.
If needed, use the app switcher to navigate to Jira. Select the app switcher icon (1), and then choose Jira (2).

## Step 2. Enable development mode
Now we’ll enable development mode for your Atlassian Cloud instance. Development mode gives you the ability to install apps that are not from the Atlassian Marketplace.

Navigate to Jira settings (cog icon) > Apps > Manage apps.
Scroll to the bottom of the Manage apps page, and click Settings (1).
Select Enable development mode (2), and click Apply.

## Step 3. Set up your local development environment
If you install an Atlassian Connect app in an Atlassian Cloud instance, the app is usually hosted elsewhere (for example, a cloud platform service like Heroku). However, when you are building an app it’s easiest to develop it on your local machine and make it available over the internet using tunneling (via HTTPS). This allows you to work locally, but test against your Atlassian Cloud instance. 

On your command line, run the following:
```
npm install -g ngrok 
```


Verify that ngrok is installed correctly by running the following command:
```
ngrok help
```

We’ll show you how to use ngrok to make your app available to the internet later in this tutorial.

# Build a basic app
Now let’s build a simple Atlassian Connect app. This part of the tutorial gives you a hands-on introduction to Atlassian Connect and validates that your development environment is set up correctly.

## Step 1. Define the app descriptor
The fundamental building block of an app is the app descriptor file, usually named atlassian-connect.json. The descriptor file describes your app to the Atlassian application (in this case, Jira Cloud), including the key, name, permissions needed to operate, and modules it uses for integration.

Let’s create a project directory and define the atlassian-connect.json file for your app. The app defined in the sample code below uses a generalPages module, and adds a link titled Greeting to the Jira sidebar. 

Create a project directory for your app’s source files.
In the project directory, create a new file named atlassian-connect.json with the following contents:
```
{
 "name": "Hello World",
 "description": "Atlassian Connect app",
 "key": "com.example.myapp",
 "baseUrl": "https://<placeholder-url>",
 "vendor": {
     "name": "Example, Inc.",
     "url": "http://example.com"
 },
 "authentication": {
     "type": "none"
 },
 "apiVersion": 1,
 "modules": {
     "generalPages": [
         {
             "url": "/helloworld.html",
             "key": "hello-world",
             "location": "system.top.navigation.bar",
             "name": {
                 "value": "Greeting"
             }
         }
     ]
 }
}
```
{{% note %}}Note, you don’t need to change the placeholder used for baseUrl for now. You’ll update it later in this tutorial when you’re ready to deploy your app.{{% /note %}}

Save the descriptor file.

(optional) Validate your descriptor using [Atlassian Connect validator](https://atlassian-connect-validator.herokuapp.com/validate). This handy tool shows you any errors in your app descriptor, such as missing properties or syntax errors.

## Step 2. Create the web application
Now that you’ve created the app descriptor, let’s create the web application using a simple static HTML page. This is the most basic type of Atlassian Connect app. It consists only of an app descriptor and an HTML page. This is not a typical app, but once you understand how it works, it only takes a few more steps to turn the web application into a fully-functional app.

In your project directory, create a new file named helloworld.html with the following contents:
```
<!DOCTYPE html>
<html lang="en">
    <head>
        <link rel="stylesheet" href="https://unpkg.com/@atlaskit/css-reset@2.0.0/dist/bundle.css" media="all">
        <script src="https://connect-cdn.atl-paas.net/all.js" async></script>
    </head>
    <body>
        <section id="content" class="ac-content">
            <h1>Hello World</h1>
        </section>
    </body>
</html>
```
Note that the app descriptor file, atlassian-connect.json, references this file in the generalPages element: url: /helloworld.html

Save the file.

That’s all the coding you need to do. Let’s have a look at the contents of the helloworld.html file in more detail:

Styling: The helloworld.html page uses CSS from Atlaskit. Atlaskit is a library of reusable front-end UI components. This app uses the CSS styling to render the h1 element using the Atlassian font stack.
script tag:  We use the script tag to include the all.js file, which is the client library for the Atlassian Connect JavaScript API. It simplifies client interactions with the Atlassian application, such as making an XMLHttpRequest.
ac-content class: This class wraps the contents of your app and dynamically resizes the iframe in Jira. This keeps your app content visible without scrollbars.

# Deploy and install your app
Let’s deploy your app and install it in your Atlassian Cloud instance. We’ll use ngrok to make the local app available to the internet, and then tell your Jira instance how to find the descriptor.

## Step 1: Host the app on a local web server
You’ll need a simple web server to serve the current directory containing your atlassian-connect.json and helloworld.html  files. There are number of tools you can use to do this, but in this tutorial we’ll be using  http-server (available via npm).

Open your command line, and change to your project directory.
Install http-server by running the following command:
```
npm install http-server -g
```

Start the server on port 8000 by running the following command:
```
http-server -p 8000
```

A message on your command line indicates that the server is serving HTTP at the current address and port. It will look something like this:

Starting up http-server, serving ./ on: http://0.0.0.0:8000/


Confirm that the files you created in steps 1 and 2 are being served by visiting the following URLs:
http://localhost:8000/atlassian-connect.json
http://localhost:8000/helloworld.html

## Step 2: Make the app files available to the internet
Now that your app is hosted on a local web server, let’s use ngrok to make it available over the internet.

In a new command line window, run the following command to expose your web server to the internet. If your app is not running on port 8000, then change the command to use your app’s port number.
```
ngrok http 8000
```

You’ll see a status page on your command line that shows the public URL of your tunnel and other information about connections made over your tunnel. If your app is not running when you try to start ngrok, you’ll see a “Failed to complete tunnel connection” message.

Get the HTTPS URL from the ngrok status page (1).

Edit the app descriptor file, and set the baseUrl property to the ngrok HTTPS URL (from the previous step). For example:
```
"baseUrl": "https://4176ee25.ngrok.io"
```

Confirm that the descriptor is available by going to the ngrok HTTPS URL in your browser. For the URL shown in the image above this would be: https://4176ee25.ngrok.io/atlassian-connect.json. This is the URL you will use to install your app in the next section.

## Step 3: Install and test your app
We’re nearly there! The final step in deploying your app is to install it in your Atlassian Cloud instance. You’ll do this by adding a link to your app’s descriptor file from your Atlassian Cloud instance. This allows Jira to install your app.

Navigate to Jira in your Atlassian Cloud instance, then choose Jira settings (cog icon) > Apps > Manage apps.
Click Upload app.
In the From this URL field, provide a link to your app descriptor. This URL is the same as the hosted location of your atlassian-connect.json descriptor file. Our example uses the following URL:
```
https://4176ee25.ngrok.io/atlassian-connect.json
```

Click Upload. Jira displays the *Installed and ready to go* message when the installation is complete.

Click Close.

Verify that your app appears in the User installed apps list. For example, if you used Hello World for your app name, then Hello World will appear in the list.

Navigate to the Jira home page. A new item labeled Greeting appears in the sidebar.

Click Greeting (1). The Hello World message displays (2).