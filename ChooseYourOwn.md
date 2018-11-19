## Lab 15

# Overview
This lab will use S3, lambda, Amazon Translate and Amazon SES.  The user will be able to set up a serverless S3 webpage
that takes an input, translates it to another language of the user's choice, and then emails the message to an email address.

## SET Question #1 - AWS Technologies: S3, Lambda, Amazon Translate, Amazon SES
Skim through each of these links and write a summary about *ONE* of these technolgies:
* S3: https://aws.amazon.com/s3/
* Lambda: https://aws.amazon.com/lambda/
* Translate: https://aws.amazon.com/translate/
* SES: https://aws.amazon.com/ses/

## Step 1
- [ ] Sign into the AWS Console
- [ ] Navigate to S3
- [ ] Create a bucket called `ses-translate-[firstname]-[lastname]`
- [ ] Use the default settings.
- [ ] On your computer, create an HTML file called `index.html`
- [ ] Copy and paste the following code into `index.html`:
```
<!DOCTYPE html>
<html>
<body>

<h2>HTML Forms</h2>

<form>
  Email address:<br>
  <input type="email" name="mailTo" value="Mickey@Mouse.com">
  <br>
  Subject:<br>
  <input type="text" name="subject" value="Subject">
  <br>
  Email Body:<br>
  <textarea type="text" name="body" value="Email Body" rows="10" cols="50" maxlength="5000"></textarea>
  <br>
  Target Language:<br>
  <select name="targetLanguage">
    <option value="ar">Arabic</option>
    <option value="zh">Chinese (Simplified)</option>
    <option value="zh-TW">Chinese (Traditional)</option>
    <option value="cs">Czech</option>
    <option value="fr">French</option>
    <option value="de">German</option>
    <option value="it">Italian</option>
    <option value="ja">Japanese</option>
    <option value="pt">Portuguese</option>
    <option value="ru">Russian</option>
    <option value="es">Spanish</option>
    <option value="tr">Turkish</option>
  </select>
  <br><br>
  <!-- <input type="submit" value="Submit"> -->
  <button>Submit</button>
</form>

</body>
</html>


<script>
$("button").click(function(){
    $.ajax({
            type: "POST",
            url: 'https://x7x39c25p6.execute-api.us-east-1.amazonaws.com/myStage' ,
            contentType: 'application/json',
            dataType: 'json',
            data: JSON.stringify({
                email:document.getElementById("email").value,
                subject:document.getElementById("subject").value,
                body:document.getElementById("body").value,
                targetLanguage:targetLanguage.options[targetLanguage.selectedIndex].value
            })
        });
    });
</script>

```
- [ ] Upload `index.html` to your bucket
- [ ] In the Upload card, click "next"
- [ ] Change "Manage Public Permissions" to `Grant public read access to this object(s)`
- [ ] Next, Next, Upload...
- [ ] In your bucket, navigate to the `Properties` tab
- [ ] Enable "Static Web Hosting"
- [ ] Use index.html as your index file, don't worry about other files
- [ ] Save.

## Step 2
We will:
- Create a new lambda function
- Create a new role to attach the lambda function
- Create a new custom policy with the following JSON new policy and attach it to the role
-[ ] Navigate to Services > Lambda in the AWS Console
-[ ] Click CREATE A FUNCTION
-[ ] Select `Author From Scratch`
-[ ] Name your function something creative, like `sesLambdaFunction`
-[ ] Under Runtime, select Python 3.6
-[ ] Under Role, select `create a custom role`
-A new tab will open up
-[ ] Name your role `amazonSESFullAccess`
-[ ] Select `edit`
-[ ] Yes, we've read the docs.
-[ ] Paste the following code as your role:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "translate:TranslateText",
                "comprehend:DetectDominantLanguage"
             ],   
            "Resource": "*"
        }
    ]
}
```
- [ ] AWS should automatically attach the policy amazonSESFullAccess to this function.  It will allow SES to send emails
- [ ] Click Create Function
- [ ] Scroll down and paste the following code into the code editor
```
import json
import boto3

def lambda_handler(event, context):
    translate = boto3.client(service_name='translate', region_name='us-east-1', use_ssl=True)
    result = translate.translate_text(Text=event['body']['body'], SourceLanguageCode="en", TargetLanguageCode=event['body']['targetLanguage'])

    ses = boto3.client('ses', region_name='us-east-1')

    ses.send_email(
        Source='email@email.com', //CHANGE ME PLEASE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
        Destination={
            'ToAddresses': [
                event['body']['email'],
            ],
        },
        Message={
            'Subject': {
                'Data': event['body']['subject'],
            },
            'Body': {
                'Text': {
                    'Data':  result.get('TranslatedText'),
                },
            }
        },
    )

    return 'done'
```
-Read through the code and get the idea of what's going on.
-[ ] Change the `Source='email@email.com',` to your own email address.
Boto3 is the python sdk for interacting with AWS services. It makes it really easy for you to run python on AWS.

-[ ] SAVE THE FUNCTION



## Step 3
##### Set Up SES to send emails
###### READ THIS:
Every AWS account SES starts in sandbox mode by default mode. This simply means that you can only send emails to emails that have been verified. AWS does this to limit the amount of spam it sends out. If a use case demands that users don’t have to verify their email, you can request that from aws here. For this lab we will stay in sandbox mode, meaning that the translations can only be sent to people who have verified their emails with SES.

- [ ] Did you read the above paragraph? [ this is on the quiz ]
##### Yes, we're serious. Read the paragraph above.
##### Now that you've read that paragraph, let's move on. Thanks again for finally reading it.
_
-[ ] Navigate to `Services > Simple Email Service` in the AWS Console
-[ ] On the left-hand side of the page (Toward the top, under Identity Management) click Email Addresses
-[ ] Click Verify A New Email Address and add at least one email (this will be the email we are sending to, so you will need access to this email address.)
-[ ] Go into the email and click the link to confirm the email

## Step 4
##### Set Up API Gateway to Call the Lambda Function
-[ ] Navigate to `Services > API Gateway` in the AWS Console
-[ ] Click 'Get Started'
-[ ] Select 'New API'
-[ ] Name your api `email`
-[ ] Give it a description if you so choose!
-[ ] Click CREATE
-[ ] You will be transported to the API Management Page
```
You haven't seen code in a little while.  Here's a visual break for you.
#############################################################
###################################################   #######
###############################################   /~\   #####
############################################   _- `~~~', ####
##########################################  _-~       )  ####
#######################################  _-~          |  ####
####################################  _-~            ;  #####
##########################  __---___-~              |   #####
#######################   _~   ,,                  ;  `,,  ##
#####################  _-~    ;'                  |  ,'  ; ##
###################  _~      '                    `~'   ; ###
############   __---;                                 ,' ####
########   __~~  ___                                ,' ######
#####  _-~~   -~~ _                               ,' ########
##### `-_         _                              ; ##########
#######  ~~----~~~   ;                          ; ###########
#########  /          ;                        ; ############
#######  /             ;                      ; #############
#####  /                `                    ; ##############
###  /                                      ; ###############
#                                            ################
#############################################################

It's a wolf. Probably Balto.
```

-[ ] Ok, back on topic.  On this page, click `Actions > Create Method`
-[ ] In the dropdown, select `POST`
-[ ] Click the little check that's hard to see. It's right next to `POST`
-[ ] Now select region your lambda function is in (probably US-EAST-1)
-[ ] Start to type in the name your lambda function and select your function from the dropdown
-[ ] That's right. Ours *is* called `sesLambdaFunction`
-[ ] Save
-[ ] Click 'OK'

##### We'll now integrate the request JSON data
-[ ] Now click integration request
-[ ] Expand the 'Mapping Templates' section at the bottom
-[ ] Click the plus `+` to add a mapping template
-[ ] Type in application/json
-[ ] Click the check
-[ ] click yes, secure this integration.
-[ ] Paste the following into the template:
```
{
  "body" : $input.json('$'),
  "headers": {
    #foreach($param in $input.params().header.keySet())
    "$param": "$util.escapeJavaScript($input.params().header.get($param))" #if($foreach.hasNext),#end

    #end
  }
}
```
-[ ] Press SAVE

##### Let's Test this Method
-[ ] Go back to method execution (scroll up) and click test (w/ lightning bolt!)
-[ ] Test with this body json object like the object below.
```
{
    "body":"Hello World",
    "subject":"test subject",
	    "targetLanguage":”de",
    "email":"youremail@gmail.com"
}
```
##### If it was successful you should receive an email with the German translation of `Hallo Welt`.

-[ ] Be sure to enable CORS so our website can use the endpoint. You can do this by going to the resources screen, clicking on the resource (the “/”) and clicking the action drop down and selecting enable CORS.
-[ ] Go ahead and deploy your api. Either create a new stage or use an existing one.
-[ ] Copy the Invoke URL and paste it in your index.html. It should go in the javascript ajax call under the property of `url`.
```
$.ajax({
            type: "POST",
            url: 'https://ae92s602pl.execute-api.us-east-1.amazonaws.com/prod' ,
            contentType: 'application/json',
            dataType: 'json',
            data: JSON.stringify({
                email:document.getElementById("email").value,
                subject:document.getElementById("subject").value,
                body:document.getElementById("body").value,
                targetLanguage:targetLanguage.options[targetLanguage.selectedIndex].value
            })
        });
```
You should now be able to send emails with translations from your index.html. Go ahead and give it a try.



With Amazon Translate you can Translate up to 2M  *characters* monthly - free for the first 12 months, starting from your first
translation request.

## Step 5

`ăn tiệc`
`xong.`
[[it means you're finished and can now party]]
