# VM Import via AWS CLI
Scripts to import a VM image to AWS

<h2>VM Prerequisites</h2>

<ul>
<li> Disable an A/V on VM </li>
<li> Uninstall VMWare Tools </li>
<li> Disconnect CD-ROM drives </li>
<li> In Windows, set NIC to DHCP rather than static </li>
<li> Ensure RDP is Enabled </li>
<li> All local accounts must have passwords set </li>
<li> Ensure .NET 3.5 or later is installed </li>
<li> Ensure Autologon is not enabled </li>
<li> Ensure no pending MS updates waiting for installation on reboot </li>
<li> Install the following KB’s if applicable </li>
<ul>
<li> https://support.microsoft.com/en-us/kb/2922223 </li>
<li> https://support.microsoft.com/en-us/kb/2800213 </li>
</ul>
<li> Enable RealTimeIsUniversal registry key </li>
<ul> 
<li> HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation </li>
<li> REG_DWORD – RealTimeIsUniversal </li>
<li> Value – 1 </li>
</ul>
</ul>
<br />

<h2>Export Process</h2>
<ol>
<li> Perform a Graceful Shutdown of VM </li>
<li> Select the Export OVF Template menu option </li>
<li> Export to desired location, using Format: Single File (OVA) </li>
</ol>

<h2>AWS Prerequisites</h2>
<ul>
<li> Create an IAM user with the AdministratorAccess role attached </li>
<li> Save the Access Key and Secret Key details – these can only be retrieved <b>once</b> </li>
<li> Create an S3 bucket with an easily identifiable name to upload the OVA templates to </li>
<li> Create below two files in notepad </li>
<ul>
<li> trust-policy.json </li>
<ul>
<li> Save into the AWS CLI directory – default: C:\Program Files\Amazon\AWSCLI </li>
<li> Contents listed at end of document </li>
</ul>
<li> role-policy.json </li>
<ul>
<li> Save into the AWS CLI directory – default: C:\Program Files\Amazon\AWSCLI </li>
<li> Contents listed at end of document </li>
</ul>
</ul>

<h2>Import Process</h2>
This procedure assumes the AWS CLI tools have already been installed on the machine that you are using to perform the import. If not:  <a href>http://docs.aws.amazon.com/cli/latest/userguide/installing.html#install-msi-on-windows</a> </br>
The machine being used to do the import should have file access to the exported OVA</br>
<ol>
<li> Change to CLI tools directory </li>
<ol type="a">
<li> CD “C:\Program Files\Amazon\AWSCLI” </li>
</ol>
<li> Configure the AWS Tools for relevant account </li>
<ol type="a">
<li> aws configure </li>
<ol type="i">
<li> Access Key: <Saved Access Key> </li>
<li> Secret Key: <Saved Secret Key> </li>
<li> Region: Select region code from http://docs.aws.amazon.com/general/latest/gr/rande.html </li>
<li> Default Output: JSON </li>
</ol>
</ol>
<li> Configure Role and Trust policy to allow VM Import </li>
<ol type="i">
<li> aws iam create-role --role-name vmimport --assume-role-policy-document file://trust-policy.json </li>
<li> aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document file://role-policy.json </li>
</ol>
<li> Upload OVA to S3 </li>
<ol type="a">
<li> aws s3 cp <local OVA location> s3://<s3 bucket name> --recursive </li>
</ol>
<li> Import image from S3 to EC2 </li>
<ol type="a">
<li> aws ec2 import-image --cli-input-json "{  \"Description\": \"<OVA file Name>\", \"DiskContainers\": [ { \"Description\": \"<OVA file Name>\", \"UserBucket\": { \"S3Bucket\": \"<S3 Bucket Name>\", \"S3Key\" : \"<OVA File Name>\" } } ]}" </li>
</ol>
<li> To view the status of the import </li>
<ol type="a">
<li> aws ec2 describe-import-image-tasks </li>
</ol>
</ol>

