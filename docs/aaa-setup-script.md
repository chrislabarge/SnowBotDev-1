# Script for managing Account Activity API configuration

## Getting started

+ Setting client-side URL for webhook events. Where should we send the event JSON?
+ Looking up what webhook IDs are set up? Confirming what webhook 'bridges' have been set up.



## Setting up webhooks

The setup_webooks.rb script helps automate Account Activity configuration management. https://dev.twitter.com/webhooks/managing

```
Usage: setup_webhooks [options]
    -c, --config CONFIG              Configuration file (including path) that provides account OAuth details. 
    -t, --task TASK                  Securing Webhooks Task to perform: trigger CRC ('crc'), set config ('set'), list configs ('list'), delete config ('delete'), subscribe app ('subscribe'), unsubscribe app ('unsubscribe'),get subscription ('subscription').
    -u, --url URL                    Webhooks 'consumer' URL, e.g. https://mydomain.com/webhooks/twitter.
    -i, --id ID                      Webhook ID
    -h, --help                       Display this screen.  
```


Here are some example commands:

   + setup_webhooks.rb -t "list"

    '''
    Retrieving webhook configurations...
    No existing configurations... 
    '''


   


  + setup_webhooks.rb -t "set" -u "https://snowbotdev.herokuapp.com/snowbot"




  
```
Setting a webhook configuration...
Created webhook instance with webhook_id: 890716673514258432 | pointing to https://snowbotdev.herokuapp.com/snowbot
```
  
If your web app is not running, or your CRC code is not quite ready, you will receive the following response:  
  
```
  Setting a webhook configuration...
error code: 400 #<Net::HTTPBadRequest:0x007ffe0f710f10>
{"code"=>214, "message"=>"Webhook URL does not meet the requirements. Please consult: https://dev.twitter.com/webhooks/securing"}
```  
  + setup_webhooks.rb -t "list"

```
Retrieving webhook configurations...
Webhook ID 890716673514258432 --> https://snowbotdev.herokuapp.com/snowbot
```

  + setup_webhooks.rb -t "delete" -i 883437804897931264 
  
```
Attempting to delete configuration for webhook id: 883437804897931264.
Webhook configuration for 883437804897931264 was successfully deleted.
```


### Adding Subscriptions to a Webhook ID

  + setup_webhooks.rb -t "subscribe" -i webhook_id
  
```
Setting subscription for 'host' account for webhook id: 890716673514258432
Webhook subscription for 890716673514258432 was successfully added.
```

  + setup_webhooks.rb -t "unsubscribe" -i webhook_id
  
```
Attempting to delete subscription for webhook: 890716673514258432.
Webhook subscription for 890716673514258432 was successfully deleted.
```

  + setup_webhooks.rb -t "subscription" -i webhook_id
  
```
Retrieving webhook subscriptions...
Webhook subscription exists for 890716673514258432.
```


### Triggering CRC check 

  + setup_webhooks.rb -t "crc"

```
Retrieving webhook configurations...
204
CRC request successful and webhook status set to valid.
```

If you receive a response saying the 'Webhook URL does not meet the requirements', make sure your web app is up and running. If you are using a cloud platform, make sure your app is not hibernating. 

```
Retrieving webhook configurations...
Too Many Requests  - Rate limited...
error: #<Net::HTTPTooManyRequests:0x007fc4239c1190>
{"errors":[{"code":88,"message":"Rate limit exceeded."}]}
Webhook URL does not meet the requirements. Please consult: https://dev.twitter.com/webhook/security
```

If you receive this message you'll need to wait to retry. The default rate limit is one request every 15 minutes. 


### Updating Webhook URL

Sometimes you need to have the Twitter AA API point to a new client-side webhook URL. When building a bot, you may have Twitter initially send webhook events to a development site, then later point to a production system. 

When this is the case, you need to first delete the current webhook maping, then create a new one. 

+  setup_webhooks.rb -t "list"
```
Retrieving webhook configurations...
Webhook ID 890716673514258432 --> https://snowbotdev_test.herokuapp.com/snowbot
```

  + setup_webhooks.rb -t "delete" -i 890716673514258432 
```
Attempting to delete configuration for webhook id: 890716673514258432.
Webhook configuration for 890716673514258432 was successfully deleted.
```
Running a 'list' task should confirm there are no longer any webhook ids set up for this Twitter app.

Now, we are ready to update to our production 

  + setup_webhooks.rb -t "set" -u "https://snowbotdev.herokuapp.com/snowbotdev"


### Error responses

Only set the consumer key and secret.

```
Setting a webhook configuration...
POST ERROR occurred with /1.1/account_activity/webhooks.json?url=https%3A%2F%2Fsnowbotdev.herokuapp.com%2Fsnowbotdev, request:  
Error code: 403 #<Net::HTTPForbidden:0x007f93fad1f048>
Error Message: {"errors":[{"code":261,"message":"Application cannot perform write actions. Contact Twitter Platform Operations through https://support.twitter.com/forms/platform."}]}
{"code"=>261, "message"=>"Application cannot perform write actions. Contact Twitter Platform Operations through https://support.twitter.com/forms/platform."}
```

Failing CRC check.
```
Setting a webhook configuration...
POST ERROR occurred with /1.1/account_activity/webhooks.json?url=https%3A%2F%2Fsnowbotdev.herokuapp.com%2Fsnowbotdev, request:  
Error code: 400 #<Net::HTTPBadRequest:0x007fe23a197840>
Error Message: {"errors":[{"code":214,"message":"Webhook URL does not meet the requirements. Please consult: https://dev.twitter.com/webhooks/securing"}]}
{"code"=>214, "message"=>"Webhook URL does not meet the requirements. Please consult: https://dev.twitter.com/webhooks/securing"}
```

