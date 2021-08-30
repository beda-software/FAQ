## GitLab runner on EC2 instance (AWS)

#### Instance setup
1. Open **AWS console** - choose **EC2** - choose **Instances** - Press **Launch Instance**
2. Amazon machine image: Choose **Amazon Linux 2 AMI (HVM), SSD Volume Type**, click **Next**
3. Instance type: Choose **t3.medium**, click **Next**
4. Configure instance details - skip this step by clicking **Next**
5. Storage: Add **20gb** storage (gp2), click **Next**
6. Tags: skip by clicking **Next**
7. Create new security group for ssh 22 port (or use existing for ssh), click **Review and Lauch**
8. Generate new ssh key-pair by entering name and downloading .pem file
9. Once .pem is downloaded, set right chmod: `chmod 0600 filename.pem`

#### Instance post-setup
1. Open **AWS console** - choose **EC2** - choose **Instances**
2. Choose just-created instance, click **Actions** - **Security** - **Change security groups**
3. Add existing cluster’s security group to ec2's security groups (it will give an access to the cluster to make deployments from the runner)

#### Machine setup
1. Open **AWS console** - choose **EC2** - choose **Instances**
2. Click to the just-created instance to see detail page
3. Click **Connect** - choose **SSH client**
4. Run the suggested command to connect via ssh:
    1. `ssh -i "path-to-pem/filename.pen" ec2-user@HOST.compute.amazonaws.com`
5. Run following commands to setup required dependencies:
    1. `curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash`
    2.  `sudo -E yum install gitlab-runner`
    3.  `sudo amazon-linux-extras install docker`
    4.  `sudo service docker start`
    5.  `sudo usermod -a -G docker ec2-user`
    6.  `sudo systemctl enable docker.service`
    7.  `sudo systemctl enable containerd.service`
    8.  `sudo yum install -y git`
    9.  `sudo gitlab-ci-multi-runner register -n --url GITLAB_URL --registration-token "TOKEN"   --executor docker   --description "Name of docker runner"   --docker-image "docker:latest" --docker-privileged`

**GITLAB_URL** and **TOKEN** can be obtained on GitLab CI/CD runners settings for the group/project.

#### Finish

Now you're able to use this runner.

