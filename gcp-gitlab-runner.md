## GitLab runner on Google compute engine instance

#### Instance setup (Creating the VM)
1. Go to [https://console.cloud.google.com/compute/instances](https://console.cloud.google.com/compute/instances) and log in with your Google credentials.
2. Click on "Create instance".
3. Choose an appropriate name.
4. Region should be the same as your k8s cluster.
5. Choose machine type (better to have at least 4gb of memory).
6. Select disk size (I used 20gb SSD) and system (my choice was Debian) in Boot disc.
7. Allow https traffic.

#### Machine setup
1. Connect to an instance using SSH (it's possible to open SSH just in browser window using GCP interface).
2. Install the latest stable version of docker using [official docker instruction](https://docs.docker.com/engine/install/debian/#install-using-the-repository)
3. Install using [gitlab repository](https://docs.gitlab.com/runner/install/linux-repository.html#installing-gitlab-runner)
   Don't forget to setup [APT pinning](https://docs.gitlab.com/runner/install/linux-repository.html#apt-pinning) (only Debian).
4. Register a runner
`sudo gitlab-ci-multi-runner register -n --url GITLAB_URL --registration-token "TOKEN"   --executor docker   --description "Name of docker runner"   --docker-image "docker:latest" --docker-privileged`

**GITLAB_URL** and **TOKEN** can be obtained on GitLab CI/CD runners settings for the group/project.

#### Finish

Check that runner is visible in CI/CD>Runners of your repository/group.
