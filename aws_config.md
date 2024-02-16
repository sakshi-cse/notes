# AWS Config EventBridge Rule for Resource Creation Notification

## Overview

This GitHub organization file provides step-by-step instructions to set up an EventBridge rule in AWS Config. This rule will notify you through Amazon SNS when a new resource of a specified type is created.

### Prerequisites

- An existing Amazon SNS topic in the same Region as your AWS Config service.
- Access to the AWS Management Console.

## Steps

1. If you haven't already created an Amazon SNS topic, follow the instructions in the [Getting started with Amazon SNS](https://docs.aws.amazon.com/sns/latest/dg/sns-getting-started.html).

   Note: Ensure that the Amazon SNS topic is in the same Region as your AWS Config service.

2. Open the [EventBridge console](https://console.aws.amazon.com/events/), and choose **Rules** from the navigation pane.

3. Choose **Create rule**.

4. Enter a name for your rule and optionally provide a description.

5. For **Rule type**, choose **Rule with an event pattern**, then choose **Next**.

6. For **Event source**, choose **AWS events** or **EventBridge partner events**.

7. In the **Event pattern** pane, choose **Custom patterns (JSON editor)**, and paste the following example event pattern:

    ```json
    {
      "source": ["aws.config"],
      "detail-type": ["Config Configuration Item Change"],
      "detail": {
        "messageType": ["ConfigurationItemChangeNotification"],
        "configurationItem": {
          "resourceType": ["AWS::EC2::Instance"],
          "configurationItemStatus": ["ResourceDiscovered"]
        }
      }
    }
    ```

    Note: You can replace the `AWS::EC2::Instance` resource type with other resources. For a list of available resource types, see the `resourceType` section in [ResourceIdentifier](https://docs.aws.amazon.com/config/latest/developerguide/resource-config-reference.html#resourceidentifier).
   ```json
    {
      "source": ["aws.config"],
      "detail-type": ["Config Configuration Item Change"],
      "detail": {
        "messageType": ["ConfigurationItemChangeNotification"]
      }
    }
    ```

    Note: This event pattern will match all resource creation events in AWS Config.


9. Choose **Next**.

10. For **Target types**, select **AWS service**.

11. For **Select a target**, choose **SNS topic**.

12. For **Topic**, choose your SNS topic.

13. Expand **Additional settings**. Then, for **Configure target input**, choose **Input transformer**.

14. Choose **Configure input transformer**. Then, under **Target input transformer** for the **Input Path** text box, enter the following example path:

    ```json
    {
        "awsRegion": "$.detail.configurationItem.awsRegion",
        "awsAccountId": "$.detail.configurationItem.awsAccountId",
        "resource_type": "$.detail.configurationItem.resourceType",
        "resource_ID": "$.detail.configurationItem.resourceId",
        "configurationItemCaptureTime": "$.detail.configurationItem.configurationItemCaptureTime"
    }
    ```

15. For the **Template** text box, enter the following example template:

    ```
    "On <configurationItemCaptureTime> AWS Config service recorded a creation of a new <resource_type> with Id <resource_ID> in the account <awsAccountId> region <awsRegion>. For more details open the AWS Config console at https://console.aws.amazon.com/config/home?region=<awsRegion>#/timeline/<resource_type>/<resource_ID>/configuration"
    ```

16. Choose **Confirm**. Then, choose **Next**.

17. Optionally, you can **Add new tag**. Then, choose **Next**.

18. Choose **Create rule**.

19. If an event type is initiated, you will receive an SNS email notification with the custom fields populated from step 13 similar to the following:

    ```
    "On ExampleTime AWS Config service recorded a creation..."
    ```

That's it! You have successfully set up an EventBridge rule to receive notifications for new resource creations.
