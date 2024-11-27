# Lab 4: S3 Access Control

_Duration of this lab: 2 periods_

## Pedagogical objectives:

In this lab, you will learn how to configure permissions by using AWS Identity and Access Management (IAM). You will learn to distinguish identity-based from resource-based policies. You will examine policies that control who has access to S3 buckets and the objects stored therein (access control policies).

After completing this lab, you should be able to do the following:

- Recognize how to use IAM identity-based policies and resource-based policies to define fine-grained access control to AWS services and resources.
- Describe how an IAM user can assume an IAM role to gain different access permissions to an AWS account.
- Explain how S3 bucket policies and IAM identity-based policies that are assigned to IAM users and roles affect what users can see or modify across different AWS services in the AWS Management Console.

#### Tasks

In this lab you will perform a number of tasks and document your progress in a lab report (PDF file). Each task specifies one or more deliverables to be produced. Collect all the deliverables in your lab report. Give the lab report a structure that mimics the structure of this document.

#### Prerequisites

This lab requires an AWS account with specially configured IAM users and S3 buckets that is provided to you.

- Group A, D, G, .. uses the IAM user `devuser1`;
- Group B, E, H, .. uses the IAM user `devuser2`;
- Group C, F, I, .. uses the IAM user `devuser3`; and so on.

In the following we refer to the IAM user without the number as `devuser`.

## Task 1: Accessing the console as an IAM user

In this task you will log into an AWS account as the IAM user named _devuser_, and you will explore the level of access this IAM user has to a few AWS services, including Amazon Elastic Compute Cloud (Amazon EC2), Amazon S3, and IAM.

1. Use the sign-in URL [https://851725581851.signin.aws.amazon.com/console](https://851725581851.signin.aws.amazon.com/console) to sign in as the IAM user named _devuser_:
    
    - For **Password**, enter
        - devuser1: `*Tp7_$T&Xw67K8*_`
        - devuser2: `KcN%=^m80E=rEXQ|`
        - devuser3: `4@WaL0I11C*^7C1(`
    
    The AWS Management Console appears.
    
    **Warning:** To avoid issues, ensure the selected region is `us-east-1` and do not change it.
    
2. Open the Amazon EC2 console. Choose **EC2 Dashboard**. Many _API Error_ messages display. This is expected.
    
3. Attempt some actions in the Amazon EC2 console:
    
    - In the left navigation pane, choose **Instances**.
        
        In the Instances list, a message displays _You are not authorized to perform this operation_.
        
    - Choose **Launch instance**.
        
    - Scroll down and choose the Key pair name drop down list.
        
        A message displays _You are not authorized to perform this operation_.
        
        Notice that Key pair name is a _required_ setting that must be configured if you want to log into an instance. This is just one of many indications that you will not be able to launch and connect to an EC2 instance successfully with the permissions that have been granted to you as the devuser.
        
    - Choose **Cancel**.
        
4. To explore what you can access in the Amazon S3 console, open it.
    
    Three buckets are listed. The bucket names are unique, but one bucket name contains _bucket1_, another contains _bucket2_, and the third contains _bucket3_.
    
    In the list of buckets, notice that the **Access** column displays the message _Insufficient permissions_ for all three buckets. This is expected.
    

## Task 2: Analyzing the identity-based policy applied to the IAM user

You have observed how the _devuser_ IAM user is unable to access certain information and actions in both the Amazon S3 console and Amazon EC2 console. In this task, you will look at the IAM policy details that apply to _devuser_ to understand why you can’t perform these actions.

1. Access the IAM console, and observe user and group membership settings:
    
    - Open the IAM console.
        
        On the IAM dashboard page, notice that you do not have permissions to view certain parts of the page. Both messages state _User: arn:aws:iam:::user/devuser is not authorized to perform: iam:GetAccountSummary on resource: *_. This is expected.
        
    - In the left navigation pane, choose **User groups**.
        
    - Choose the **DeveloperGroup** group name.
        
        On the **Users** tab, notice that _devuser_ is a member of this IAM group.
        
    - Choose the **Permissions** tab.
        
        Notice that an IAM policy named _DeveloperGroupPolicy_ is attached to this IAM group.
        
        **Note:** When a policy is attached to a group, the policy applies to any IAM users who are members of the group. Therefore, this policy currently governs your access to the console, because you are logged in as _devuser_, who is a member of this IAM group.
        
2. Review the IAM policy details:
    
    - On the lower portion of the page, choose the plus icon to the left of **DeveloperGroupPolicy** to display the policy details.
        
    - Review the JSON policy details, and recall the level of access that you had for Amazon EC2 and Amazon S3 in the previous task.
        
        - Notice that the policy does not allow any Amazon EC2 actions.
        - Notice the IAM actions that the policy allows. When you accessed the IAM dashboard, you saw a message that stated that you did not have _iam:GetAccountSummary_ authorization. That action is not permitted in this policy document. However, many read-level IAM permissions are granted. For example, you are able to review the details for this policy.
        - Notice the Amazon S3 actions that the policy allows. No object-related actions are granted, but some actions related to buckets are allowed.
3. Save the policy to a file on your computer.
    

## Task 3: Attempting write-level access to AWS services

Any action that you attempt when you interact with an AWS service is an API call, whether you are using the console, AWS Command Line Interface (AWS CLI), or AWS software development kits (SDKs). All attempted API calls are recorded in the AWS CloudTrail event logs.

In this task, you will attempt to make two API calls that require _write-level_ access within Amazon S3. The first action is to create an S3 bucket, and the second action is to upload an object to that bucket. After you attempt the two tasks, you will again analyze the policy attached to the IAM group to analyze why you could or could not perform the specific API calls.

1. Attempt to create an S3 bucket:
    
    - For **Bucket name**, following the lab naming convention, enter a name similar to `testbucket-grf-xxxx`, where xxxx is a random four-digit number to make the bucket name globally unique.
        
        **Note:** By default, new buckets, access points, and objects don’t allow public access. Diving deeper into this goes beyond the scope of this lab, but it’s important to note.
        
    - For **AWS Region**, choose **US East (N. Virginia) us-east-1**.
        
    - Review the settings, and then choose Create bucket at the bottom of the page.
        
        You successfully created an S3 bucket.
        
2. Access the bucket, and attempt to upload an object. A message displays _Upload failed_.
    
    - On the **Files and folders** tab on the lower part of the page, in the **Error** column, choose the **Access Denied** link.
        
        The message states _You don’t have permissions to upload files and folders_.
        
3. Review the policy details for Amazon S3 access:
    
    - Return to the DeveloperGroupPolicy that you saved on your computer.
        
    - Review the policy details to understand why you were able to create an S3 bucket but couldn’t upload objects to it.
        
        **Tip:** The _Service Authorization Reference_ document provides a list of actions that each AWS service supports. For information about Amazon S3 actions, open the [IAM documentation](https://docs.aws.amazon.com/iam/) page, and then open the _Service Authorization Reference_ document. In the left navigation pane, expand **Actions, resources, and condition keys**, and then choose **Amazon S3**. In the **Actions defined by Amazon S3** section, the table lists every possible Amazon S3 action that can be granted or denied, along with a description of the action.
        

Deliverables:

- Explain why you were able to create an S3 bucket but couldn’t upload objects to it.

## Task 4: Assuming an IAM role and reviewing a resource-based policy

In this task, you will try to access _bucket1_ and _bucket2_ while logged in as the _devuser_ IAM user. You will also try to access the buckets by using a role that was preconfigured as part of the lab setup.

1. Try to download an object from the buckets that were created during lab setup:
    
    - In the Amazon S3 console, choose the bucket name that contains **bucket1**.
        
    - Select **Image2.jpg**, and then choose **Download**.
        
        An AccessDenied error page appears.
        
    - Try to download the **Image1.jpg** file from _bucket2_.
        
        You receive the same error.
        
        **Analysis:** As shown in the following diagram, with the permissions that are granted through membership in the _DeveloperGroup_, you were able to create a new bucket. However, you cannot access objects in _bucket1_ or _bucket2_.
        
2. Assume the _BucketsAccessRole_ IAM role in the console:
    
    - In the upper-right corner of the page, choose **devuser**, and then choose **Switch role**.
        
    - If the Switch role page appears, choose **Switch Role.**
        
    - Configure the following:
        
        - **Account:** Enter the **AccountID** value 851725581851.
            
        - **Role:** Enter `BucketsAccessRole`
            
        - **Display Name:** Leave this field blank.
            
        - Choose **Switch Role**.
            
            You successfully assumed the IAM role named _BucketsAccessRole_, which was preconfigured for this lab.
            
            **Tip:** You can tell that you switched into the role by looking at the upper-right corner of the console. Notice that **BucketsAccessRole** is displayed where **devuser** was previously displayed.
            
3. Try to download an object from Amazon S3 again:
    
    - In the Amazon S3 console, choose the bucket name that contains **bucket1**.
        
    - Select **Image2.jpg**, and then choose **Download**.
        
    - Open the file to verify that the file downloaded.
        
        **Analysis:** The download was successful, which means that the policy or policies applied to the _BucketsAccessRole_ allow the _s3:GetObject_ action on _bucket1_.
        
4. Test IAM access with the _BucketsAccessRole_:
    
    - Navigate to the IAM console.
        
        **Note:** By changing roles, the permissions that you have to interact with different AWS services have changed. As you navigate the IAM console, you will see new error messages that state that you are not authorized.
        
    - In the left navigation pane, choose **User groups**.
        
        **Analysis:** An error message displays. You no longer have permissions to view the IAM user groups page because _BucketsAccessRole_ does not have the _iam:ListGroups_ action applied to it.
        
5. Assume the _devuser_ role again, and test access to the user groups page:
    
    - In the upper-right corner of the page, choose **BucketsAccessRole**, and then choose **Switch back**.
        
    - In the left navigation pane, choose **User groups** again.
        
        **Analysis:** Now that you unassumed the _BucketsAccessRole_, you have the permissions that are assigned to the _devuser_ IAM user (through this user’s membership in the _DeveloperGroup_). You are able to view the user groups page again.
        
6. Analyze the IAM policy that is associated with the _BucketsAccessRole_:
    
    - In the left navigation pane, choose **Roles**.
        
    - Search for `BucketsAccessRole` and choose the role name when it appears.
        
    - On the **Permissions** tab, in the **Permissions policies** section, you can see all of the IAM policies that are applied to _BucketsAccessRole_.
        
    - Choose the arrow to the left of **ListAllBucketsPolicy**.
        
    - To view the policy in JSON format, choose **{ } JSON**.
        
        This policy grants the same _s3:ListAllMyBuckets_ action to every resource. This permission allows you to see all S3 buckets when you assume _BucketsAccessRole_.
        
    - Choose the arrow to the left of **GrantBucket1Access**.
        
        To view the JSON document details, choose **{ } JSON** if necessary.
        
        **Analysis:** This policy allows the _s3:GetObject_, _s3:ListObjects_, and _s3:ListBucket_ actions. Notice that this policy does _not_ grant _s3:PutObject_ access. The allowed actions are only granted for specific resources, _bucket1_ and all objects within _bucket1_ (as indicated by **/***). The asterisk (*) is a wildcard character, which indicates that this would match any value.
        
        Because of this policy, when you assumed the _BucketsAccessRole_, you could see and download objects from _bucket1_.
        
7. Save a copy of the _GrantBucket1Access_ policy to your computer.
    
8. Complete your analysis of the _BucketsAccessRole_ details:
    
    - Scroll back up the page, and choose the **Trust relationships** tab.
        
        Notice that the _devuser_ IAM user in this AWS account is listed as a trusted entity that can assume this role.
        
        Notice that the account number that appears in the upper-right corner of the console (after **devuser**) matches the account number in the **Trusted entities** list (without the dashes).
        
        **Note:** The AWS Security Token Service (AWS STS) will provide temporary credentials to any trusted entity that requests to assume the role. This trust policy trusts an IAM user in the same account. However, a trust policy could be configured to trust one or more principals, even in other AWS accounts. Examples of other principals are AWS services, IAM roles, and IAM users.
        
9. Assume the _BucketsAccessRole_, and try to upload an image to _bucket2_:
    
    - To assume the _BucketsAccessRole_ again, in the upper-right corner of the page, choose **devuser**.
        
    - Under **Role history**, choose **BucketsAccessRole**.
        
    - Navigate to the Amazon S3 console.
        
    - Choose the bucket name that contains **bucket2**.
        
        On your local machine rename the Image2.jpg file and add the name of your group, similar to Image2-GrF.jpg. Notice that this bucket does not yet have that file.
        
    - Choose **Upload**, and then choose **Add files**.
        
    - Browse to and choose the **Image2-GrF.jpg** file that you downloaded earlier from _bucket1_.
        
    - Choose **Upload**.
        
        The file uploads successfully.
        
    - Choose **Close**.
        
        **Analysis:** After assuming the _BucketsAccessRole_, you successfully accessed _bucket1_ to download an object. You then uploaded the same object to _bucket2_.
        
        After inspecting the policies attached to the _BucketsAccessRole_, you know that the Amazon S3 permissions that were granted to that role were limited to _bucket1_, as shown in the following diagram.
        
        ![The GrantBucket1Access policy is attached to the BucketsAccessRole. When the role is assumed, the user can access objects in bucket1 and upload objects to bucket2.](https://cyberlearn.hes-so.ch/pluginfile.php/2149887/mod_assign/intro/role-assumed.png)
        
        The GrantBucket1Access policy is attached to the BucketsAccessRole. When the role is assumed, the user can access objects in bucket1 and upload objects to bucket2.
        
    - So, how were you just now able to upload an object to _bucket2_? The reason will become clear in the next task.
        

## Task 5: Understanding resource-based policies

In this task, you will inspect the bucket policy that is associated with _bucket2_.

1. Observe the details of the bucket policy that is applied to _bucket2_:
    
    - On the details page for _bucket2_, choose the **Permissions** tab.
        
    - In the **Bucket policy** section, review the policy that is applied to _bucket2_.
        
        The policy has two statements.
        
        The first statement ID (SID) is _S3Write_. The principal is the _BucketsAccessRole_ IAM role that you assumed. This role is allowed to call the actions _s3:GetObject_ and _s3:PutObject_ on the resource, which is _bucket2_.
        
        The second SID is _ListBucket_. The principal is _BucketsAccessRole_. This role is allowed to call the action _s3:ListBucket_ on the resource, which is _bucket2_.
        
        **Analysis:** You should now have a better understanding of how resource-based policies (such as S3 bucket policies) and role-based policies (policies associated with IAM roles) can interact and be used together.
        
        In this lab, the _role-based policies_ attached to the _BucketsAccessRole_ IAM role granted _s3:GetObject_ and _s3:ListBucket_ access to _bucket1_ and the objects in it. These role-based policies did not explicitly allow access to _bucket2_; however, they also did not explicitly deny access.
        
        The following diagram shows how the policies that were applied to the IAM user, IAM role, and bucket determined what actions you were able to perform.
        BucketsAccessRole has access to bucket1 because of a role-based policy and has access to bucket2 because of a resource-based policy.
        
        Then, while still assuming the _BucketsAccessRole_, you tried to upload an object to _bucket2_, and you were able to do it. That seemed strange based on the IAM policies that you reviewed. However, after you reviewed the _resource-based policy_ (in this case, a bucket policy) that was attached to the bucket, your access made sense. That bucket policy grants access, including the _s3:PutObject_ action, to _bucket2_ to the _BucketsAccessRole_ principal.
        

## Task 6: Find a way to upload an object to _bucket3_

Your objective for this challenge task is to figure out a way to upload the Image2.jpg file to _bucket3_.

1. Try to upload the file as _devuser_ with no role assumed:
    
    - Unassume the _BucketsAccessRole_.
        
    - Attempt to upload Image2.jpg, which you downloaded from _bucket1_ earlier in this lab, to _bucket3_.
        
        The upload fails.
        
    - Check whether a bucket policy is associated with _bucket3_. Maybe that will give you some indication about how to accomplish this task.
        
        You can’t view the bucket policy.
        
2. Assume the _BucketsAccessRole_, and try the actions from the previous step:
    
    - Can you upload a file to _bucket3_?
    - Can you view the bucket policy now? Review the bucket policy details. Do you have an idea for how you can upload Image2.jpg to _bucket3_?

## Task 7: Design and implement permission policies for S3

In this task we assume you are working for the company Acme Data Products, a startup offering data analysis services. You will create IAM roles and IAM policies to manage the access to the data.

For this task you use your usual IAM account. It has been configured to allow you to create and modify roles and policies. Be responsible and careful in your use of IAM!

The data Acme stores in AWS falls into one of three categories:

- _internal:_ Data that has been gathered from various places, some of it can be a copy of public data, that has not yet been processed and therefore does not have particular value and does not need particular protection, but Acme does not want this data to be publicly accessible. All Acme employees can access this data, but no outsider.
- _private:_ This category contains data that either needs to be protected because of regulations, for example personal data that falls under GDPR, or because it is the result of Acme’s proprietary analysis algorithms and Machine Learning models and therefore valuable. Only certain Acme employees with “need to know” can access this data.
- _public:_ This is data that Acme wants to make publicly available, for example for marketing purposes. Everyone can access this data.

Create a bucket that at the top level has three folders for internal, private, and public data.

Create the following IAM roles:

- An _AcmeStaff_ role that has read access to internal and public data.
- An _AcmeDataScientist_ role that has read and write access to all data.
- An _AcmeDataIngester_ role that has write access to internal and private data.

Create customer-managed policies and attach them to the roles.

Configure policies that allow anybody to read public data.

You will find the list of all S3 actions in the AWS documentation: [Actions, resources, and condition keys for AWS services](https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html).

Naming conventions: Use the lab naming conventions and name

- the bucket similar to `acmedata-grf`
- the policies similar to `AcmeDataGrFFullAccess`
- the roles similar to `AcmeGrFStaff`

In your design apply the principle of least privilege.

**Hint:** When writing policies, the JSON format is not very practical. YAML is much easier to edit in a code editor. The [yq tool](https://mikefarah.gitbook.io/yq/) wrangles YAML files and converts to/from JSON.

Convert JSON to YAML as pipeline:

```
cat file.json | yq -P . - 
```

Convert YAML to JSON as pipeline:

```
cat file.yaml | yq -o=json eval
```

Btw. `yq` is also useful to convert the JSON output of the AWS CLI into YAML.

## Task 8: Simulating policy evaluation

IAM policies are complex, and as an engineer implementing them, you will rarely succeed in setting up the correct permissions on the first attempt. How did you develop and troubleshoot new policies in the previous tasks? How do you ensure that users have neither less nor more permission rights than what is adequate when developing IAM policies? In this task you will use a policy simulator provided by AWS to test policies.

- The policy simulator is available at [https://policysim.aws.amazon.com/](https://policysim.aws.amazon.com/).
- Documentation is available at [Testing IAM policies with the IAM policy simulator](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html).

In the following we will test a policy that was used for the first tasks of this lab. The _devuser_ account is in a user group that has a policy called _DeveloperGroupPolicy_ attached to it.

1. Navigate to the policy simulator. On the top you see a drop-down menu for the mode. The policy simulator has two modes:
    
    - The first mode is for testing identity-based policies that are already defined and are already attached to a user, user group or role: **Mode: Existing Policies**.
        
    - The second mode is for testing arbitrary policies that you have to paste into the policy simulator: **Mode: New Policy**.
        
    
    Select **Mode: Existing Policies**. On the left pane titled **Users, Groups, and Roles** select the IAM user **devuser1**. Navigate to the single policy that is attached to this user.
    
    Click on the name of the policy to see the content of the policy in JSON.
    
2. Now we construct several requests which consist of a _principal_, an _action_ and a _resource_. For each request, we want the policy simulator to tell us if this request will be _allowed_ or _denied_.
    
    The principal is already implicitly set to _devuser1_ because we selected it in the first step.
    
    Next we define several requests with different actions. In the pane titled `Policy Simulator` select in the first drop-down menu the service **S3**, then select in the second drop-down menu the actions **ListBucket** and **DeleteBucket**. You should see two requests appear in the pane **Action Settings and Results**. They correspond to the two selected actions.
    
3. We can now specify a resource for each requests. For that you click on the request to expand it and a field appears where you can specify a bucket name. The field has a default value of __*__ which is the wildcard for any bucket resource. Leave that value, so that we test if _in general_, the user _devuser1_ is allowed to list or delete buckets.
    
4. Click on **Run Simulation**. On the right of the requests you should see the result of the simulation: listing buckets is **allowed** while deleting buckets is **denied**.
    
    When the request is denied, the policy simulator shows whether it was denied because no policy matched the request _(implicitly denied)_, or because there was an explicit _deny_ effect in a matching policy.
    

Now we apply the two requests we created to a different policy to see the outcome of that policy.

1. In the left pane navigate back to **Users, Groups, and Roles**. Select **Roles** and select the role **OtherBucketAccessRole**. Look at the content of the **GrantBucket1Access** policy. This policy gives access to a specific bucket.
    
2. Re-run the simulation with the existing two requests. Both should be denied.
    
    Modify the _ListBucket_ request and specify the bucket mentioned in the policy so that it is _allowed_.
    

Deliverables:

- How did you modify the _ListBucket_ request so that it is _allowed_ for the _OtherBucketAccessRole_?
    
- Create a new policy, by copying and modifying the _GrantBucket1Access_ policy, that explicitly _denies_ access to a bucket. Using the _New Policy_ mode of the simulator, create a request that will be _denied_ explicitely. Copy the policy into the report and make a screenshot of the simulator showing the explicit deny of the request.