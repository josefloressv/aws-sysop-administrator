# Scaling EC2 Using SQS
## Introduction
In this scenario, you're a solutions architect at an e-commerce firm. The company runs flash sales from time to time, and when there's a spike in orders, the fulfillment backend can struggle to meet demand. One way to solve the problem is to overprovision EC2 instances in the fulfillment system to provide headroom to process all the orders. However, this can be very costly, since you'll have unused capacity when the traffic subsides. What if there's a better way? Well, there is, and this is the problem you'll solve here. In this lab, you will learn to create Auto Scaling rules for EC2 based on the number of messages in an SQS queue.

## Solution
Log in to the live AWS environment using the credentials provided. Make sure you're in the N. Virginia (us-east-1) region throughout the lab.

### Preparing
> Is necesary to generate logs in CloudWatch to create SQS metrics for the alarms and to simulate a spike in traffic by sending messages into the queue.
1. Connect to the *Bastion Host instance*  through SSH and the public IP.
```
ssh cloud_user@<BASTION_HOST_PUBLIC_IP>
```
7. Run the script named send_messages.py
```
./send_messages.py
# (this will send messages continuously into our SQS queue, simulating a large volume of orders)
```

### Create CloudWatch Alarms
#### Create Scale-Out Alarm
1. Back in the AWS console, navigate to CloudWatch > Alarms.
2. Click Create alarm.
3. Click Select metric.
4. Select SQS > Queue Metrics.
5. Click the checkbox of the metric named *ApproximateNumberOfMessagesVisible*.
>Note: it takes a few minutes before the *ApproximateNumberOfMessagesVisible* metric appears in CloudWatch).
6. Click Select metric.
7. On the Specify metric and conditions page, change the Period to 1 minute.
8. In the Conditions section, set the following values:
```
Threshold type: Static
Whenever *ApproximateNumberOfMessagesVisible* is...: Greater
than...: 500
```
9. Click Next.
10. On the Configure actions page, click into the Send a notification to... box and select the listed AutoScalingTopic.
11. Click Next.
12. For Alarm name, enter "Scale Out".
13. Click Next.
14. Click Create alarm.

#### Create Scale-In Alarm
1. Repeat steps 1-7 above
2. In the Conditions section, set the following values:
```
Threshold type: Static
Whenever *ApproximateNumberOfMessagesVisible* is...: Lower
than...: 500
```
3. Click Next.
4. On the Configure actions page, click into the Send a notification to... box and select the listed AutoScalingTopic.
5. Click Next.
6. For Alarm name, enter "Scale In".
7. Click Next.
8. Click Create alarm.

### Create Simple Scaling Policies
#### Scale-Out Policy
1. Navigate to EC2 > Auto Scaling Groups.
2. Click the listed Auto Scaling group.
3. Click the Automatic scaling tab.
4. Click "Create dynamic scaling policy"
5. Set the following values:
```
Policy type: Simple scaling
Scaling policy name: Scale Out
CloudWatch alarm*: Scale Out
Take the action: Add 1 capacity units
And then wait: 60 seconds before allowing another scaling activity
```
6. Click Create.

#### Scale-In Policy
1. Repeate the steps 1-4 above
2. Set the following values:
```
Policy type: Simple scaling
Scaling policy name: Scale In
CloudWatch alarm*: Scale In
Take the action: Remove 1 capacity units
And then wait: 60 seconds before allowing another scaling activity
```
3. Click Create.

### Create a Dashboard
1. Navigate to CloudWatch > Dashboards.
2. Click in Create Dashboard.
3. Select the widget type *Stacked Area* to add to the dashboard.
4. Chose Metrics for the source of the widget
```
SQS > Queue Metrics
ApproximateNumberOfMessagesVisible
Period: 1 minute
```
5. Set the name for the grap: SQS
6. Clic in Create Widget.
7. Click in Save Dashboard.

## Verify
1. See the dashboard in the AWS console.
2. See the alarm in the AWS console.
3. See the Activity tab for the AutoScaling Group in the AWS Console.
> After a minute or so, we should see it launches an additional EC2 instance.
4. See the SQS events in the AutoScaling Group instance through SSH.
```
tail -f receive_messages.log

# We'll see this instance is retrieving messages from the queue.
# Leave the terminals open and running.
```
5. Simulate a slow traffic by stopping sending messages into the queue.
```
# In the terminal running the send_messages.py script, press Ctrl+C to cancel the process (simulating the volume of sales slowing down).
# Wait a few minutes more, and we should see the graph takes a downward turn as the messages are drained from our SQS queue.
```
