# Migrate an Existing Terraform Project into Terraform Cloud

## Introduction 

In this hands-on lab, we will take an existing Terraform configuration and convert it to run the code using Terraform Cloud. We will use the CLI-driven method to deploy our resources to AWS with Terraform Cloud doing the bulk of the work.

## Solution 

1. Open a terminal session, and log in to the lab server using the credentials provided:

    ```
    ssh cloud_user@<PUBLIC_IP_ADDRESS>
    ```

1. Log in to your GitHub account in the browser. 

1. Log in to your Terraform Cloud account in the browser: `https://app.terraform.io/`

1. In an incognito or private browser window, log in to the live AWS environment using the credentials provided. Make sure you're in the N. Virginia (`us-east-1`) region throughout the lab.

### Prepare the Environment

1. In the terminal, clone the lab repo: 

    ```
    git clone https://github.com/ACloudGuru-Resources/content-introduction-terraform-cloud-lab.git
    ```

1. Make sure you see the `content-introduction-terraform-cloud-lab` directory: 

    ```
    ls 
    ```

1. Change directory: 

    ```
    cd content-introduction-terraform-cloud-lab
    ```

1. List the files in the directory: 

    ```
    ls 
    ```

    You should see `main.tf` and `README.md`.

1. Open `main.tf`: 

    ```
    vim main.tf
    ```

1. In the Terraform Cloud console, click your avatar and then **User Settings**. 
1. In the left-hand menu, click **Organizations**.
1. Click **Create new organization**.
1. For **Organization name**, enter *terraform-guru* (followed by some characters to make it globally unique).
1. Click **Create organization**.
1. On the **Create a new Workspace** screen, select **CLI-driven workflow**.
1. For **Workspace Name**, enter *terraform-cloud-guru*.
1. Click **Create workspace**.
1. In the left-hand menu, click **Workspaces**.
1. In the left-hand menu, click **Settings**.
1. In the left-hand menu, click **Variable sets**.
1. Click **Create variable set**, and set the following values: 
    * **Name:** Enter *AWS Credentials*.
    * **Workspaces:** Select **Apply to specific workspaces**.
    * **Search for workspaces:** Search for and select **terraform-cloud-guru**.
1. Click **Add variable**, and set the following values: 
    * **Select variable category:** Select *Environment variable*.
    * **Key:** Enter *AWS_ACCESS_KEY_ID*.
    * **Value:** 
        1. In the AWS Management Console, navigate to **IAM** > **Users**.
        1. Click **cloud_user**.
        1. Click **Security credentials**.
        1. Click **Create access key**.
        1. Once it's created, copy the access key ID and secret access key, and paste them into a text file. 
        1. Back on the variable creation page in Terraform Cloud, paste the access key ID into the **Value** field.
        1. Check **Sensitive**.
1. Click **Add variable**.
1. Click **Add variable** again, and set the following values: 
    * **Select variable category:** Select *Environment variable*.
    * **Key:** Enter *AWS_SECRET_ACCESS_KEY*.
    * **Value:** Paste the secret access key ID you copied.
    * **Sensitive:** Check the box.
1. Click **Add variable**.
1. Click **Create variable set**.
1. In the AWS Management Console, navigate to **VPC** > **Subnets**.
1. Copy the listed subnet ID, and paste it into a text file.
1. Back in the terminal, still in the `main.tf` file, replace `<SUBNET_ID>` with the one you just copied.  
1. Save and exit the file by pressing **Escape** followed by `:wq!`.
1. In the Terraform Cloud console, click your user profile and select **User Settings**.
1. In the left-hand menu, click **Tokens**.
1. Click **Create an API token**.
1. For **Description**, enter *terraform-cloud*.
1. Click **Create API token**.
1. Copy the API token, and paste it into a text file. 
1. In the terminal, enter: 

    ```
    terraform login
    ```

1. At the prompt, enter `yes`.
1. At the prompt asking for the API token, the API token you just copied. 
    * You should then see a welcome message. 

### Migrate the Existing Terraform Project into Terraform Cloud

1. Reopen `main.tf`: 

    ```
    vim main.tf
    ```

1. In the Terraform Cloud console, click **Done** in the API token dialog. 
1. Click the Terraform logo in the upper left. 
1. Click the **terraform-guru** organization.
1. Click the **terraform-cloud-guru** workspace. 
1. Under **CLI-driven runs**, copy everything in the **Example code** box except the first and last lines, which would be: 

    ```
    cloud {
        organization = "terraform-guru"

        workspaces {
          name = "terraform-cloud-guru"
        }
      }
    ```

1. In the terminal, beneath `terraform {`, indent two spaces and paste in that code.
1. Copy the AMI ID in the `variable "ami"` block. 
1. In the Terraform Cloud console, in the left-hand menu, click **Variables**.
1. Click **Add variable**, and set the following values: 
    * **Select variable category:** Select *Terraform variable*.
    * **Key:** Enter *instance_count*.
    * **Value:** Enter *2*.
1. Click **Save variable**.
1. Click **Add variable**, and set the following values: 
    * **Select variable category:** Select *Terraform variable*.
    * **Key:** Enter *ami*.
    * **Value:** Paste in the AMI ID you copied.
1. Click **Save variable**.
1. Click **Add variable**, and set the following values: 
    * **Select variable category:** Select *Terraform variable*.
    * **Key:** Enter *instance_type*.
    * **Value:** Enter *t3.micro*.
1. Click **Save variable**.
1. Click **Add variable**, and set the following values: 
    * **Select variable category:** Select *Terraform variable*.
    * **Key:** Enter *subnet*.
    * **Value:** Paste in the subnet ID you copied earlier.
1. Click **Save variable**.
1. In the terminal, delete the `type` and `default` lines from each variable block, so they all look like: 

    ```
    variable "instance type" {}
    ```

1. Save and exit the file by pressing **Escape** followed by `:wq!`.

### Deploy the EC2 Instances with Terraform Cloud

1. Initialize the working directory:

    ```
    terraform init
    ```

    You should then see a success message.

1. Validate the code:

    ```
    terraform validate
    ```

    You should then see a success message.

1. Apply your configuration:

    ```
    terraform apply
    ```

1. At the prompt, enter `yes`.
1. In the Terraform Cloud console, in the left-hand menu, click **Runs**.
    * You should see a run has been triggered. 
1. Click the **Triggered via CLI** box to see the **Apply running** output. 
1. In the terminal, note that you see the same output listed. 
1. In the AWS Management Console, navigate to **EC2** > **Instances**.
    * You should see three instances listed, including the two you just created. 

### Create a Terraform Module from Existing Code

1. In the GitHub console, create a new repo, and set the following values: 
    * **Owner:** Select yourself. 
    * **Repository name:** Enter *terraform-as-ec2-psacg*.
    * **Private:** Select. 
    * **Add .gitignore:** Search for and select **Terraform**.
1. Click **Create repository**.
1. Click **Add file** > **Create new file**. 
1. For the file name, enter *main.tf*.
1. In the terminal, reopen `main.tf`: 

    ```
    vim main.tf 
    ```

1. Copy everything from the `resource` block down, which should be:

    ```
    resource "aws_instance" "inst" {
      count         = var.instance_count
      ami           = var.ami
      instance_type = var.instance_type
      subnet_id     = var.subnet

      tags = {
        Name = "TERRAFORM-GURU-${count.index}"
      }
    }

    variable "instance_count" {}

    variable "ami" {}

    variable "instance_type" {}

    variable "subnet" {}

    output "aws_instances" {
      value = aws_instance.inst.*.tags.Name
    }
    ```

1. Paste it into the **Edit new file** window. 
1. Click **Commit new file**.
1. Click **0 tags**.
1. Click **Create a new release**.
1. Click **Choose a tag**. 
1. Enter *1.0.0*.
1. Click **Create new tag: 1.0.0 on publish**.
1. For **Release title**, enter *Module v1.0.0*.
1. Click **Publish release**.
1. Click **Code**.
1. In the upper right, click your avatar, and select **Settings**.
1. In the left-hand menu, click **Applications**.
    * You shouldn't see any apps listed. 
1. In the Terraform Cloud console, in the left-hand menu, click **Overview**.
1. Click **Actions** > **Lock workspace**.
1. In the left-hand menu, click **Settings** > **Version Control**.
1. Click **Connect to version control**.
1. Click **Version control workflow**. 
1. Click **GitHub**.
1. In the **Install Terraform Cloud** dialog, select your GitHub user account. 
1. Select **Only select repositories**.
1. Click **Select repositories**. 
1. Click the **terraform-as-ec2-psacg** repository. 
1. Click **Install**.
1. In the Terraform Cloud console, under **Choose a repository**, click **terraform-as-ec2-psacg**.
1. Leave all the default settings, and click **Update VCS settings**.
1. In the left-hand menu, click **terraform-cloud-guru**.
1. In the left-hand menu, click **Workspaces**.
1. In the left-hand menu, click **Registry**.
1. With **Modules** selected, click **Publish a module**.
1. Click **GitHub**.
1. Click the **terraform-as-ec2-psacg** repository. 
1. Click **Publish module**.

### Modify the Terraform Configuration to use the Module and Redeploy

1. On the right-hand side, copy what's provided under **Copy configuration details**.
1. In the terminal, still in the `main.tf` file, delete the entire `resource` block.
1. Paste in the configuration details you copied. 
1. Replace `# insert required variables here` with the following lines: 

    ```
    instance_count = var.instance_count
    ami            = var.ami
    instance_type  = var.instance_type
    subnet         = var.subnet
    ```

1. In the `output` block, change the value to `module.ec2-psacg.*.aws_instances`:

    ```
    output "aws_instances" {
      value = module.ec2-psacg.*.aws_instances
    ```

1. Save and exit the file by pressing **Escape** followed by `:wq!`.
1. In the Terraform Cloud console, in the left-hand menu, click **Workspaces**.
1. Click **terraform-cloud-guru**.
1. Click **Settings** > **Version Control**.
1. Under **Connected to VCS**, click **Change source**.
1. Click **CLI-driven workflow**.
1. Click **Update VCS settings**.
1. In the left-hand menu, click **terraform-cloud-guru**.
1. Under **Latest Run**, click **See details**.
1. Click **Cancel Run**.
1. Click **Cancel Run** again.
1. Click **Actions** > **Unlock workspace**.
1. Click **Yes, unlock workspace**.
1. In the terminal, initialize the working directory:

    ```
    terraform init
    ```

    You should then see a success message.

1. Validate the code:

    ```
    terraform validate
    ```

    You should then see a success message.

1. Apply your configuration:

    ```
    terraform apply
    ```

1. At the prompt, enter `yes`.
1. In the Terraform Cloud console, in the left-hand menu, click **Runs**.
    * You should see a run has been triggered. 
1. Click the **Triggered via CLI** box to see the **Apply running** output. 
1. In the terminal, note that you see the same output listed. 
1. In the AWS Management Console, navigate to **EC2** > **Instances**.
    * You should see the new instances you originally created have been terminated, and two new instances have been created. 
    
## Conclusion

Congratulations on successfully completing this hands-on lab!