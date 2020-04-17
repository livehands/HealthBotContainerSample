# Get User Location by IP
This repo is a fork of the Microsoft Health Bot Container Sample.  It uses 3rd party APIs to go a Ge-location reverse lookup of the user's location via thier IP Address.  The thought behind this is to be able to drive the user to a testing site and/or hospital/care center to get help with their COVID-19 Symptoms.  

## User Location Acquisition 
In order for the Location Services in this repo to work we have to acquire the user's location from the Health Bot Container.  The [Health Bot Container Sample](https://github.com/livehands/HealthBotContainerSample) has code that leverages the geolocation API, a HTML5 feature, that allows a web page’s visitor to share their location with you **if they so choose**. The key statement being **if they so choose** hence, the issue is the user has to give us permission to their location. If they don't give permission then the chat bot cannot acquire the user's location and the calls to the geolocation API services will not provide what we need. Secondly, if the user is using an older browser, like say Opera or IE 10, the geolocation API is not available and you won't be able to acquire the location in this situation either.  

## IP Location Services
The solution that I provide here is the "LocationViaIP" endpoint in the function.  My solution is using [GeoLocation DB](http://geolocation-db.com/) API because it did not require me to sign up and is open. 

In my research I found 3 IP Geolocation services that seemed to be the most popular.  You can certainly use your own for this purpose just be aware that you might have to update you client side code to read the values as some APIs return the location as "lat"/"long" and others use "latitude"/"longitude".  I will point this out below in the section on modifying your bot container.

Here are the three APIs I found.  To get a good understanding of how to use this services, read their documentation and modify the LocationViaIP function accordingly.:

1. [GeoLocation DB](http://geolocation-db.com/) Open & seems FREE as no license information is available on the site.  No sign up required.
1. [IP Geolocation API](http://ip-api.com) - Open and Free of non-commercial use. No sign up required. If you exceed the usage limit of 45 requests per minute your access to the API will be temporarily blocked. Repeatedly exceeding the limit will result in your IP address being banned for up to 1 hour. [See the Terms](https://ip-api.com/docs/legal)
1. [IPinfo](https://ipinfo.io) - Account required Free usage of the API is limited to 50,000 API requests per month.

## Difference between My Fork & the OFFICIAL Repo
I added/modified the "getUserLocation(callback)" & "getUserLocationIp(callback)" functions to the my forked repo to enable the capabilities in this solution.  Here is the code snippet that changed.

``` javascript
function getUserLocation(callback) {
    navigator.geolocation.getCurrentPosition(
        function(position) {
            var latitude  = position.coords.latitude;
            var longitude = position.coords.longitude;
            var location = {
                lat: latitude,
                long: longitude
            }
            callback(location);
        },
        function(error) {
            // user declined to share location
            console.log("location error:" + error.message);

            // Get Location via IP
            getUserLocationIp(callback);

            //callback();
        });
}

function getUserLocationIp(callback) {
    const oReq = new XMLHttpRequest();

    oReq.open("GET", ipLocationLookupEndpoint, true);

    oReq.onload = function () {
        var position = JSON.parse(this.response);
        console.log("IP Position: " + JSON.stringify(position));
        if (oReq.status >= 200 && oReq.status <= 400) {
            var latitude = position.lat;
            var longitude = position.lon;
            var location = { lat: latitude, long: longitude };
            callback(location);
        }
        else {
            callback();
        }
    }

    oReq.send();
}

```


# Setup 
## Original Health Bot Container Instructions

**Note:** In order to use this Web Chat with the Health Bot service, you will need to obtain your Web Chat secret by going to ["Integration/Secrets"](./secrets.png) on the navigation panel.
Please refer to [Microsoft Health Bot](https://www.microsoft.com/en-us/research/project/health-bot/) for a private preview and details.

A simple web page to hand off users to the Microsoft Health bot

1.Deploy the website:

[![Deploy to Azure](https://azuredeploy.net/deploybutton.png)](https://azuredeploy.net/)

Note: It is recommended you use the default Linux host type when deploying the container.
However, if you wish to enable online file editing using the App Service Editor, select 'Windows'.

2.Set the following environment variables:

`APP_SECRET`

`WEBCHAT_SECRET`

3.Configure scenario invocation (optional):

The Healthcare Bot service uses [language models](https://docs.microsoft.com/HealthBot/language_model_howto) to interpret end user utterances and trigger the relevant scenario logic in response.

Alternatively, you can programmaticaly invoke a scenario before the end user provides any input.

To implement this behavior, uncomment the following code from the `function initBotConversation()` in the `/public/index.js` file:
```javascript
triggeredScenario: {
    trigger: "{scenario_id}",
    args: {
        myVar1: "{custom_arg_1}",
        myVar2: "{custom_arg_2}"
    }
}
```
Replace {scenario_id} with the scenario ID of the scenario you would like to invoke.
You can also pass different values through the "args" object. 

You can read more about programmatic client side scenario invocation [here](https://docs.microsoft.com/HealthBot/integrations/programmatic_invocation)


4.Set the Bot service direct line channel endpoint (optional)

In some cases it is required to set the endpoint URI so that it points to a specific geography. The geographies supported by the bot service each have a unique direct line endpoint URI:

- `directline.botframework.com` routes your client to the nearest datacenter. This is the best option if you do not know where your client is located.
- `asia.directline.botframework.com` routes only to Direct Line servers in Eastern Asia.
- `europe.directline.botframework.com` routes only to Direct Line servers in Europe.
- `northamerica.directline.botframework.com` routes only to Direct Line servers in North America.

Pass your preferred geographic endpoint URI by setting the environment variable: `DIRECTLINE_ENDPOINT_URI` in your deployment. If no variable is found it will default to `directline.botframework.com`

**Note:** If you are deploying the code sample using the "Deploy to Azure" option, you should add the above secrets to the application settings for your App Service.

## Agent webchat
If the agent webchat sample is also required, [switch to the live agent handoff branch](https://github.com/Microsoft/HealthBotContainerSample/tree/live_agent_handoff)
