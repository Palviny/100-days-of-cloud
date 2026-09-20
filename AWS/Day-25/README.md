Day 25 — Setting Up an EC2 Instance and CloudWatch Alarm

## Objective

The Nautilus DevOps team needs an EC2 instance for an application and wants to monitor its CPU utilization.

The task is to:

Launch an Ubuntu EC2 instance named datacenter-ec2
Create a CloudWatch alarm named datacenter-alarm
Monitor the instance's CPU utilization
Use the Average statistic
Trigger the alarm when CPU utilization is greater than or equal to 90%
Evaluate the condition over one consecutive 5-minute period
Send an alarm notification to the existing SNS topic datacenter-sns-topic
Create all resources in us-east-1

## 1. The Problem

Running an application on an EC2 instance is only part of operating a cloud application.

We also need to know when the instance is experiencing unusually high resource utilization.

For example, if an application server is consistently using 95% CPU, it could indicate:

Increased application traffic
An inefficient application process
A runaway process
Insufficient instance capacity
A need to scale the application

Without monitoring, the operations team may not know that the instance is under pressure until users begin experiencing problems.

The solution is to use Amazon CloudWatch to monitor the EC2 instance and create an alarm that can notify the team when CPU utilization reaches the defined threshold.

### 2. AWS Services Used

This lab uses three main AWS services.

# Amazon EC2

Amazon Elastic Compute Cloud (EC2) provides virtual servers in AWS.

In this lab, the EC2 instance runs the application workload that we want to monitor.

We created:

datacenter-ec2 with an Ubuntu AMI and a t2.micro instance type.

# Amazon CloudWatch

Amazon CloudWatch is an AWS monitoring and observability service. It collects and provides access to metrics from AWS resources and applications.

Examples of metrics that can be monitored include:

EC2 CPU utilization
Load balancer request counts
Database performance
Network activity
Application-specific metrics

In this lab, CloudWatch monitors:

CPUUtilization for datacenter-ec2.

# CloudWatch Alarm 
A CloudWatch alarm evaluates a metric against a defined condition. It can change state when the metric crosses a threshold.

For this lab:
CPU Utilization >= 90%

for: 1 consecutive 5-minute period causes the alarm to enter the ALARM state.

The alarm is named: datacenter-alarm

# Amazon SNS

Amazon Simple Notification Service (SNS) is a messaging and notification service. CloudWatch detects the condition, while SNS is responsible for distributing the notification.

The existing SNS topic used in this lab is: datacenter-sns-topic

# 3. Important CloudWatch Terms

Understanding the terminology is more important than simply knowing where to click.

Metric:A metric is a measurement collected over time.

For this lab, the metric is: CPUUtilization

It represents the percentage of CPU capacity being used by the EC2 instance.

For example: CPUUtilization = 25%

means the measured CPU utilization is approximately 25%.

Statistic: A statistic determines how CloudWatch summarizes metric data over the selected period.

Common statistics include:

Average
Minimum
Maximum
Sum
Sample count

The lab specifically requires: Average

So CloudWatch evaluates the average CPU utilization during each selected period.

Period: The period determines the length of time represented by each data point.

The lab requires: 5 minutes

Therefore, CloudWatch evaluates CPU utilization in 5-minute intervals.

Threshold: A threshold is the value against which the metric is evaluated.

Our threshold is: 90%

The condition is:

CPUUtilization >= 90%

Evaluation Periods: The number of evaluation periods determines how many consecutive periods CloudWatch evaluates before changing the alarm state.

The lab requires:

1 out of 1 datapoints

with a: 5-minute period

Therefore, one 5-minute datapoint meeting the condition is sufficient for the alarm to enter the ALARM state.

CloudWatch does not currently have enough data to determine whether the alarm condition is satisfied. This is important because immediately after creating an alarm, it may temporarily show:

Insufficient data. This does not necessarily indicate a configuration problem.

In this lab, the newly created alarm initially showed Insufficient data, while the alarm configuration itself was correct.


# 4. Implementation
Step 1 — Launch the EC2 Instance

Navigate to: AWS Console → EC2 → Instances → Launch instances

Configure the instance as follows:

Setting	Value
Name	datacenter-ec2
AMI	Ubuntu
Instance type	t2.micro
Region	us-east-1

The task does not require a specific Ubuntu AMI version, so an appropriate Ubuntu AMI can be selected.

Why Ubuntu?

The task specifically asks for an Ubuntu-based EC2 instance.

The exact operating system is not important to the CloudWatch CPU monitoring portion of the lab because standard EC2 CPU utilization is provided by AWS.

Step 2 — Launch the Instance

After reviewing the configuration, launch the instance.

Navigate back to:

EC2 → Instances

Find: datacenter-ec2

Wait until: Instance state: Running

and:

Status checks: 2/2 checks passed Verification

The instance successfully reached the Running state with both status checks passing.

<img width="1663" height="215" alt="Screenshot 2026-09-20 043703" src="https://github.com/user-attachments/assets/7e2c0a2f-db45-4eae-ba37-e3e1a65ec73d" />


## 5. Create the CloudWatch Alarm

Navigate to: CloudWatch → Alarms → All alarms → Create alarm

Select the EC2 metric: AWS/EC2 → Per-Instance Metrics → CPUUtilization

Select the metric associated with: datacenter-ec2

The EC2 instance ID can be used to distinguish the correct instance.

CPU utilization is a standard EC2 metric provided by AWS. It is useful for identifying workloads that may be approaching the processing capacity of an instance.

For standard EC2 CPU monitoring, we do not need to install the CloudWatch Agent simply to obtain CPU utilization.

<img width="1685" height="850" alt="Screenshot 2026-09-20 044722" src="https://github.com/user-attachments/assets/69f035a4-7676-4d64-be46-2e84250f2072" />


## 6. Configure the Metric

Configure the metric with the following settings:

Statistic
Average

This tells CloudWatch to calculate the average CPU utilization during each evaluation period.

Period
5 minutes

Each datapoint represents a 5-minute period.

Condition
Greater than or equal to
Threshold
90%

Therefore, the complete condition is:

Average CPUUtilization >= 90%
Evaluation
1 out of 1 datapoints

This means that one consecutive 5-minute period meeting the threshold is sufficient for the alarm to trigger.

The resulting configuration is:

Metric: CPUUtilization
Statistic: Average
Period: 5 minutes
Condition: >= 90%
Evaluation: 1 out of 1 datapoints

<img width="1685" height="850" alt="Screenshot 2026-09-20 044722" src="https://github.com/user-attachments/assets/589b0c87-995a-4d7b-9b34-b85b0304bd8f" />


## 7. Configure the SNS Notification

Under the alarm notification settings, select:

In alarm: This means the action is performed when the CloudWatch alarm enters the ALARM state.

Select an existing SNS topic

Then choose: datacenter-sns-topic

We do not create a new SNS topic because the lab specifically provides an existing topic.

The resulting relationship is:

CloudWatch Alarm
       │
       │ In alarm
       ▼
datacenter-sns-topic

<img width="1681" height="856" alt="Screenshot 2026-09-20 045023" src="https://github.com/user-attachments/assets/bb202bef-eb8c-422a-8abf-580ad39bfb9f" />

 
## 8. Name the Alarm

Set the alarm name to: datacenter-alarm

Review the configuration and create the alarm.

<img width="1905" height="427" alt="Screenshot 2026-09-20 045245" src="https://github.com/user-attachments/assets/2820d344-a2d5-4d97-b986-e76dde4c5f35" />


## 9. Verification

After creating the alarm, verify the alarm details.

<img width="1908" height="760" alt="Screenshot 2026-09-20 045337" src="https://github.com/user-attachments/assets/0f11f8a1-6461-4d79-9c9f-060ea453876b" />


<img width="1633" height="820" alt="Screenshot 2026-09-20 045413" src="https://github.com/user-attachments/assets/7838652a-e493-4af0-b6eb-98e36e712445" />


<img width="1689" height="616" alt="Screenshot 2026-09-20 045430" src="https://github.com/user-attachments/assets/b747a1a6-f613-4774-9fff-4a11f0facb4a" />

The resulting configuration should show:

Setting	Value
Alarm name	datacenter-alarm
Namespace	AWS/EC2
Metric	CPUUtilization
Instance	datacenter-ec2
Statistic	Average
Period	5 minutes
Threshold	>= 90%
Datapoints to alarm	1 out of 1
Actions	Enabled
SNS topic	datacenter-sns-topic

The alarm initially showed:

Insufficient data: This was expected because CloudWatch had not yet accumulated enough metric data to evaluate the newly created alarm.

<img width="1666" height="808" alt="Screenshot 2026-09-20 045619" src="https://github.com/user-attachments/assets/3bf4c430-3960-48be-86c4-e51790316c8e" />

<img width="1267" height="768" alt="Screenshot 2026-09-20 045643" src="https://github.com/user-attachments/assets/4b498e91-142c-4bb4-855a-6c83453b5859" />




Real-World Relevance: This type of monitoring is common in cloud operations and DevOps environments.

For example, a company may configure:

CPU >= 80%
        ↓
CloudWatch Alarm
        ↓
SNS
        ↓
Operations notification

The team can then investigate the cause.

Possible causes of high CPU could include:

Increased traffic, Inefficient application code, Background processing, Resource-intensive workloads, Unexpected processes, Insufficient instance capacity

CloudWatch alarms can also become part of automated systems.

For example:

High CPU
   ↓
CloudWatch Alarm
   ↓
Auto Scaling
   ↓
Additional EC2 capacity

That moves the architecture from simply monitoring a problem to potentially responding automatically to it.
