#Prerequisites for the backup and cleanup operation:
•	SETUP IAM PERMISSIONS
	1. Login to your AWS Management console -> Services -> IAM under Security & Identity.
	2. IAM Dashboard -> Roles -> Create New Role with the Role Name: lamda-ec2-ami-role.
	3. AWS Service Roles -> Select AWS Lambda as the Role Type -> proceed to create a role
	4. Go to policies -> Create Policy -> Create your own policy i.e. lamda-ec2-ami-policy
	5. Paste the following json in Policy Document -> Create Policy 
		{
		  "Version":"2012-10-17",
		  "Statement":[
		    {
		      "Effect":"Allow",
		      "Action":[
		        "logs:*"
		      ],
		      "Resource":"arn:aws:logs:*:*:*"
		    },
		    {
		      "Effect":"Allow",
		      "Action":"ec2:*",
		      "Resource":"*"
		    }
		  ]
		}

•	SCHEDULE FUNCTIONS
	1. Login to AWS Management console -> Services -> click on Lambda under Compute.
	2. Go to Triggers tab -> Click on Add Trigger
	3. Select Cloudwatch Events - Schedule
	4. Type Rule Name (ex. create-ami)
	5. Schedule Expression ->select cron and modify accordingly with the schedule time
	6. Enable Trigger -> Submit

•	TAGGING EC2 INSTANCE
	1. Login to AWS Management console -> Go to Services -> click on EC2 under Compute
	2. Instances -> Select the Instance you want to tag (e.g. Linux-test) 
	3. Tags -> Add Tags and add a tag with Backup with no value and with Retention value 7(which is also the default value)

# Python Script 1 -> Automated AMI Backups

•	This script will first search for all ec2 instances having a tag with "Backup” on it and the instances in "Running" state.

•	As soon as it has the instances list, it loops through each instance and then creates an AMI of it.

•	Also, it will look for a "Retention" tag key which will be used as a retention policy number in days. If there is no tag with that name, it will use a 7 days default value for each AMI.

•	After creating the AMI it creates a "DeleteOn" tag on the AMI indicating when it will be deleted using the Retention value and another Lambda function.


# Python Script 2 -> Automated AMI and Snapshot Deletion

•	This script first searches for all ec2 instances having a tag with "Backup” on it and the instances in "Running" state.

•	As soon as it has the instances list, it loops through each instance and reference the AMIs of that instance.

•	It checks that the latest daily backup succeeded then it stores every image that's reached its DeleteOn tag's date for deletion.

•	It then loops through the AMIs, de-registers them and removes all the snapshots associated with that AMI.


# Important Note:

Please specify your AWS Account Number in the place of "XXXXX" where ever in the code