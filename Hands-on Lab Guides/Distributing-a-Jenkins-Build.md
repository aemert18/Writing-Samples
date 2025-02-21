# Distributing a Jenkins Build

## Introduction

In this hands-on lab, we will create a build agent on a second server and then create a build that runs only on that server. This will demonstrate the use of labels for the build.

## Solution 

Log in to the `Jenkins Controller` server with the credentials provided: 

```
ssh cloud_user@<CONTROLLER_PUBLIC_IP_ADDRESS>
```

### Configure SSH on the Remote Agent

1. View the home directory contents: 

    ```
    [cloud_user@jenkins ~]$ ls 
    ```

    Once you see a `SERVER-READY` file listed, continue with the rest of the lab.

1. Log in to the agent node using the credentials provided: 

    ```
    [cloud_user@jenkins ~]$ ssh cloud_user@<AGENT_PUBLIC_IP_ADDRESS>
    ```

1. Create the directory for the user's home:

    ```
    [cloud_user@agent ~]$ sudo mkdir /var/lib/jenkins
    ```

1. Add the user, assigning the home directory:

    ```
    [cloud_user@agent ~]$ sudo useradd -d /var/lib/jenkins jenkins
    ```

1. Make the user the owner of their home directory:

    ```
    [cloud_user@agent ~]$ sudo chown -R jenkins:jenkins /var/lib/jenkins
    ```

1. Create an `.ssh` directory for the `jenkins` user:

    ```
    [cloud_user@agent ~]$ sudo mkdir /var/lib/jenkins/.ssh
    ```

1. Generate an SSH key: 

    ```
    [cloud_user@agent ~]$ ssh-keygen
    ```

1. Press **Enter** to accept defaults until it completes.

1. View the contents of `~/.ssh/id_rsa.pub`:

    ```
    [cloud_user@agent ~]$ cat ~/.ssh/id_rsa.pub
    ```

1. Copy the output, and paste it into a text file. 

1. Create a `/var/lib/jenkins/.ssh/authorized_keys` file:
    ```
    [cloud_user@agent ~]$ sudo vim /var/lib/jenkins/.ssh/authorized_keys
    ```

1. Paste in the `id_rsa.pub` contents you just copied. 

1. Save and exit the file by pressing **Escape** followed by `:x`.

1. View the contents of the `id_rsa` private key:

    ```
    [cloud_user@agent ~]$ cat ~/.ssh/id_rsa
    ```

1. Copy the entirety of the output, and paste it into a text file. 

1. Exit the agent node:

    ```
    [cloud_user@agent ~]$ exit 
    ```

1. Create an `.ssh` directory on the controller in the `jenkins` directory:

    ```
    [cloud_user@jenkins ~]$ sudo mkdir /var/lib/jenkins/.ssh
    ```

1. Copy the `known_hosts` entry over from the `.ssh` directory in `/home/cloud_user` to the `jenkins` user's `.ssh` directory:

    ```
    [cloud_user@jenkins ~]$ sudo cp ~/.ssh/known_hosts /var/lib/jenkins/.ssh
    ```

### Set Up the Node on the Jenkins Controller

1. Navigate to the Jenkins controller web interface: `http://<PUBLIC_IP_OF_JENKINS_CONTROLLER>:8080`

1. In the terminal, get the initial admin password: 

    ```
    [cloud_user@jenkins ~]$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```

1. Copy the output, and paste it into the **Administrator password** field in the Jenkins web interface.

1. Click **Continue**.

1. Click **Install suggested plugins**.

1. On the **Create First Admin User** screen, 

    * **Username:** Enter whatever you'd like (e.g., *student*).
    * **Password:** Use the same one as the provided Jenkins controller password on the lab page
    * **Full name:** Enter whatever you'd like. 
    * **E-mail address:** Enter a fake email address.

1. Click **Save and Continue**.

1. Click **Save and Finish**.

1. Click **Start using Jenkins**.

1. On the **Manage Jenkins** page, click **Manage Nodes and Clouds**.

1. In the left-hand menu, click **New Node**, and set the following values:
    - **Node name:** Enter *worker1*.
    -  **Permanent Agent:** Select.
    - Click **Create**.
    - **#of executors:** Enter *2*.
    - **Remote root directory:** Enter */var/lib/jenkins*.
    - **Labels:** Enter *linux*.
    - **Launch method:** Select **Launch agents via SSH**.
    - **Host:** Enter the public IP address of the agent node.
    - **Credentials:** Click **Add** > **Jenkins**. 
        - **Kind:** Select **SSH Username with private key**.
        - **Description:** Enter *jssh*.
        - **Username:** Enter *jenkins*.
        - **Private Key:** Select **Enter directly**, click **Add**, and paste in the `id_rsa` private key you copied earlier. 
        - Click **Add**.
    - **Credentials**: Click **- none -**, and select **jenkins (jssh)**.

1. Leave the rest as their defaults, and click **Save**.

1. On the **Nodes** page, click **worker1**.

1. In the left-hand menu, click **Log** to make sure there was a successful SSH authentication. 
    * If you see it in the log, and you see an entry that says *Agent successfully connected and online*, then it all worked correctly.

### Test a Remote Build

1. In the upper left corner, click **Jenkins**. 

1. In the left-hand menu, click **Manage Jenkins**.

1. Click **Configure System**, and then set the following values:
    - **Labels:** Enter *jenkins*.
    - **Usage**: Select **Only build jobs with label expressions matching this node**.
    
1. Click **Save**.

1. Click **New Item**. 

1. Under **Enter an item name**, enter *test*.

1. Select **Freestyle project**.

1. Click **OK**.

1. Under **Build Steps**, click **Add build step** > **Execute shell**.

1. In the **Command** box, enter *hostname > location.txt*.

1. Under **Post-build Actions**, click **Add post-build action** > **Archive the artifacts**.

1. In the **Files to archive** box, enter *location.txt*. 

1. Click **Advanced**.

1. Check the box next to **Fingerprint all archived artifacts**.

1. Click **Save**.

1. In the left-hand menu, click **Build Now**.

1. Under **Build History**, click the build number. 

1. In the left-hand menu, click **Console Output** to see the build process happening.
    * You should see that it's building remotely on `worker1` on the `test` workspace, and it should say it's archiving artifacts and recording fingerprints. 

1. In the breadcrumb navigation at the top of the screen, click **test**.
    * You should see `location.txt` listed under **Last Successful Artifacts**.
    
1. Right-click **view** to open it in a new browser tab.
    * You should see the `agent` hostname. 

## Conclusion

Congratulations on successfully completing this hands-on lab!