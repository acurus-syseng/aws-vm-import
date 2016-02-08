# aws-vm-import
Scripts to import a VM image to AWS


VM Import via AWS CLI
VM Prerequisites
•	Disable an A/V on VM
•	Uninstall VMWare Tools
•	Disconnect CD-ROM drives
•	In Windows, set NIC to DHCP rather than static
•	Ensure RDP is Enabled
•	All local accounts must have passwords set
•	Ensure .NET 3.5 or later is installed
•	Ensure Autologon is not enabled
•	Ensure no pending MS updates waiting for installation on reboot
•	Install the following KB’s if applicable
o	https://support.microsoft.com/en-us/kb/2922223
o	https://support.microsoft.com/en-us/kb/2800213
•	Enable RealTimeIsUniversal registry key
o	HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation
o	REG_DWORD – RealTimeIsUniversal
o	Value – 1

Export Process
1.	Perform a Graceful Shutdown of VM
2.	Select the Export OVF Template menu option
3.	Export to desired location, using Format: Single File (OVA)

AWS Prerequisites
•	Create an IAM user with the AdministratorAccess role attached
•	Save the Access Key and Secret Key details – these can only be retrieved once
•	Create an S3 bucket with an easily identifiable name to upload the OVA templates to
•	Create below two files in notepad
o	trust-policy.json
    - Save into the AWS CLI directory – default: C:\Program Files\Amazon\AWSCLI
    - Contents listed at end of document
o	role-policy.json
    - Save into the AWS CLI directory – default: C:\Program Files\Amazon\AWSCLI
    - Contents listed at end of document

Import Process
This procedure assumes the AWS CLI tools have already been installed on the machine that you are using to perform the import. If not:: http://docs.aws.amazon.com/cli/latest/userguide/installing.html#install-msi-on-windows
The machine being used to do the import should have file access to the exported OVA
1.	Change to CLI tools directory
a.	CD “C:\Program Files\Amazon\AWSCLI”
2.	Configure the AWS Tools for relevant account 
a.	aws configure
i.	Access Key: <Saved Access Key>
ii.	Secret Key: <Saved Secret Key>
iii.	Region: <Select region code from http://docs.aws.amazon.com/general/latest/gr/rande.html >
iv.	Default Output: JSON
3.	Configure Role and Trust policy to allow VM Import
a.	aws iam create-role --role-name vmimport --assume-role-policy-document file://trust-policy.json
b.	aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document file://role-policy.json
4.	Upload OVA to S3
a.	aws s3 cp <local OVA location> s3://<s3 bucket name> --recursive
5.	Import image from S3 to EC2
a.	aws ec2 import-image --cli-input-json "{  \"Description\": \"<OVA file Name>\", \"DiskContainers\": [ { \"Description\": \"<OVA file Name>\", \"UserBucket\": { \"S3Bucket\": \"<S3 Bucket Name>\", \"S3Key\" : \"<OVA File Name>\" } } ]}"
6.	To view the status of the import
a.	aws ec2 describe-import-image-tasks


JSON File Contents
trust-policy.json
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Sid":"",
         "Effect":"Allow",
         "Principal":{
            "Service":"vmie.amazonaws.com"
         },
         "Action":"sts:AssumeRole",
         "Condition":{
            "StringEquals":{
               "sts:ExternalId":"vmimport"
            }
         }
      }
   ]
}






role-policy.json

{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect":"Allow",
         "Action":[
            "s3:ListBucket",
            "s3:GetBucketLocation"
         ],
         "Resource":[
            "arn:aws:s3:::<S3 Bucket Name>"
         ]
      },
      {
         "Effect":"Allow",
         "Action":[
            "s3:GetObject"
         ],
         "Resource":[
            "arn:aws:s3:::<S3 Bucket Name>/*"
         ]
      },
      {
         "Effect":"Allow",
         "Action":[
            "ec2:ModifySnapshotAttribute",
            "ec2:CopySnapshot",
            "ec2:RegisterImage",
            "ec2:Describe*"
         ],
         "Resource":"*"
      }
   ]
}
