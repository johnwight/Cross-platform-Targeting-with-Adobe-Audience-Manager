# Cross-platform-targeting-with-Adobe-Audience-Manager
Integrate Adobe Audience Manager with Adobe I/O for Cross-Platform Targeting







Skip to end of metadata
Created by Hiren Shah, last modified on Dec 04, 2017 Go to start of metadata
Table of Contents
Introduction
Step 1: Adobe Launch
Step 2: Analytics Triggers
Dimensions
Step 3: Adobe I/O Runtime
Step 4: Adobe I/O Events
Step 5: Adobe Audience Manager
Step 6: Adobe Target
Step 7: Let's Test 
 

Introduction
This blog is a step by step guide to implement the cross platform targeting solution using Adobe Experience Cloud and Adobe I/O products integration. The solution uses below Adobe products:

Adobe Launch (The next generation dynamic tag management platform)
Adobe Analytics/Triggers
Adobe I/O Events
Adobe I/O Runtime
Adobe Audience Manager APIs
Adobe audience Manager
Adobe Target-Experience Targeting


User Story:

"An anonymous user visits a retail website. The user is interested in buying a product. He/she visits the product page, selects a color and size of the product and adds the product into the cart. The user proceeds for checkout but before completing the transaction, user leaves the website abandoning the cart. The cart abandonment activity is captured in Adobe Audience Manager segment and the user profile is served on multiple platform to create personalized experiences."



Architecture:





Step 1: Adobe Launch
Adobe Launch is the next generation of Dynamic Tag Management. It provides a platform-based approach to building DTM extensions and a streamlined distribution system to quickly deploy client-side DTM libraries. Custom resources can now be created and reused within DTM to simplify the distribution of client side web applications.

Go to launch.adobe.com
Create new "Property"



Enter "Property" details



Go to Extensions→Catalog and install below extensions



Go to Rules and create a "Target" rule defined as below.



Create an Analytics rule defined as below.



In the "Adobe Analytics-Set Variables" action, go to the bottom of the page and add custom script in </> Open Editor as below:


1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
var cart=ContextHub.getItem("cart");
var profile=ContextHub.getItem("profile");
 
var price=[];
var items=[];
var links=[];
var thumbs=[];
var origin=window.location.origin;
for(i=0;i<cart.entries.length;i++)
{
     price[i]=cart.entries[i].price;
     items[i]=cart.entries[i].title;
     links[i]=origin+cart.entries[i].page;
     thumbs[i]=origin+cart.entries[i].thumbnail;
}
 
s.eVar3=document.cookie.split("PC#")[1].split("#")[0];
s.eVar4=price.join("|");
s.eVar5=items.join("|");
s.eVar6=links.join("|");
s.eVar7=thumbs.join("|");
s.eVar8=_satellite.getVisitorId().getAudienceManagerLocationHint();



Go to "Environments" and create Dev, Stage and Production environment.




Save Rule and go to "Publishing", Click on "Add New Library".




Give name for the build, select Dev (Development) environment and then click "Add all changed resources".




Build for development and staging and approve for production.









Step 2: Analytics Triggers
Triggers is a Marketing Cloud Activation core service that enable marketers to identify, define, and monitor key consumer behaviors, and then generate cross-solution communication to re-engage visitors. You can use triggers in real-time decisions and personalization.

New to Triggers? Before we proceed, We recommend you to read: Step by step guide for Analytics Triggers#Step5:Analytics/TriggersSetup

Create a simple cart abandonment Trigger rule as shown below.



Add below dimensions with Trigger.
Dimensions
 
Custom eVar 3
Custom eVar 4
Custom eVar 5
Custom eVar 6
Custom eVar 7
Custom eVar 8
Page URL
Note: To enables these dimensions in your report suite go to Adobe Analytics->Admin→Report Suites and follow below steps.

i) Select a report suite then click on Edit Settings->Conversion→Conversion Variables



ii) Click on Add New→check the status check box and select "Enabled" from the drop down menu. Click Save.



iii) Add as many conversion variables you need, it may take some time to show up in the Trigger dimensions UI.

 

Step 3: Adobe I/O Runtime
The Adobe I/O Runtime is a serverless platform that allows you to quickly deploy custom code to respond to events, execute functions right in the cloud, all with no server set-up.

For our demo we will be deploying a script for handling Triggers I/O Events and updating the visitor profile in Customer Attributes via Target Profile APIs.


webhook.js
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
var request = require('request');
 
function main(args) {
 
    var method = args.__ow_method;
 
    if (method == "get") {
        var res = args.__ow_query.split("challenge=");
        var challenge = res[1];
        if (challenge)
            console.log("got challenge: " + challenge);
        else
            console.log("no challenge");
 
        return {body: challenge}
    }
 
    if (method == "post") {
        try {
            var body = new Buffer(args.__ow_body, 'base64');
            var jSon = JSON.parse(body);
            var mcId = jSon.trigger.mcId;
            var index = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar5.data.length - 1;
            var pcId = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar3.data[index];
            var trPrices = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar4.data[index];
            var trProducts = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar5.data[index];
            var trLink = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar6.data[index];
            var trThumb = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar7.data[index];
            var dcs_region = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar8.data[index];
 
 
            var url = 'http://adobeiosolutionsdemo.demdex.net/event?trProducts=' + trProducts + '&trPrices=' + trPrices + '&d_mid=' + mcId + '&d_orgid=<your_org_id>@AdobeOrg&d_rtbd=json&d_jsonv=1&dcs_region=' + dcs_region;
            console.log("Calling AAM API with:" + url);
            return new Promise(function(resolve, reject) {
                request.get(url, function(error, response, body) {
                    if (error) {
                        reject(error);
                    } else {
                        resolve({body: response});
                    }
                });
            });
        } catch (e) {
            console.log("Error occured while calling the API", e);
        }
 
    }
 
}



Execute below commands to deploy the webhook.js and create a web action.


wsk action create aam webhook.js
 
wsk action update aam --web raw



After successful execution of the above commands, the web action is accessible in WWW at below location.


https://runtime-preview.adobe.io/api/v1/web/<your_openwhisk_namespace>/default/aam



Step 4: Adobe I/O Events
With Adobe I/O Events, you can code event-driven experiences, applications, and custom workflows that leverage and combine the Adobe Experience Cloud, Creative Cloud, and Document Cloud.

New to I/O Events? Before we proceed, We recommend you to read: Step by step guide for Analytics Triggers#Step6:AdobeI/OConsoleIntegration

Provide your web action URL for I/O Events integration webhook, this is where Trigger messages will be delivered by I/O Events.

 


Step 5: Adobe Audience Manager
It’s a data management platform (DMP) that helps you build unique audience profiles so you can identify your most valuable segments and use them across any digital channel.

Go to Adobe Audience Manager.

Click on "Manage Data" on the left panel.



Click on Traits.



Create a new "Trait" with below expression.




Click on "Segments" and create a new "Segment" as below. Make sure you choose the correct report suite. Select the trait created in the above step.



Click on "Destinations" and create a new "destination" as below. Choose "Browser" platform and add the segment created in the above step in segment mapping.



Save the changes.


Step 6: Adobe Target
Adobe Target is a personalization solution that makes it easy to identify your best content through tests that are easy to execute. So you can deliver the right experience to the right customer.

Go to "Target" and click on Launch.



Go to Activity and click on "Create Activity" and select "Experience Targeting".



Provide the target page URL.



Select an empty container and click on "Edit Text/HTML", we will be using this space to add custom UI elements for the user.



Click on HTML button and paste below HTML code. Save and click Next.




 

Click on "Change Audience".



Select the "Trigger-segment" audience by aam-integration-user, this is the same segment that we created in Adobe Audience Manager.
Note: If this segment doesn't appear in your target then please make sure that "Shared Audiences" is enabled for your organization. Please visit https://adobe.allegiancetech.com/cgi-bin/qwebcorporate.dll?idx=X8SVES



Update the activity settings as per your goals.



Click Save & Close.
 

Step 7: Let's Test 
Execute below steps to test and debug the solution.

Go to http://localhost:4502/content/we-retail/us/en/products/men.html
Click on any product you like.



Select a Color and Size and click on "ADD TO CART".



Click on Checkout.



Enter details and click continue.



Close the tab to initiate the cart abandonment scenario.

Go to Triggers UI page, wait for your trigger event to surface.



Go to Command Line Interface (CLI) and execute below commands.

wsk activation list runtime



Select the top most activation and execute below command.

wsk activation get f6f5ae1dcb3d4292991d63f22283fb94





Go to http://localhost:4502/content/we-retail/us/en/products/men.html again and you should see custom elements on the page.




