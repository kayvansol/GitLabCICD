# GitLab CI/CD

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/1.webp?raw=true)

GitLab is a web-based Git repository that provides free open and private repositories, issue-following capabilities, and wikis. It is a complete DevOps platform that enables professionals to perform all the tasks in a project — from project planning and source code management to monitoring and security.

This article will guide you on how to do CI/CD pipeline setup with GitLab.

Continuous integration and continuous delivery (CI/CD) is a methodology of automatically building and deploying code to provide you with greater speed and reliability. It is done in two parts: continuous integration (CI) and continuous delivery (CD).

Continuous delivery is then getting your code to a deliverable state, so it can be deployed at the click of a button. Or, in the case of continuous deployment, automatically deploy your code if all tests pass.

</hr>

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/2.webp?raw=true)

Installing docker on 2 servers with ansible 🎉
Related files are stored at my github :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/3.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/4.webp?raw=true)

Run the ansible playbook 🚀

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/5.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/6.webp?raw=true)

The result of installing docker ✔️:

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/7.webp?raw=true)

</hr>

Installing gitlab on docker (GitLabServer) 🎉
Pull the gitlab docker image from repository.
```
docker pull gitlab/gitlab-ce:nightly
```

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/8.webp?raw=true)

Gitlab compose file :
```
version: '3.6'

volumes:
  gitlab_backup:
    name: gitlab_backup
  gitlab_data:
    name: gitlab_data
  gitlab_logs:
    name: gitlab_logs
  gitlab_config:
    name: gitlab_config

services:
  gitlab:
    image: gitlab/gitlab-ce:nightly
    container_name: gitlab
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        # Add any other gitlab.rb configuration here, each on its own line
        external_url 'http://gitlab.example.com'
 
        # Change the initial default admin password
        gitlab_rails['initial_root_password'] = "y@1234567"
        gitlab_rails['display_initial_root_password'] = "false"
        gitlab_rails['store_initial_root_password'] = "false"

        # Nginx Configuration
        nginx['client_max_body_size'] = '10240m'
        nginx['gzip_enabled'] = true
        nginx['listen_port'] = 80
        nginx['listen_https'] = false
        nginx['proxy_cache'] = 'gitlab'
        nginx['http2_enabled'] = true
        nginx['listen_port'] = 80
        nginx['listen_https'] = false
        nginx['http2_enabled'] = false
        nginx['proxy_set_headers'] = {
          "Host" => "$$http_host",
          "X-Real-IP" => "$$remote_addr",
          "X-Forwarded-For" => "$$proxy_add_x_forwarded_for",
          "X-Forwarded-Proto" => "https",
          "X-Forwarded-Ssl" => "on"
        }

        # Configure a failed authentication ban
        gitlab_rails['rack_attack_git_basic_auth'] = {
          'enabled' => false,
          'ip_whitelist' => ["127.0.0.1"],
          'maxretry' => 10,
          'findtime' => 60,
          'bantime' => 3600
        }

        # Disable unuse services
        prometheus['enable'] = false
        # grafana['enable'] = false
        alertmanager['enable'] = false
        pgbouncer_exporter['enable'] = false
        puma['exporter_enabled'] = false
        gitlab_exporter['enable'] = false
        node_exporter['enable'] = false
        sidekiq['metrics_enabled'] = false
        redis_exporter['enable'] = false
        postgres_exporter['enable'] = false

        # gitlab backup config
        gitlab_rails['manage_backup_path'] = true
        gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"
        gitlab_rails['backup_archive_permissions'] = 0644
        gitlab_rails['backup_keep_time'] = 604800
        gitlab_rails['env'] = {"SKIP" => "registry"}

        # Gitlab registry config
        registry_external_url 'https://reg.gitlab.example.com'
        registry_nginx['listen_port'] = 5100
        registry_nginx['listen_https'] = false
        registry_nginx['proxy_set_headers'] = {
          "Host" => "$$http_host",
          "X-Real-IP" => "$$remote_addr",
          "X-Forwarded-For" => "$$proxy_add_x_forwarded_for",
          "X-Forwarded-Proto" => "https",
          "X-Forwarded-Ssl" => "on"
        }

    ports:
      - '80:80'
      - '443:443'
      - '2424:22'
    volumes:
      - gitlab_backup:/var/opt/gitlab/backups
      - gitlab_data:/var/opt/gitlab
      - gitlab_logs:/var/log/gitlab
      - gitlab_config:/etc/gitlab
```
```
docker compose up -d
```

Run the compose file 🚀

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/9.webp?raw=true)

Htop on the gitlab server :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/10.webp?raw=true)

Welcome to the GitLab. 🎉

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/11.webp?raw=true)

Create project related Users :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/12.webp?raw=true)

Now we need to create a Project and create a new branch or default (main) inside the Project Repository :

Here is the project has been created successfully.✔️

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/13.webp?raw=true)

Specify users to the project :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/14.webp?raw=true)

Global dashboard overview :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/15.webp?raw=true)

add user’s machine’s ssh public key for working with gitlab project :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/16.webp?raw=true)

Clone the project to your local area :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/17.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/18.webp?raw=true)

Create file & folders related to your project & commit and push them :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/19.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/20.webp?raw=true)

</hr>

Set Up a GitLab Runner 🎉
GitLab Runner is an application that works with GitLab CI/CD to run jobs in a pipeline.

Runners are available based on who you want to have access to :

Shared runners are available to all groups and projects in a GitLab instance.
Group runners are available to all projects and subgroups in a group.
Specific runners are associated with specific projects. Typically, specific runners are used for one project at a time.
Configuring the Runner server on docker compose file :
```
version: '3.8'

volumes:
  runner_data:
    name: runner_data
    external: false

services:

  gitlab-runner:
    image: gitlab/gitlab-runner:alpine-v17.6.1
    restart: unless-stopped
    container_name: gitlab-runner
    hostname: gitlab-runner
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - runner_data:/etc/gitlab-runner
```
```
docker compose pull

docker compose up -d
```
Running the compose 🚀

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/21.webp?raw=true)

Then create a runner inside gitlab (Settings > CI/CD > Runners > New project runner > Create > Register runner) :
```
gitlab-runner register  
--url http://gitlab.example.com  
--token glrt-t3_PtA9egtEgpG5AyZsoz6R
```
Inside the runner server :
```
docker exec -it gitlab-runner /bin/bash

gitlab-runner register  
--url http://gitlab.example.com  
--token glrt-t3_PtA9egtEgpG5AyZsoz6R
```

After running this command, it will ask below-mentioned details, provide those details, and your Runner will be ready :

1. Enter your GitLab instance URL (also known as the gitlab-ci coordinator URL).

2. Enter the token you obtained to register the runner(These details you can get from your GitLab Project > Settings > CI/CD > Runners )

3. Enter a description for the runner. You can change this value later through the GitLab user interface.

4. Enter the tags associated with the runner, separated by commas. You can change this value later through the GitLab user interface.

5. Enter any optional maintenance note for the runner.

6. Provide the runner executor. For most use cases, enter docker.

7. If you entered docker as your executor, you are asked for the default image to be used for projects that do not define one in .gitlab-ci.yml.
   
![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/22.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/23.webp?raw=true)

After providing those details we are able to see our Specific Runner has been Configured Successfully.✔️

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/24.webp?raw=true)

Create some changes at runner’s config.toml file (runner server) :
```
docker inspect --format='{{.NetworkSettings.Networks}}' 4819f97432fc
```

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/25.webp?raw=true)

```
vi /etc/gitlab-runner/config.toml
```

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/26.webp?raw=true)

</br>
It’s time to establish a CI/CD pipeline 🎉
For Running the Pipeline, first we need to write the .gitlab-ci.yml YML file. Create a file named .gitlab-ci.yml at the root of the project and write your desire dscripts & stages :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/27.webp?raw=true)
```
stages:
  - build

build-job:
  image: localhost:5000/docker:dind 
  stage: build  
  before_script:
    - 'command -v ssh-agent >/dev/null || ( apk add --update openssh )'
    - eval $(ssh-agent -s)
    - chmod 400 ${SSH_PRIVATE_KEY}
    #- echo ${SSH_PRIVATE_KEY}
    - ssh-add ${SSH_PRIVATE_KEY}
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan 192.168.56.157 >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
  script:    
    - |
      scp -o StrictHostKeyChecking=no -r app  root@192.168.56.157:/opt/text/
      scp -o StrictHostKeyChecking=no newfile.txt  root@192.168.56.157:/opt/text/
      # Create directory if not exist
      ##ssh -o StrictHostKeyChecking=no  -i  ${SSH_PRIVATE_KEY} root@192.168.56.157 "            
      # move compose and env file to server
      ##scp -o StrictHostKeyChecking=no  newfile.txt  root@192.168.56.157:/opt/text/            
      ##"        
  when: manual
```

Note : manual means continuous delivery (CD) & without this line we have continuous deployment (CD)

Note : For docker:dind image, we pull it from docker repository and then tag & push it to our local registry (on port 5000) for performance :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/28.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/29.webp?raw=true)

For copying objects to production server, we need a ssh key on gitlab runner & use it’s private key at .gitlab-ci.yml ${SSH_PRIVATE_KEY} and also copy it’s public key to production server :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/30.webp?raw=true)

Go to Settings > CI/CD > Variables :

Create a file typed variable named SSH_PRIVATE_KEY :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/31.webp?raw=true)

Copy the id_rsa (private key) entire content to the Value box and insert a new line at the end of it and then save changes :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/32.webp?raw=true)

and then copy the id_rsa.pub content to the production server :
```
ssh-copy-id -i .ssh/id_rsa.pub root@192.168.56.157
```

And after commit the file, pipeline will begin to work and send jobs to the runner server for executing and sending project files & folders to the production server at your desired location. 🚀

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/33.webp?raw=true)

After committing, our Pipeline has been executed and completed successfully, for more details click on Jobs and then click on specific job to see its logs and details.

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/34.webp?raw=true)

Related job execution details :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/35.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/36.webp?raw=true)

Here, GitLab Pipeline has been executed Successfully.✔️

Check the production server for transferred project contents :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/37.webp?raw=true)

</br>
Test again the pipeline with other user (root) ♻️ :

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/38.webp?raw=true)

Note : Commit 80e16891 🎉

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/39.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/40.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/41.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/42.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/43.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/44.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/45.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/46.webp?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/GitLabCICD/refs/heads/main/img/47.webp?raw=true)
