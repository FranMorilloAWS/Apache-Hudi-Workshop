# Self Paced Labs

If you've already signed up for Amazon Web Services (AWS) and have a user with administrator access, you can move on to Deploy CloudFormation Template.

If you haven't signed up for AWS account yet, or if you need assistance, follow the following instructions to set up your AWS account for the workshop.

## Sign Up for AWS

When you sign up for Amazon Web Services (AWS), your AWS account is automatically signed up for all services in AWS, including Lake Formation. You are charged only for the services that you use. If you have an AWS account already, skip to the next task. If you don't have an AWS account, use the following procedure to create one.

To create an AWS account
* Open [https://aws.amazon.com/](https://aws.amazon.com/) and then choose Create an AWS Account.

>[!Tip]
> If you previously signed in to the AWS Management Console using AWS account root user credentials, choose Sign in to a different account. If you previously signed in to the console using IAM credentials, choose Sign-in using root account credentials. Then choose Create a new AWS account.

* Follow the online instructions. Part of the sign-up procedure involves receiving a phone call and entering a verification code using the phone keypad. Note your AWS account number, because you'll need it for the next task.

## Create IAM User

Services in AWS, such as AWS Glue, require that you provide credentials when you access them, so that the service can determine whether you have permission to access its resources. The console requires your password. You can create access keys for your AWS account to access the command line interface or API. However, we don't recommend that you access AWS using the credentials for your AWS account; we recommend that you use AWS Identity and Access Management (IAM) instead. Create an IAM user, and then add the user to an IAM group with administrative permissions or grant this user administrative permissions. You can then access AWS using a special URL and the credentials for the IAM user.

If you signed up for AWS but have not created an IAM user for yourself, you can create one using the IAM console. If you aren't familiar with using the console, see Working with the AWS Management Console for an overview.

### To create an IAM user for yourself and add the user to an Administrators group
* Use your AWS account email address and password to sign in as the AWS account root user to the IAM console at [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/)

> [!Tip]
> We strongly recommend that you adhere to the best practice of using the Administrator IAM user below and securely lock away the root user credentials.

* Sign in as the root user only to perform a few account and service management tasks.

* In the navigation pane of the console, choose Users, and then choose Add user.

* For User name, type Administrator.

* Select the check box next to AWS Management Console access, select Custom password, and then type the new user's password in the text box. You can optionally select Require password reset to force the user to create a new password the next time the user signs in.

* Choose Next: Permissions.

* On the Set permissions page, choose Add user to group.

Choose **Create group**.

* In the Create group dialog box, for Group name type Administrators.

* For Filter policies, select the check box for AWS managed - job function.

* In the policy list, select the check box for AdministratorAccess. Then choose Create group.

* Back in the list of groups, select the check box for your new group. Choose Refresh if necessary to see the group in the list.

* Choose Next: Tags to add metadata to the user by attaching tags as key-value pairs.

* Choose Next: Review to see the list of group memberships to be added to the new user. When you are ready to proceed, choose Create user.

* You can use this same process to create more groups and users, and to give your users access to your AWS account resources.

To sign in as this new IAM user, sign out of the AWS console, then use the following URL, where your_aws_account_id is your AWS account number without the hyphens (for example, if your AWS account number is 1234-5678-9012, your AWS account ID is 123456789012):

Enter the IAM user name (not your email address) and password that you just created. When you're signed in, the navigation bar displays "your_user_name @ your_aws_account_id".

If you don't want the URL for your sign-in page to contain your AWS account ID, you can create an account alias. From the IAM console, choose Dashboard in the navigation pane. From the dashboard, choose Customize and enter an alias such as your company name.

To verify the sign-in link for IAM users for your account, open the IAM console and check under IAM users sign-in link on the dashboard.

