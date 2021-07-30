# Chapter 2. Cloud Native Fundamentals -> Lesson 5: CI/CD with Cloud Native Tooling

## 7. Exercise: The CI Fundamentals

### Exercise

Create a new GitHub Actions in the `.github/workflows/docker-build.yml` that will build and push the Docker
image for a Python web application, with the following requirements:

```
Image name: python-helloworld
Tag: latest
Platforms: platforms: linux/amd64,linux/arm64
```

Please note that there are two typos in the description of this exercise:

1. There should be no leading slash in `/.github/workflows/docker-build.yml`
2. The link to https://github.com/udacity/nd064_course_1/tree/main/solutions/lesson1 is incorrect,
   it should be https://github.com/udacity/nd064_course_1/tree/main/solutions/python-helloworld


### Initial setup

The exercise can be performed inside of the vagrant box, using the Vagrantile provided in the
course repository under `exercises`
```sh
git clone https://github.com/udacity/nd064_course_1
cd nd064_course_1
cd exercises
vagrant up
vagrant ssh
```

1. Sign up for an account at https://hub.docker.com/ and
generate an "Access Token" at https://hub.docker.com/settings/security.
See https://docs.docker.com/docker-hub/access-tokens/.
Store it in a secret place, like a password manager.

2. Sign up for an account at https://github.com/join, and create the **private** github repository
called `python-helloworld` for this exercise.

3. Store the dockerhub username the "Access token" generated in step 1.
in the github repository secrets, as `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`, respectively.
Those secrets will be used by the github action to push images to dockerhub.
See https://docs.github.com/en/actions/reference/encrypted-secrets for more about secrets.

4. Install docker https://en.opensuse.org/Docker, then exit and enter the `vagrant ssh` session again,
so `vagrant` user gains the `docker` group membership.
This is needed to execute `docker` commands without sudo
  ```sh
  sudo zypper install -y docker
  sudo systemctl enable docker
  sudo usermod -G docker -a $USER
  sudo systemctl start docker
  exit
  vagrant ssh
  id | grep docker  # verify the docker group membership
  ```

5. Install git, curl and tree
  ```sh
  sudo zypper install -y git curl tree
  ```

6. Generate the ssh key-pair and upload the **public** key to the github account Settings
   See https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account
   ```sh
   ssh-keygen -t ed25519 -C "exercise7" -f ~/.ssh/id_ed25519
   ```
   Come up with a password that will protect that key pair.
   The password will be used every time you commit to the remote repository.
   The contents of the public key to be uploaded is here
   ```sh
   cat ~/.ssh/id_ed25519.pub
   ```
   
7. Add the ssh private key to the ssh agent. It will remember (store) your password,
  so pushing to github won't hang on password prompt
  ```sh
  eval "$(ssh-agent -s)"
  ssh-add ~/.ssh/id_ed25519
  ```

8. If github.com blocks your ssh (git push hanging), use ssh over https
https://docs.github.com/en/github/authenticating-to-github/troubleshooting-ssh/using-ssh-over-the-https-port
  ```sh
cat << 'EOF' > ~/.ssh/config
Host github.com
  Hostname ssh.github.com
  Port 443
  User git
EOF
```

9. Install https://github.com/nektos/act. It's a tool for testing github actions locally, using `docker`
  ```sh
  curl -sLO https://github.com/nektos/act/releases/download/v0.2.23/act_Linux_x86_64.tar.gz
  curl -sLO https://github.com/nektos/act/releases/download/v0.2.23/checksums.txt
  grep act_Linux_x86_64 checksums.txt > checksum.txt
  sha256sum -c checksum.txt
  sudo sh -c "tar -zxOf act_Linux_x86_64.tar.gz act > /usr/local/bin/act"
  sudo chmod ugo+x /usr/local/bin/act
  which act
  act --version
  ```


### Exercise solution

1. Clone the course repository, while inside of the `vagrant ssh` session,
   under the `/vagrant` folder to have the exercise accessible also from the laptop host
  ```sh
  cd /vagrant
  git clone https://github.com/udacity/nd064_course_1
  ```

2. Configure the git email/username, and setup the exercise directory to use it as the remote repository
  ```sh
  git clone git@github.com:marcindulak/python-helloworld python-helloworld1
  git config --global user.email "you@example.com"
  git config --global user.name "Your name"
  ```

3. Copy the solution code into the local repository directory
  ```sh
  cd python-helloworld1
  cp ../nd064_course_1/solutions/python-helloworld/* .
  ```

4. Commit (push) the initial state of the repository changes to github
  ```sh
  git add .
  git commit -m "Add the course materials"
  git push -u origin main
  ```

5. Create `.gitignore` and `.dockerignore` files to to avoid committing the
`my.secrets` file to git and inside of the docker image. The file will be located one directory up,
and this is just a double protection measure
  ```sh
  echo my.secrets > .gitignore
  echo my.secrets > .dockerignore  
  ```
  
6. Implement the github action, store it in the `.github/workflows/main.yaml` file.
  ```sh
  mkdir -p .github/workflows
  ```
  Use `act` for local testing before pushing the changes to github.
  Save the dockerhub `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` in the local
  `../my.secrets` file to be used by `act`. Place the file one directory up,
  to avoid committing it to git. The file should look like:
  ```sh
  cat ../my.secrets
  DOCKERHUB_USERNAME="dockerhub-username-here"
  DOCKERHUB_TOKEN="token-here"
  ```
  Execute the github action workflow locally
  ```sh
  act --secret-file ../my.secrets
  ```
  This should build and push the image to dockerhub.

7. Push the changes to github
  ```sh
  git add .dockerignore .gitignore .github
  git commit -m"Add github action"
  git push
  ```

8. Test the docker image
  ```sh
  docker run -d --rm -it --name test marcindulak/python-helloworld:latest
  docker exec -it test curl localhost:5000
  Hello World!
  ```
