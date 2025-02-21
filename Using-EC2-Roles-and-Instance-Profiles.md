# Using EC2 Roles and Instance Profiles

## Introduction

AWS Identity and Access Management (IAM) roles for Amazon Elastic Compute Cloud (EC2) provide the ability to grant instances temporary credentials.  These temporary credentials can then be used by hosted applications to access permissions configured within the role. IAM roles eliminate the need for managing credentials, help mitigate long-term security risks, and simplify permissions management. Prerequisites for this lab include understanding how to log in to and use the AWS Management Console, EC2 basics (including how to launch an instance), IAM basics (including users, policies, and roles), and how to use the AWS CLI.

>**Note:** When connecting to the bastion host and the web server, do so independently of each other. The bastion host is used for interacting with AWS services via the CLI.

## Solution

Log in to the AWS console using the `cloud_user` credentials provided. Once inside the AWS account, make sure you are using `us-east-1` (N. Virginia) as the selected region.

**Hint:** *When copying and pasting code into Vim from the lab guide, first enter `:set paste` (and then `i` to enter insert mode) to avoid adding unnecessary spaces and hashes.*

### Create a Trust Policy and Role Using the AWS CLI

#### Obtain the `labreferences.txt` File

 1. Navigate to S3.

 1. From the list of buckets, open the one that contains the text _s3bucketlookupfiles_ in the middle of its name.

 1. Select the `labreferences.txt` file.

 1. In the pane that appears, click **Download**.

 1. Open the `labreferences.txt` file, as we will need to reference it throughout the lab.

#### Log in to Bastion Host and Set the AWS CLI Region and Output Type

 1. Navigate to **EC2** > **Instances**.

 1. Cop the public IP of the *Bastion Host* instance.

 1. Open a terminal, and log in to the bastion host via SSH:

    ```
    ssh cloud_user@<BASTION_HOST_PUBLIC_IP>
    ```

 1. Enter the password provided for it on the lab page.

 1. Run the following command:

    ```
    [cloud_user@bastion]$ aws configure
    ```

 1. Press **Enter** twice to leave the *AWS Access Key ID* and *AWS Secret Access Key* blank.

 1. Enter `us-east-1` as the default region name.

 1. Enter `json` as the default output format.

#### Create IAM Trust Policy for an EC2 Role

 1. Create a file called `trust_policy_ec2.json`

    ```
    [cloud_user@bastion]$ vim trust_policy_ec2.json
    ```

 1.  To avoid adding unnecessary spaces or hashes, type `:set paste` and then `i` to enter `insert` mode.

 1. Paste in the following content:

    ```
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {"Service": "ec2.amazonaws.com"},
          "Action": "sts:AssumeRole"
        }
      ]
    }
    ```

 1. Save and quit the file by pressing **Escape** followed by `:wq!`.

#### Create the `DEV` IAM Role

 1. Run the following AWS CLI command:

    ```
    [cloud_user@bastion]$ aws iam create-role --role-name DEV_ROLE --assume-role-policy-document file://trust_policy_ec2.json
    ```

#### Create an IAM Policy Defining Read-Only Access Permissions to an S3 Bucket

 1. Create a file called `dev_s3_read_access.json`:

    ```
    [cloud_user@bastion]$ vim dev_s3_read_access.json
    ```

 1.  To avoid adding unnecessary spaces or hashes, type `:set paste` and then `i` to enter `insert` mode.

 1. Enter the following content, replacing `<DEV_S3_BUCKET_NAME>` with the bucket name provided in the `labreferences.txt` file:

    ```
    {
        "Version": "2012-10-17",
        "Statement": [
            {
              "Sid": "AllowUserToSeeBucketListInTheConsole",
              "Action": ["s3:ListAllMyBuckets", "s3:GetBucketLocation"],
              "Effect": "Allow",
              "Resource": ["arn:aws:s3:::*"]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:Get*",
                    "s3:List*"
                ],
                "Resource": [
                    "arn:aws:s3:::<DEV_S3_BUCKET_NAME>/*",
                    "arn:aws:s3:::<DEV_S3_BUCKET_NAME>"
                ]
            }
        ]
    }
    ```

 1. Save and quit the file by pressing **Escape** followed by `:wq!`.

 1. Create the managed policy called `DevS3ReadAccess`:

    ```
    [cloud_user@bastion]$ aws iam create-policy --policy-name DevS3ReadAccess --policy-document file://dev_s3_read_access.json
    ```

 1. Copy the policy ARN in the output, and paste it into the `labreferences.txt` file — we'll need it in a minute.

### Create Instance Profile and Attach Role to an EC2 Instance

#### Attach Managed Policy to Role

 1. Attach the managed policy to the role, replacing `<DevS3ReadAccess_POLICY_ARN>` with the ARN you just copied:

    ```
    [cloud_user@bastion]$ aws iam attach-role-policy --role-name DEV_ROLE --policy-arn "<DevS3ReadAccess_POLICY_ARN>"
    ```

 1. Verify the managed policy was attached:

    ```
    [cloud_user@bastion]$ aws iam list-attached-role-policies --role-name DEV_ROLE
    ```

#### Create the Instance Profile and Add the `DEV_ROLE` via the AWS CLI

 1. Create instance profile named `DEV_PROFILE`:

    ```
    [cloud_user@bastion]$ aws iam create-instance-profile --instance-profile-name DEV_PROFILE
    ```

 1. Add role to the `DEV_PROFILE` called `DEV_ROLE`:

    ```
    [cloud_user@bastion]$ aws iam add-role-to-instance-profile --instance-profile-name DEV_PROFILE --role-name DEV_ROLE
    ```

 1. Verify the configuration:

    ```
    [cloud_user@bastion]$ aws iam get-instance-profile --instance-profile-name DEV_PROFILE
    ```

#### Attach the `DEV_PROFILE` Role to an Instance

 1. In the AWS console, navigate to **EC2** > **Instances**.

 1. Copy the instance ID of the instance named *Web Server* instance and paste it into the `labreferences.txt` file — we'll need it in a second.

 1. In the terminal, attach the `DEV_PROFILE` to an EC2 instance, replacing `<LAB_WEB_SERVER_INSTANCE_ID>` with the *Web Server* instance ID you just copied:

    ```
    [cloud_user@bastion]$ aws ec2 associate-iam-instance-profile --instance-id <LAB_WEB_SERVER_INSTANCE_ID> --iam-instance-profile Name="DEV_PROFILE"
    ```

 1. Verify the configuration (be sure to replace `<LAB_WEB_SERVER_INSTANCE_ID>` with the *Web Server* instance ID again):

    ```
    [cloud_user@bastion]$ aws ec2 describe-instances --instance-ids <LAB_WEB_SERVER_INSTANCE_ID>
    ```

    This command's output should show that this instance is using `DEV_PROFILE` as an `IamInstanceProfile`. Verify this by locating the `IamInstanceProfile` section in the output, and look below to make sure the `"Arn"` ends in `/DEV_PROFILE`.

### Test S3 Permissions via the AWS CLI

 1. In the AWS console, copy the public IP of the *Web Server* instance.

 1. Open a new terminal.

 1. Log in to the web server instance via SSH:

    ```
    ssh cloud_user@<WEB_SERVER_PUBLIC_IP>
    ```

 1. Use the same password for the bastion host provided on the lab page.

 1. Verify the instance is assuming the `DEV_ROLE` role:

    ```
    [cloud_user@webserver]$ aws sts get-caller-identity
    ```

    We should see `DEV_ROLE` in the `Arn`.

 1. List the buckets in the account:

    ```
    [cloud_user@webserver]$ aws s3 ls
    ```

    Copy the entire name (starting with `cfst`) of the bucket with `s3bucketdev` in its name.

 1. Attempt to view the files in the `s3bucketdev-` bucket, replacing `<s3bucketdev-123>` with the bucket name you just copied:

    ```
    [cloud_user@webserver]$ aws s3 ls s3://<s3bucketdev-123>
    ```

    We should see a list of files.

### Create an IAM Policy and Role Using the AWS Management Console

#### Create Policy

 1. In the AWS console, navigate to **IAM** > **Policies**.

 1. Click **Create policy**.

 1. Click the **JSON** tab.

 1. Paste the following text as the policy, replacing `<PROD_S3_BUCKET_NAME>` with the bucket name provided in the `labreferences.txt` file:

    ```
    {
        "Version": "2012-10-17",
        "Statement": [
            {
              "Sid": "AllowUserToSeeBucketListInTheConsole",
              "Action": ["s3:ListAllMyBuckets", "s3:GetBucketLocation"],
              "Effect": "Allow",
              "Resource": ["arn:aws:s3:::*"]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:Get*",
                    "s3:List*"
                ],
                "Resource": [
                    "arn:aws:s3:::<PROD_S3_BUCKET_NAME>/*",
                    "arn:aws:s3:::<PROD_S3_BUCKET_NAME>"
                ]
            }
        ]
    }
    ```

 1. Click **Review policy**.

 1. Enter **ProdS3ReadAccess** as the policy name.

 1. Click **Create policy**.

#### Create Role

 1. Click **Roles** in the left-hand menu.

 1. Click **Create role**.

 1. Select **EC2** as the service that will use this role.

 1. Click **Next: Permissions**.

 1. In the search box, enter **ProdS3ReadAccess**.

 1. Click the checkbox to select **ProdS3ReadAccess**.

 1. Click **Next: Tags**.

 1. Click **Next: Review**.

 1. Give it a *Role name* of **PROD_ROLE**.

 1. Click **Create role**.

### Attach IAM Role to an EC2 Instance Using the AWS Management Console

 1. Navigate to **EC2** > **Instances**.

 1. Select the **Web Server** instance.

 1. Click **Actions** > **Instance Settings** > **Modify IAM Role**.

 1. Select **PROD_ROLE** as the IAM role.

 1. Click **Save**.

#### Test the Configuration

 1. Open the existing terminal connected to the *Web Server* instance. (You may need to reconnect if you've been disconnected.)

 1. Determine the identity currently being used:

    ```
    [cloud_user@webserver]$ aws sts get-caller-identity
    ```

    This time, we should see `PROD_ROLE` in the `Arn`.

 1. List the buckets:

    ```
    [cloud_user@webserver]$ aws s3 ls
    ```

 1. Copy the entire name (starting with `cfst`) of the bucket with `s3bucketprod` in its name.

 1. Attempt to view the files in the `s3bucketprod-` bucket, replacing `<s3bucketprod-123>` with the bucket name you just copied:

    ```
    [cloud_user@webserver]$ aws s3 ls s3://<s3bucketprod-123>
    ```

    It should list the files.

 1. In the `aws s3 ls` command output, copy the entire name (starting with `cfst`) of the bucket with `s3bucketsecret` in its name.

 1. Attempt to view the files in the `<s3bucketsecret-123>` bucket, replacing `<s3bucketsecret-123>` with the bucket name you just copied:

    ```
    [cloud_user@webserver]$ aws s3 ls s3://<s3bucketsecret-123>
    ```

    This time, our access will be denied — which means our configuration is properly set up.

## Conclusion

Congratulations on successfully completing this hands-on lab!
