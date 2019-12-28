<p align="center">
    <img src="images/s3_logo.jpg">
</p>

This repo is a concise summary and replacement of the [S3 Masterclass](https://acloud.guru/learn/s3-masterclass) tutorial by "A Cloud Guru".

- [Chapter 2 - The Basics of S3](#Chapter-2---the-basics-of-s3)
- [Chapter 3 - Security: Access Control](#Chapter---3-security-access-control)
- [Chapter 4 - Security: Logging and Monitoring](#Chapter-4---security-logging-and-monitoring)
- [Chapter 5 - Security: Data Protection](#Chapter-5---security-data-protection)
- [Chapter 6 - Lifecycle Management](#Chapter-6---lifecycle-management)
- [Chapter 7 - Event Notifications](#Chapter-7---event-notifications)
- [Chapter 8 - Performance Optimization](#Chapter---8-performance-optimization)
- [Chapter 9 - Website Hosting](#Chapter-9---website-hosting)


# Chapter 2 - The Basics of S3

### S3 Basics

- __Do buckets have regions?__ - (5:30) Yes. A bucket must be created in a specific region.
- __What type of namespace does s3 have?__ - (5:45) a universal namespace. Bucket names must be unique globally (across everybody's accounts) regardless of the region they're created in.
- __How can you access a bucket?__ (6:25)


![s3 URL](images/s3URL.png)

- __What do s3 objects consist of?__ (8:40)
  - Key = name of object.
  - Value = the data being stored (up to 5TB).
  - Version ID = a string of data assigned to an object when versioning is enabled.
    - Bucket + Key + Version ID uniquely identify an object in S3.
  - Metadata = name-value pairs which are used to store information about the object
  - Subresources = additional resources specifically assigned to an object.
    - Example Subresource: Access control information = Policies for controlling access to the resource.
- __What kind of file structure is S3?__ - (9:55) S3 is a flat file structure. Directories are imitated by use of "prefixes".
  - For example, `http://acloudguru.s3.amazonaws.com/images/thumbnails/public/me.jpg` has `http://acloudguru.s3.amazonaws.com` as the bucket name, and `images/thumbnails/public/me.jpg` as the object key. The `images/thumbnails/public/` part are the nested prefixes.
- __What is object tagging?__ - (12:10) Lets you categorize objects by key/value pair tags, such as `Project = acloudguru`
- __What is s3's consistency model?__ - (13:25)
  - for "puts of new objects": read-after-write consistency. Data can only be read after its been successfully written to all physical facilities and returns success.
  - for "overwrite puts" (updates, deletes): eventual consistency. This is done to provide a lower latency and high throughput.
- __Does s3 lock objects while writing to them?__ - (14:50) No. If 2 requests are made at about the same time, the one with the latest timestamp wins
- __Does s3 have cross-region replication?__ - (15:40) Yes, it's optional
- __Does s3 have versioning?__ - (15:55) Yes, if enabled, it provides the ability to retrieve every version of every object ever stored in s3, even if the object is deleted.
- __What security does s3 have?__ (16:15)
  - _access_: only the bucket and object owners have access by default. Permissions to objects and buckets can be granted using "policies"
  - _encryption_: data can be encrypted at rest, and in transit.
  - _audit trails_: all access to s3 resources can be optionally logged to provide audit trails
- __What kind of service is S3?__ - (16:50) S3 is a RESTful web service. We interact with it over web-based protocols such as http & https. Instead of directly using the REST API, we usually interact with it using wrappers such as
  - AWS Management Console
  - AWS CLI (Command Line Interface)
  - AWS SDKs (Software Development Kits)

### Object Storage classes

- __What types of storage classes does s3 have?__ (1:00)
  - Standard - millisecond access to data.
  - Standard Infrequent Access - lower fee than "Standard", but you're charged a retrieval/revival fee. You still have millisecond access to data. A good example is storing "backup data" in this storage class.
  - One Zone Infrequent Access - Same as "Standard Infrequent Access", but cheaper since it's stored in only 1 availability zone.
  - Glacier - designed for archived data. Data is not available in real-time. Restoring data takes between minutes (more expensive) and hours (less expensive). Minimum storage duration is 90 days, so even if you store your object for 1 day, you're billed for 90 days.
  - Glacier Deep Archive - Same as Glacier. Cheaper, slower. Restoring data takes hours. Minimum storage duration is 180 days.
  - Intelligent Tiering - Automatically moves data between tiers. This storage class is useful if you're unsure how often you'll want to access the data.
  - [More information from AWS s3 documentation](https://aws.amazon.com/s3/storage-classes)
- __What are "Lifecycle Policies__?" (9:40) Allows objects to transition between storage classes. Applied to an entire bucket or group of objects. This is different than "Intelligent Tiering" which applied to a single object at a time.

### Buckets and Objects Lab

- __How do you retrieve s3 bucket custom metadata that you created?__ - (17:40) it's retrieved when doing a GET request, as part of the header.
- __What are 2 ways to categorize s3 objects?__ (18:00) can categorize files by either tags, or folders (prefixes).


# Chapter 3 - Security: Access Control

### An Introduction to S3 Permissions

- __Are s3 objects private or public by default?__ - (1:15) private.
- __What are 2 types of policies we can use to access s3 data?__ - (1:35)
   - _Resource-based policies_ - applied directly to a resource such as a bucket or object.
     1. Access Control Lists (ACL) - for AWS Accounts & Predefined Groups
     1. Bucket Policies - for AWS Accounts & IAM Users. Bucket Policies mostly replace ACLs (ACL is considered legacy).
   - _User policies_ - applied to IAM users, groups, or roles in your account
- __How give access to a user in a different AWS account?__ - (5:30)
  1. Account A uses _bucket policy_ to give Account B access.
  1. Account B delegates uses a _user policy_ to give a user access to Account A
  1. User in Account B can access Account A's s3 bucket
- __If 1 policy grants access, and another denies access, who wins?__ - (8:00) deny wins.

User and bucket in same AWS account

![Policy Diagram](images/policy1.png)

User and bucket in different AWS accounts

![Policy Diagram](images/policy2.png)

User in different AWS account than bucket or object

![Policy Diagram](images/policy3.png)

- __What is 3 thing ACLs can do that Bucket Policies can't do, or can't do well?__ (12:55)
  - If a bucket has objects that are not owned by the bucket owner, then the bucket owner can't apply a bucket policy that applies to that object. The object's owner would need to use an ACL to manage access to that object.
  - Having different access for individual objects in a bucket. It's easier to do this with an ACL than a bucket policy.
  - Grant access to the "predefined group" of "S3 Log Delivery Group" on a bucket, so s3 can write access logs to a bucket.
- __What must a bucket policy be used for, that ACL can't do?__ (14:50) use bucket policy when granting permissions more complex than ACL's basic read/write.

### Access Control Lists (ACLs) Lab

- __What permissions do a bucket and object get when created?__ - (0:50) just an ACL with full permissions for resource owner. No bucket policies or user policies are created.
- __Can ACL give access to IAM users?__ - (1:40) No. Only AWS accounts and predefined users.
- __What kind of permissions do ACLs give?__ - (1:50) Only basic read/write
- __What is a Canonical ID?__ - (3:00) A long string that represents our AWS account. [More info](https://docs.aws.amazon.com/general/latest/gr/acct-identifiers.html)
- __What types of ACL permissions are there?__ - (4:40) see image below

![ACL Permissions](images/aclPermissions.png))

- __What are ACL predefined groups?__ - (6:15) groups with a URI you can use to represent them
  - _Authenticated Users_
  - _All Users_
  - _S3 Log Delivery Group_
- __What are canned ACLs?__ - (7:40) predefined grants of permissions. See image below

![Canned ACLs](images/cannedACLs.png)

### Bucket & User Policies

- __What are 2 ways to give a user access to a bucket?__ (0:35)
  1. Bucket Policy (attached to Bucket)
  1. User Policy (attached to User)
- __What 5 elements do bucket policies have?__ (2:30)
  - _Principal_ - the account or user that is allowed access to the actions and resources specified in the statement
  - _Effect_ - "allow" or "deny"
  - _Action_ - list of permissions to allow or deny
  - _Resource_ - the bucket or object for which access applies to. Specified as Amazon Resource Name (ARN)
  - _SID_ - not required for S3. Generally used as a description of the policy statement
- __Which of the 5 elements above is not needed for User Policies?__ - "_Principal_ is not needed since the policy applies to the user
- __What's a conditional in a bucket policy?__ - similar to an `if` statement, that lets us do things like
  - allow a user to delete an object only if they use multi-factor authentication.
  - allow a user to "put" an object only if the object is uploaded with "public-read" canned ACL granted (see sample policy below).

```json
{
  "Version" : "2012-10-17"
  "Statement" : [
    {
      "Sid": "AllowPutObjects",
      "Principal" : {
        "AWS" : "arn:aws:iam::123456789012:user/Steve"
      },
      "Effect" : "Allow",
      "Action" : [
        "s3:PutObject"
      ],
      "Resource" : [
        "arn:aws:s3:::stevespublicbucket/*"
      ],
      "Condition" : {
        "StringEquals" : {
          "s3:x-amz-acl" : [
            "public-read"
          ]
        }
      }
    }
  ]
}
```

### Bucket & User Policies Lab (Part 2)

- __How prevent user from uploading non-encrypted file to S3?__ - (8:15) Use a bucket policy with a "condition" (in JSON) to require encryption

### Cross Account Access Using ACL's Lab

- __If Account A gives Account B access to its s3 buckets, how does Account A access the buckets?__ - (2:15) the buckets won't magically show up in Account A's s3 dashboard. Account A must access Account B's buckets by using the command line.
- __Using ACL, how can AWS Account A give AWS Account B's user access to Account A's s3 bucket?__ (entire video)
  1. (6:35) Account A gives _entire_ Account B "read" access using an ACL (since ACLs cant give access to just 1 user in Account B)
  1. (11:35) Account B gives its user "s3:ListBucket" access using a User Policy.

### Cross Account Access Using Bucket Policies Lab

- __Using bucket policies, how can AWS Account A give AWS Account B's user access to Account A's s3 bucket?__
  1. (2:00) Account A can give a _specific user_ in Account B "read" access using a bucket policy.
  1. (5:50 - 6:15) Account B gives its user "s3:ListBucket" access using a User Policy
- __How can Account A enforce user in Account B to upload files to Account A's s3 bucket with `bucket-owner-full-control`?__ - (2:35) You can use a "conditional" in the bucket policy (which is not possible with ACLs)

### Timed URL Lab

- __What's a timed URL?__ - (0:25) A URL (to a resource, such as a picture for example) that's only available for a certain amount of time (such as 1 day).
- __How create timed URL? What inputs are needed__ - (1:55) Can do it programatically (Python works). Need:
  1. AWS Security Credentials
  1. Bucket Name
  1. Object Name
  1. HTTP method (like GET or PUT)
  1. Expiration date & time.


# Chapter 4 - Security: Logging and Monitoring

### Monitoring S3 With CloudWatch

- __What is CloudWatch?__ - (0:20) A monitoring tool for AWS services. It can track metrics, collect log files, set alarms, etc.

### Monitoring S3 With CloudWatch Lab

- __What metrics come by default with s3 buckets?__ (1:00) storage metrics. They're updated once every 24 hours.
- __What metrics must you enable by paying?__ (1:20) "Requests", and "data transfer" metrics. They're updated in 1 minute intervals.
- __What are CloudWatch Dashboards?__ - (7:50) A simple place to put Cloudwatch graphs.
- __What kind of alerting/alarms are there for s3?__ - (9:35) For a metric, you can set an alarm on it that emails someone when the alarm is triggered. These are just standard CloudWatch alarms for s3.

### S3 Access Logging

- __What is "s3 access logging"__ (0:00) it tracks requests for access to a bucket. It records access time, requestor IP, object key name, operation type, response codes, etc.
- __Is bucket access logging enabled or disabled by default?__ (0:40) it's disabled by default
- __Are logs real-time?__ (1:05) No. Logs are periodically delivered. They're collected, consolidated, and then delivered.
- __Where are s3 access logs saved?__ (1:30) to a target bucket you specify. Using a different bucket is usually a good idea for organization purposes.
- __Why enable s3 access logging?__ (3:20) for security and audit reasons, to help understand AWS bill, or to learn about customers.
- __What should you do if logs build up over time?__ (4:05) use lifecycle rules to archive or delete logs after a period of time.
- __What is difference between Access logging and CloudTrail logging__ - (5:05) Access logging only shows requests within your bucket, while CloudTrail logging shows API calls in all of s3.

### S3 Access Logging Lab

- __What service can analyze logs?__ - (9:15) ElasticSearch.

### CloudTrail Logging

- __What is CloudTrail?__ - (0:00) tracks user activity by recording API calls made to your AWS account. Can record name of API call, identity of caller, time of API call, request parameters, and response elements returned by the AWS service.
- __Where are CloudTrail logs written?__ (4:35) an s3 bucket.

![CloudTrail To CloudWatch](images/CloudTrailToCloudWatch.png)

![CloudTrail vs S3 Access Logging](images/CloudTrailVsS3AccessLogging.png)

### CloudTrail Logging Lab

- __Is CloudTrail enabled by default?__ - (0:40) Yes, it's enabled by default in all regions, but it only saves 90 days worth of data in the CloudTrail console.
(at 3m in this video)
- __What's a CloudTrail "trail"?__ - (3:20) with the default CloudTrail settings, you can't do any event monitoring, can't trigger alerts, can't store data for more than 90 days, etc. You need to create a "trail" that will enable logging of CloudTrail to s3, and it will let you have advanced configuration options.
- __Why send CloudTrail logs to CloudWatch?__ - (9:50) it enables you to do basic filtering and monitoring on those logs.

### CloudTrail & CloudWatch Metrics Lab

- __Assume CloudTrail wrote access logs to s3 and CloudWatch. How can you set an alarm that detects bucket creation`?__
  1. (3:15) In `CloudWatch -> Logs`, for the CloudTrail log group, use a "filter pattern" of `{ ($.eventSource = s3.amazonaws.com) && ($.eventName = CreateBucket) }` to filter for logs that create a bucket.
  1. (4:10) On the metric filter you just created, click "Create Alarm". When the alarm triggers, it will send a notification to an SNS topic or email.

### CloudTrail & CloudWatch Events Lab

- __What are CloudWatch events?__ - (5:05) Lets you respond to state changes in your AWS resources.
  1. Create a "CloudWatch event" for
    - `Service Name: S3`
    - `Event Type: Bucket Level Operations`
    - `Specific operation(s): CreateBucket`
  1. Create a "CloudWatch Rule" that will tie the "CloudWatch Event" to a Lambda function (that will delete the created bucket).


# Chapter 5 - Security: Data Protection

### S3 Encryption

- __For s3, what are 2 places we should encrypt data?__ - (0:30) In transit, and at rest.
- __What is "Plaintext"?__ - unencrypted data.
- __What is "Ciphertext"?__ (1:50) the output of an encryption algorithm. It's your file in encrypted format. It's unreadable without knowledge of the algorithm and secret key.
- __What's an encryption algorithm?__ - (2:20) step-by-step instructions that precisely specify how plaintext is converted to ciphertext. A secret key is needed.
- __What are 2 types of encryption regarding keys?__ (3:10)
  1. Symmetric encryption - same key for encryption & decryption
  1. Asymmetric encryption - 2 keys. 1 public key for encryption. 1 private key for decryption.
- __Along with ciphertext data, what is stored in s3?__ - (3:45) An "encrypted data key"
- __What is an "encrypted data key"?__ (4:45) See image below. It's a "Symmetric Data Key" that's encrypted using a "Master Key".
- __How do you decrypt the data in s3?__ - (5:15)
  - Take the "Master Key" and use it to decrypt the "Encrypted Data Key" to get the "Symmetric Data Key"
  - Use the "Symmetric Data Key" on the "Ciphertext" to convert it back into "Plaintext"

![Encryption keys](/images/encryptionKeys.png)

- __What is server-side encryption (for s3)?__ - (7:00) s3 encrypts your data before writing it to disk, and decrypts it when your data is read from disk.
- __What are 3 types of server-side encryption?__ See images below
  1. _SSE-S3_ - Free. s3 handles all keys (Data Key, Master Key) for you using AES-256
    - Each object is encrypted with a unique data key.
    - The data key is encrypted by a master key
    - Master key is automatically rotated monthly
  1. _SSE-KMS_ - Not free. Keys are managed in Key Management Service (KMS) with a lot more control, outside of s3.
    - Not free.
    - Keys can be used across AWS Services (not just s3).
    - User creates, controls, and rotates master keys.
    - Uses AES-256
    - Every use of KMS is logged in CloudTrail
  1. _SSE-C_ - Free. You (the customer) provide the keys. Encryption is AES-256. You must use HTTPS to upload objects.
    1. The symmetric key is upload with the data
    1. s3 encrypts the data with the key, then deletes the key
    1. to decrypt the data, you must supply the key again

SSE-S3 or SSE-KMS:

![Server-side Encryption](images/serverSideEncryption.png)

SSE-C:

![SSE-C](images/ssec.png)

- __What is client-side encryption?__ (13:15) - It's when you encrypt your data on the client, then upload that encrypted data to s3. In this case, server-side encryption isn't needed.

You can keep track of your keys yourself:

![Client-side Encryption](images/clientSideEncryption.png)

Or you can have KMS supply the keys (you get the keys using AWS SDK to make an API call to KMS):

![Client-side Encryption](images/clientSideEncryptionWithKMS.png)

- __How encrypt data in transit?__ - (16:25)
  - (16:40) Use client-side encryption, so data is encrypted before upload
  - (17:00) Use SSL (HTTPS). You can actually enforce this using a bucket policy
- __What is difference between HTTP and HTTPS?__ - (17:10) HTTP is unencrypted. HTTPS is encrypted using assymetric encryption (public/private keys)

### S3 Encryption Lab

- __Are KMS keys global or region-specific?__ - (4:40) region specific.
- __Are s3 objects accessible by HTTP or HTTPS?__ - (16:50) both HTTP and HTTPS. You can enforce HTTPS access only by using a bucket policy. (Near end of course, it's mentioned static websites hosted on s3 are HTTP only, unless you use CloudFront for HTTPS)

### Versioning

- __Is s3 versioning enabled or disabled by default?__ - (0:45) disabled by default.
- __Why wouldn't you enable versioning?__ - (1:20) versioning can greatly increase storage costs
- __How can you mitigate the versioning cost?__ - (1:40) use lifecycle management to archive or delete old versions of data
- __How can you prevent a hacker in your account from turning versioning off and deleting all files?__ - (2:10) Use "Multi-factor Authentication (MFA) Delete", which can only be enabled by the root user. This only allows users that are Multi-factor authenticated to be able to:
  1. change the versioning state
  2. permanently delete an object version.
- __How are versions of an object stored in an s3 bucket (that has versioning enabled)?__ - (5:00) Each version of the object will have a unique "version id". The most recent version id is labeled as the current version.
- __How does deleting versioned objects work (in an s3 bucket with versioning enabled)?__ - (7:40) The object is not deleted. A "delete marker" with it's own "version id" is saved to s3.
- __How restore an old version of an object (in an s3 bucket with versioning enabled)?__ - (9:50)
  1. Can delete specific newer versions of an object (which is a permanent delete that cannot be undone) until you get the version you want, or
  1. Download the old version (`111111` in image below) and upload it (`131313` in image below). This will save the entire version history.

![Restore version](images/restoreVersion.png)

- __How can you stop versioning?__ - (11:20) Once turned on, versioning cannot be turned off, it can only be suspended. That means all previous versions of an object still remain in s3. All new objects are added to s3 with a "version id" of null.

![Delete Versioned Object](images/deleteVersionedObject.png)

### Cross Region Replication

- __What is cross-region replication in s3?__ - (0:10) When we copy our s3 data asynchronously to a bucket in another AWS region. A bucket-level feature.
- __Name 2 reasons to do cross-region replication__ - (0:50) Latency decrease, disaster recovery
- __What's 1 thing you can change for the copies that are made?__ (2:20) You can change the storage class in the destination bucket to use reduced redundancy.
- __When replicating data, does the bucket have to be in the same AWS account?__ - (2:50) No.
- __How does encryption affect cross-region replication?__ - (4:00) SSE-KMS encrypted files can't be transferred (since SSE-KMS is region-based). SSE-C files can't be transferred either since AWS doesn't know how to decrypt them.
- __When enabling cross-region replication, does it copy over everything to destination region?__ - (3:10) No. Any objects created before enabling cross-region replication are not actually replicated
- __What objects won't be replicated?__ - (3:40)
  - Objects that you don't have permission to read
  - objects encrypted with SSE-KMS (since these keys are in a per region basis)
  - objects encrypted with SSE-C (This is 1 reason we should use SSE-S3 encryption)
  - (5:30) objects that are already a replica
  - (6:00) [Subresource changes](https://docs.aws.amazon.com/AmazonS3/latest/dev/ObjectAndSoubResource.html) such as ACL lists are not replicated. This lets you have objects with different configurations
- __What property of the replicated objects are not replicated?__ - (5:15) Automated lifecycle actions
- __What are the requirements for cross-region replication?__ (6:20)
  - source & destination buckets must be version enabled
  - source & destination buckets must in different AWS regions
  - s3 must have permissions to replicate to the destination bucket.
  - can only replicate to a single destination bucket.
- __In a versioned, replicated bucket, when you delete an object in a source bucket, what happens to the object in the destination bucket?__ - (7:10) both source and destination get a "delete marker". The file is still there in both.
- __In a versioned, replicated bucket, when you delete a SPECIFIC VERSION of an object in a source bucket, what happens to the object in the destination bucket?__ - s3 does not delete it in destination to help you protect your data (disaster recovery). This is especially useful against hackers.


# Chapter 6 - Lifecycle Management

### Lifecycle Management

- __What is Lifecycle Management?__ - It's automating the image below, for s3 buckets or objects, to save money.

![Lifecycle Management](images/LifecycleManagement.png)

- __How is the lifecycle rule stored?__ - (3:10) it's stored as a bucket sub-resource.
- __What kind of buckets prohibit you from setting up lifecycle rules?__ - (5:45) MFA-enabled buckets
- __What is minimum days storage in certain storage classes before transitioning storage class?__ - (4:50)
  - 30+ days for STANDARD before transitioning to STANDARD_IA or ONEZONE_IA.
  - 30+ days for STANDARD_IA, ONEZONE_IA, INTELLIGENT_TIERING before transitioning to Glacier
- __What is minimum size of object that's allowed to be transitioned?__ - (5:25) Objects smaller than 128Kb cannot be transitioned to STANDARD_IA, ONEZONE_IA, or INTELLIGENT_TIERING.
- __If an object goes from s3 to Glacier, how do you view it?__ - (6:20) Transitioned objects remain in s3 and are NOT accessible by Amazon S3 Glacier Service. You have to access the objects through s3.
- __Why is it a bad idea to store lots of small files in Glacier?__
  - (8:15) 40Kb of extra storage is added in order to hold the object name, index detail and metadata. This can add to cost.
- __How can you transition non-current versions only?__ - (10:00) Use `NoncurrentVersionTransition` action. See image below:

![Lifecycle Actions](images/LifecycleActions.png)

- __How have versioning for 2 months__ - (14:20) Use "Versioning" combined with "Lifecycle Management"

### Lifecycle Management Lab

- __How can you mitigate the cost added from the 40Kb for each small file sent to Glacier?__ - one solution is to combine a bunch of the files into a single ZIP file before sending it to Glacier.

### Storage Class Analysis

- __What is Storage Class Analysis?__ - (0:20) a feature in s3 that, on a bucket/prefix/tags/etc. runs for 30+ days, then gives you detailed charts of how frequently that data is being accessed. It will also make a recommendation for you for which storage tier to use.
  - I prefer the new `INTELLIGENT_TIERING` storage class to automatically move data around storage classes for me, instead of just getting recommendations from "Storage Class Analysis" and spending time analyzing them.

# Chapter 7 - Event Notifications

### Event Notifications

- __What is an example of an Event Notification__ - Adding a new object to s3 is an example event. A notification configuration in the bucket can publish events to a destination.

![Event Notifications](images/EventNotifications.png)

- __Where are 2 places you can add trigger for s3 to send event notification to Lambda?__ - (8:20) in Lambda, or (12:40) in s3.


# Chapter 8 - Performance Optimization

### An Introduction to CloudFront

__What are 5 things that can serve as CloudFront origin?__ (7:05) s3 bucket, EC2 instance (that may have a web server), Elastic Load Balancer (that may front an EC2 instance or Route53 endpoint), Route 53 endpoint, or an external system.

### Transfer Acceleration

- __For selecting region for s3, it should be close to what 2 things?__ (0:45) proximity to users, and proximity to our AWS resources (such as our EC2 instances)
- __Is transfer acceleration same as CloudFront?__ - (5:00) No. It just uses the CloudFront network. So, there is no caching, and we're not creating a CloudFront distribution.

### Transfer Acceleration Lab

- __After enabling transfer acceleration, how do u use it?__ - (1:10) You're given a new (URL) endpoint that you can use for GET/PUT requests.

### Choosing an S3 Naming Scheme

- __Why does name of keys (filenames) in s3 matter?__ - (My summary) s3 hashes them and puts them in partitions. If all your filenames start the same (such as 2019-), then they will all get hashed to the same partition, and it will cause s3 to slow down.
- __What are 3 ways to solve above problem?__
  1. Can hash the file names. (i hate it, ruins filenames)
  1. Use a short hash to prepend the name (2nd least hated solution)
  1. Can use prefixes (folders) and put files in there. (my least hated solution)

### Optimizing S3 PUTS, GETS, & LISTS

- __How optimize PUTS, GETS, LISTS?__ - see image below.

![Performance Optimizations](images/PerformanceOptimizations.png)


# Chapter 9 - Website Hosting

### Static Website Hosting

- __How make DYNAMIC website with s3?__ - (~2:20) Static content on s3. API Gateway will call Lambda functions for dynamic stuff.
- __How do HTTPS with static website?__ - (6:00) Put CloudFront in front of website. Users will use CloudFront as https, and CloudFront will talk to origin using HTTP.

### Static Website Hosting Lab

- __How do redirects?__ - (7:10) in s3 console under "Static Website Hosting", we can write XML code for redirects.

### Cross-Origin Resource Sharing (CORS) Lab

- __How enable CORS for s3 bucket__ - (10:15) Simple. Go to s3 bucket that we're trying to get data from, click Permissions -> CORS, and add a bucket policy (with custom code).


# References

- Repo is based on [S3 Masterclass](https://acloud.guru/learn/aws-lambda) tutorial by "A Cloud Guru" - an amazing course.
  - The encryption part of this course was excellent!
  - Skip chapter 10 since it just shows how to use a specific non-AWS tool (CloudBerry).
