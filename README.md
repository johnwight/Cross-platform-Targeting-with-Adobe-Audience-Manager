# Cross-Platform Targeting with Adobe Audience Manager
Integrate Adobe Audience Manager with Adobe I/O for Cross-Platform Targeting

These instructions describe how to to implement cross-platform targeting solutions with Adobe Experience Cloud and Adobe I/O products integration. The solution uses below Adobe products:


1. [Introduction](#Introduction)

1. [Set Up Products](#Set-Up-Products)

1. [Watch the Solution Work](#Watch-It-Work)

## <a name="Introduction">Introduction</a>


### Solution Use Case

An anonymous user visits a retail website. The user visits the product page, selects a color and size of the product and adds the product into the cart. The user proceeds to checkout but before completing the transaction, leaves the website, abandoning the cart. The cart abandonment activity is captured in Adobe Audience Manager segment and the user profile is served on multiple platforms to create personalized experiences.


### Solution Architecture

The following diagram shows the architecture for this solution:

![aam architecture](https://user-images.githubusercontent.com/29133525/34681317-c1c44bca-f458-11e7-9dcc-6b5a45e27a9f.png)

### What's Needed

To complete this solution, you will need authorization to use the following services:

* Adobe Launch
* Adobe Analytics/Triggers
* Adobe I/O Events
* Adobe I/O Runtime
* Adobe Audience Manager APIs
* Adobe Audience Manager
* Adobe Target-Experience Targeting


## <a name="Set-Up-Products">Set Up Products</a>

To set up Adobe products for this solution:

1. [Set Up Adobe Launch](#Set-up-Launch)

1. [Set Up Analytics Triggers](#Set-up-Triggers)

1. [Set Up Adobe I/O Runtime](#Set-up-Runtime)

1. [Set Up Adobe I/O Events](#Set-up-Events)

1. [Set Up Adobe Audience Manager](#Set-up-AM)

1. [Set Up Adobe Target](#Set-up-Target)


### <a name="Set-up-Launch">Set Up Adobe Launch</a>

To set up Launch:

1. On www.launch.adobe.com, click **New Property**.

   ![create new property](https://user-images.githubusercontent.com/29133525/34681760-13437952-f45a-11e7-898c-1561265c27b7.png)

1. On the **Create Property** box, provide the details for the new property and click **Save.**

   ![create property box details](https://user-images.githubusercontent.com/29133525/34681834-4af921e4-f45a-11e7-9f8f-3daca980310c.png)

1. Click the **Extensions** tab and install the following extensions

   * Target
   * Analytics
   * ContextHub
   * Core
   * Experience Cloud ID Service
   
   ![extensions](https://user-images.githubusercontent.com/29133525/34681903-83087260-f45a-11e7-9f10-a9b39123e28b.png)

1. Click the **Rules** tab and create a **Target** rule with the following specifications:

   ![edit rule](https://user-images.githubusercontent.com/29133525/34681953-a1b34b40-f45a-11e7-920a-352ac58f3696.png)

1. Similarly, create an **Analytics** rule with the following specifications:

   ![analytics rule](https://user-images.githubusercontent.com/29133525/34682376-038da364-f45c-11e7-959b-413760b2c451.png)

1. For the **Adobe Analytics - Set Variables** action, click the **</> Open Editor** button at the bottom of the page and add the following custom script:

```
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
```


7. Click the **Environments** tab and create the following environments:

   * Dev
   * Stage
   * Production

      ![create environments](https://user-images.githubusercontent.com/29133525/34682566-7aee09b2-f45c-11e7-937a-0c5aadb0aef8.png)

8. Save the rule and then click the **Publishing** tab.

9. Click **Add New Library** button.

      ![add new library](https://user-images.githubusercontent.com/29133525/34682601-9a4a5860-f45c-11e7-9c40-6c7d89b20b61.png)
   
10. On the **Create New Library** form, specify a **Name** for the build and then select **Dev (development)** from the **Environment** drop down.

11. Click the **Add All Changed Resources** button. 
   
      ![specify build](https://user-images.githubusercontent.com/29133525/34682632-bb011508-f45c-11e7-98ef-7c7a55ebe898.png)
   
12. Under **Development**, select **Build for Development** in the library drop down.
   
      ![build for dev](https://user-images.githubusercontent.com/29133525/34682676-d8ac25c0-f45c-11e7-9987-2f79149b56fa.png)

13. Approve and publish the library by selecting the appropriate option under the drop down arrow for each phase of the build workflow (**Submitted** and **Approved**).

      ![full flow](https://user-images.githubusercontent.com/29133525/34682729-038d0f8e-f45d-11e7-927b-b730a66a802d.png)

14. Repeat this process for the **Stage** and **Production** environments as well.

15. The last step in the workflow is to select **Build and Publish to Production** on the drop down arrow under **Approved**.

      ![build and publish to production](https://user-images.githubusercontent.com/29133525/34682768-1f317504-f45d-11e7-819c-d29787ed322d.png)
   
### <a name="Set-Up-Analytics-Triggers">Set Up Analytics Triggers</a>

Triggers is a Marketing Cloud Activation core service that enables marketers to identify, define, and monitor key consumer behaviors, and then generate cross-solution communication to re-engage visitors. You can use triggers in real-time decisions and personalization.

For instructions on setting up Analytics Triggers, see [Specifying a New Trigger](https://github.com/adobeio/analytics-triggers-documentation#Specify-a-New-Trigger).

To set up Triggers for this solution:

1. Create a cart abandonment Triggers rule.

     ![cart abandonment rule](https://user-images.githubusercontent.com/29133525/34683199-87de61b0-f45e-11e7-976c-85cb99484ffa.png)

1. Add the following dimensions:

   * Custom eVar 3
   * Custom eVar 4
   * Custom eVar 5
   * Custom eVar 6
   * Custom eVar 7
   * Custom eVar 8

1. To enable these dimensions in your report suite:

   On Analytics **Admin** screen, click **Report Suites**.

   1. Select a report suite and then select **Edit Settings** > **Conversion** > **Conversion Variables**.
   
      ![conversion variables](https://user-images.githubusercontent.com/29133525/34684000-f458eb2e-f460-11e7-9660-b7422735772c.png)


   1. Click **Add New** and then select the **Status** checkbox. Select **Enabled** from the drop-down menu and then click **Save**.

      ![status enabled](https://user-images.githubusercontent.com/29133525/34684251-bbfa042e-f461-11e7-9d19-8828414e9f8d.png)


   1. Add as many conversion variables as you need. It may take some time for them to appear on the Trigger dimensions screen.

 

### <a name="Set-up-Runtime">Set Up Adobe I/O Runtime</a>

In this solution, you can deploy the following script to handle Triggers I/O Events and to provide updates to the visitor profile in **Customer Attributes** via Target Profile APIs.

**webhook.js**
```
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
```

To deploy the webhook.js and create a web action, enter the following commands:

```
wsk action create aam webhook.js
```
 
```
wsk action update aam --web raw
```


After entering the commands, the web action is accessible at the following location:

```
https://runtime-preview.adobe.io/api/v1/web/<your_openwhisk_namespace>/default/aam
```


### <a name="Set-up-Events">Set Up Adobe I/O Events</a>

To set up Adobe I/O Events:

1. After signing in to the [Adobe I/O Console](https://adobe.io/console), click **New Integration**.

    ![new integration button](https://user-images.githubusercontent.com/29133525/30292388-2ccdd986-96f3-11e7-93bd-93f74bb4e3a4.png)

2. Select **Receive real-time events** and click **Continue**.

    ![real time events](https://user-images.githubusercontent.com/29133525/30292486-946dc812-96f3-11e7-9bc5-2aa0196f704b.png)


3. Select **Analytics Triggers** as an event provider and click **Continue**.

    ![create io trigger](https://user-images.githubusercontent.com/29133525/30292595-064305ec-96f4-11e7-9e60-3ee811c7949a.png)

4. Click **Continue** to move on to the next page without making any changes.

5. Provide the **Name** and **Description** for your integration.

    ![name integration box](https://user-images.githubusercontent.com/29133525/30292714-6d00a8ac-96f4-11e7-8449-f93bb2fccfcb.png)

6. Generate a public certificate. To do this:

    1. Open a terminal and execute the following command:

        ```
        openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout private.key -out certificate_pub.crt
        ```
    2. Upload the public certificate by clicking the **Select a File** link and then by selecting the certificate from your computer:

        ![public certificate](https://user-images.githubusercontent.com/29133525/32476036-2e563e40-c332-11e7-9fbd-889f1ba8054f.png)

7. On the **Webhook Details** form, add details, including your web action URL for I/O Events integration webhook. This is where Triggers messages will be delivered by I/O Events. Click **Save**.

   ![webhook details](https://user-images.githubusercontent.com/29133525/34685236-efce0cca-f464-11e7-902a-27e594a82c24.png)

    For more information on creating and registering webhooks, see [Introduction to Webhooks](https://github.com/adobeio/adobeio-events-documentation/blob/master/Webhook_docs_intro.md).


### <a name="Set-up-AM">Set Up Adobe Audience Manager</a>

Adobe Audience Manager is a data management platform (DMP) that helps you build unique audience profiles so you can identify your most valuable segments and use them across any digital channel.

To set up Audience Manager:

1. In Audience Manager, click **Manage Data**.

   ![manage data](https://user-images.githubusercontent.com/29133525/34685969-4a38be56-f467-11e7-825b-ea138ea1b0ee.png)


1. Click **Traits**.

   ![traits](https://user-images.githubusercontent.com/29133525/34686031-8b31008a-f467-11e7-819e-587f67d9a769.png)



1. Create a new **Trait** with the **Expression Builder**, as shown:

   ![expression builder](https://user-images.githubusercontent.com/29133525/34685986-670e4190-f467-11e7-83be-223c0f8a9858.png)

   ![expression built](https://user-images.githubusercontent.com/29133525/34686086-bf416ed2-f467-11e7-992c-3c43288473d2.png)


1. Click **Segments** and create a new **Segment** as shown below. Make sure you choose the correct report suite. Select the trait that you created in the previous step.

   ![trigger segment](https://user-images.githubusercontent.com/29133525/34686347-7c194fca-f468-11e7-96ff-b0ca4d314e97.png)


1. Click **Destinations** and create a new destination as shown below.

   ![destinations](https://user-images.githubusercontent.com/29133525/34686733-bb9907b6-f469-11e7-8818-0412b4627bff.png)

1. Choose the **Browser** platform. 

1. Add the segment created in the previous step in segment mapping.

1. Save the changes.


### <a name="Set-up-Target">Set Up Adobe Target</a>

To set up Target:

1. Click **Launch** on the Target product card.

   ![target card launch](https://user-images.githubusercontent.com/29133525/34688275-d21fade6-f46e-11e7-8aa1-cd1d8fd080ce.png)
    


1. On the **Activities** tab, click **Create Activity** and select **Experience Targeting**.

   ![create activity](https://user-images.githubusercontent.com/29133525/34688008-e76a2830-f46d-11e7-91c7-4a34c5bed95b.png)


1. On the **Create Experience Targeting Activity** form, provide the target page URL.

   ![targeting activity](https://user-images.githubusercontent.com/29133525/34688397-3a28006e-f46f-11e7-8299-e99b4b8cacbc.png)


1. On the **Create Experience Targeting Activity** form, provide the Target page URL.

    ![targeting activity](https://git.corp.adobe.com/storage/user/17975/files/51e83034-cbc5-11e7-9966-db80874656a6)

1. Open the text editor to add a custom UI element for the user on the site. To do this, select an empty container and click **Edit Text/HTML**.

    ![container](https://git.corp.adobe.com/storage/user/17975/files/72178278-cbc6-11e7-8d11-b68437a3ae57)

1. On the text editor, click the **HTML** button, paste the following HTML and click **Save** and then click **Next**.


   ![best coat and jackets for you](https://user-images.githubusercontent.com/29133525/34688762-7089b494-f470-11e7-86d7-d10dab3a371f.png)

10. Click **Next** and then click **Change Audience**. Select the audience created in Step 4 (Trigger Audience).

    ![change audience](https://git.corp.adobe.com/storage/user/17975/files/f488f2d0-cbc8-11e7-9328-56ffcab6ed98)


11. On the **Activity Settings**, specify your **Objective**, **Priority**, **Duration**, and **Primary Goal**.

    ![activity settings](https://git.corp.adobe.com/storage/user/17975/files/96bc6c8a-cbc9-11e7-976b-c3cc08e80846)

12. Click **Save & Close**.

 

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




