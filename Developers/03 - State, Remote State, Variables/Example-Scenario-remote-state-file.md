
Agenda:
<p>

* State, remote state, variables
* Project Name: timing


* Terraform is a declarative way of approach
* Whatever you declare, you'll get that infra
* Whenver terraform creates infra, it generates file called terraform.tfstate


* To track what terraform has created, the status is automatically stored in "terraform.tfstate"



* TF files => whatever we want = Desired infra
* terraform.tfstate = Actual infra = current state of infra

* Desired Infra = Actual Infra

* terraform will always maintain the state whatever you declare

* If you remove the terraform.tfstate file, it will again recreate. 
* This is the probelem. It relies on terraform.tfstate file to maintain the change.
</p>

<br>

Problems:
<p>

* Keeping terraform.tfstate in local is a problem.

* If you lose the data then terraform can't track what happened earlier. It'll try to recreate

* In case of version control, keeping the terraform state in Github also causes problem while infra is creating through CI CD.

* The Best practice is to keep the state file in Remote S3 location for better collaboration between team members.
</p>

<br>

Question:
<p>

* Explain the state in terraform?
* Ans: Terraform uses state file to check the desired configuration VS actual configuration.

* If you keep the state file in local, there is a chance of duplication of resources and errors.

* Always you should keep the state file in remote location like S3 and you should lock it with Dynamo DB. 

* In that case, if one person is performing the terraform changes then it'll be locked and another person cannot be able to do any changes. The state is always secured.
</p>


<br>
<br>

<p>

Working on state file: Example Scenario
* Infra team, multiple members 

* Every person will contribute some work, we use Git for version control

Engineer - 1
* He cloned repo


Engineer - 2
* He too cloned the repo


Engineer - 1
* He applied terraform


Engineer - 2
* He also applied terraform

* So, duplication of resources occur. Both Engineers are working in parallel
</p>

<br>

* To overcome this,

* A central place where terraform can understand the Actual infra!


* Engineer1: When Engineer1 perform "terraform apply", it'll place terraform.statefile to S3 bucket. Infra will be created

* Engineer2: When Engineer2 also performed "terraform apply", then it will comapre with the remote file (the terraform.statefile in S3 bucket).

<br>

Advantages:
<p>

* Multiple person should not change infra at a time, only one change at a time is allowed. --> lock it using DynamoDB

* Duplication and errors in terraform will be removed.

* If you keep the terraform.statefile in local, it'll creates the duplicate infra.

* But if you place the state file in S3, terraform will compare the state with the state file in remote.
</p>

<br>
<br>

Steps to create S3 bucket and table in dynamoDB:
<p>

* Create S3 bucket and name it as For example: timing-remote-state-bucket
* Create a table in dynamoDB, Table name as: timing-lock for example.
* Partition key: LockID
* Click on Create


* Now, how terraform will keep the state inside the lock file is... 

<br>

* Understand the state:
<p>

* Inside provider.tf, you should tell terraform to don't store the state in local, instead store it in S3 bucket.

* Reference for mentioning terraform remote state S3:
https://developer.hashicorp.com/terraform/language/settings/backends/s3


<br>

* Engineer1: He is working on and making changes

```
$ terraform apply
```
<p>

Plan: 0 to add, 1 to change, 0 to destroy.
aws_route_table.private-rt: Modifying... [id=rtb-08fdbb16a1d9eb297]
aws_route_table.private-rt: Modifications complete after 0s [id=rtb-08fdbb16a1d9eb297]
</p>

<br>

* Engineer2: This guy also wants to perform some activities...
* But, the time he applies...

```
$ terraform apply -auto-approve
```
<p>

```
│ Error: Error acquiring the state lock
│
│ Error message: ConditionalCheckFailedException: The conditional request
│ failed
│ Lock Info:
│   ID:        7acf6be6-eae6-a374-7f00-a4ac2c5fad99
│   Path:      timing-remote-state-bucket/timing
│   Operation: OperationTypeApply
│   Who:       ANAGA\anaga@ANAGA
│   Version:   1.4.6
│   Created:   2023-04-30 17:03:52.1932675 +0000 UTC
│   Info:
│
│
│ Terraform acquires a state lock to protect the state from being written
│ by multiple users at the same time. Please resolve the issue above and try
│ again. For most commands, you can disable locking with the "-lock=false"
│ flag, but this is not recommended.
```
</p>

* This is the LockID you'll find when someone is working on...
LockID: timing-remote-state-bucket/timing-md5


* Actually it states that...

* One person is already running the infrastructure, you shouldn't run this at the same time! 

* Engineer2: If he applies "terraform apply"
* Engineer2 infrastructure will be comapred with the state file in the S3 location.

* There will be no changes after he applies. Resources are not duplicated!

<br>
<br>

Question:
<p>

* Explain the state in terraform?
* Ans: Terraform uses state file to check the desired configuration VS actual configuration.

* If you keep the state file in local, there is a chance of duplication of resources and errors.

* Always you should keep the state file in remote location like S3 and you should lock it with Dynamo DB. 

* In that case, if one person is performing the terraform changes then it'll be locked and another person cannot be able to do any changes. The state is always secured.
</p>

