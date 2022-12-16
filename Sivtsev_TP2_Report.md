# Discover Github Action

link: http://school.pages.takima.io/devops-resources/ch2-discover-github-actions-tp/

github repository: https://github.com/kuundyol/imp.git

### Goals

Good Practice
Do not forget to document what you do along the steps.
Create an appropriate file structure, 1 folder per image.

### Target Application

Complete pipeline workflow for testing and delivering your software application.
We are going to use different useful tools to build your application, test it automatically, and check the code quality at the same time.
link: https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions

# Git Init

First, we need to push our project (from TP1) to the git repository.

Signed in to git

Created new git repository https://github.com/kuundyol/imp

Opened git bash on my computer

Moved to the root folder of my project: `Users\igors\Documents\efrei\imp\tp1`

Used the init command to initialize the local directory as a Git repository. By default, the initial branch is called master.
````bash
git init
````

Added the files in your new local repository. This stages them for the first commit.
````bash
git add .
````

Commited the files that you've staged in your local repository.
````bash
git commit -m "Initial commit"
````

At the top of your repository on GitHub.com's Quick Setup page, clicked to copy the remote repository URL.
https://github.com/kuundyol/imp.git

In the Command prompt, added the URL for the remote repository where your local repository will be pushed.
````bash
$ git remote add origin https://github.com/kuundyol/imp.git
	#Sets the new remote
$ git remote -v
	#Verifies the new remote URL
````
Pushed the changes in your local repository to GitHub.com.
````bash
$ git push origin master
	#Pushes the changes in your local repository up to the remote repository you specified as the origin
	#My branch be default is called 'master'
````
Went to the git repository on the browser and saw that all the files are there. Success!


# Set uo GutHub Actions

I do not have maven installed on my machine so I could not perform manually on my machine the maven verification.

Created empty main.yml file in the root directory.

Moved it to the `./.github/workflows` new directory

Filled it with the following content:
````yml
name: CI devops 2022
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: master
  pull_request:

jobs:
  test-backend: 
    runs-on: ubuntu-22.04
    steps:
     #checkout your github code using actions/checkout@v2.5.0
      - uses: actions/checkout@v2.5.0

     #do the same with another action (actions/setup-java@v3) that enable to setup jdk 17
      - name: Set up JDK 17
      - uses: actions/setup-java@v3

     #finally build your app with the latest command
      - name: Build and test with Maven
        run: mvn clean verify
````

Added edited files to commit
````
git add .
````

Commited the files
````
git commit -m "Added workflow file"
````

Pushed the commit
````
git push origing master
````

The Action is triggered and running. It gave an error: 
> Error: every step must define a `uses` or `run` key

I checked the main.yml file and found a syntax mistake. Fixed it. Tried again with the new commit "Report tp1 finished"

Got an error again:
> Error: Node.js 12 actions are deprecated. For more information see: https://github.blog/changelog/2022-09-22-github-actions-all-actions-will-begin-running-on-node16-instead-of-node12/. Please update the following actions to use Node.js 16: actions/checkout@v2.5.0

To fix this, specified the java action step:
````yml
- name: Set up JDK 17
  uses: actions/setup-java@v3
  with:
    java-version: '17'
    distribution: 'temurin'
    cache: maven
````

Pushed on git again with commit "workflow 1.1". This fixed the problem. But I got an error in the next step:
> Error: The goal you specified requires a project to execute but there is no POM in this directory (/home/runner/work/imp/imp). Please verify you invoked Maven from the correct directory.

To fix this, specified the path to the pom.xml file:
````yml
- name: Build and test with Maven
  run: mvn clean verify --file ./simple-api-student-main/pom.xml
````

Pushed on git again with commit "workflow 1.2". The action passed. Succes!!

## First steps into the CD World
### 1. Added docker login 
````yml
job:
login:
    runs-on: ubuntu-22.04
    steps:
      -
        name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: kuundyol
          password: dckr_pat_jhTf3g-qkpRdncT7r3VThVc764I
````

Added changes and commited as "workflow 1.7". Login is successful

### 2. Built my docker images inside your GitHub Actions pipeline. 

Added job build-and-push-docker-image:
````yml
build-and-push-docker-image:
   needs: test-backend
   # run only when code is compiling and tests are passing
   runs-on: ubuntu-22.04

   # steps to perform in job
   steps:
     - name: Checkout code
       uses: actions/checkout@v2.5.0

     - name: Build image and push backend
       uses: docker/build-push-action@v3
       with:
         # relative path to the place where source code with Dockerfile is located
         context: ./simple-api-student-main
         # Note: tags has to be all lower-case
         tags:  kuundyol/my-database/backend

     - name: Build image and push database
       uses: docker/build-push-action@v3
       with:
         # relative path to the place where source code with Dockerfile is located
         context: ./database
         # Note: tags has to be all lower-case
         tags:  kuundyol/my-database/database
         # DO the same for database

     - name: Build image and push httpd
       uses: docker/build-push-action@v3
       with:
         # relative path to the place where source code with Dockerfile is located
         context: ./http
         # Note: tags has to be all lower-case
         tags:  kuundyol/my-database/httpd
         # DO the same for httpd
````
Added changes and commited as "workflow 1.9". Success!!

### 3. Publishing my docker images when there is a commit on the main branch.

Added following lines as a step:
````yml
- name: Build image and push backend
  uses: docker/build-push-action@v3
  with:
    # relative path to the place where source code with Dockerfile is located
    context: ./simple-api-student-main
    # Note: tags has to be all lower-case
    tags:  kuundyol/my-database:simple-api
    # build on feature branches, push only on main branch (I have master branch)
    push: true
````

I kept getting an error
> ERROR: denied: requested access to the resource is denied

Tried to resolve it but could not do that.



# Setuo Quality Gate

Created account on SonarCloud

Created organization with a key 'kuundyol'

Added this line to the script:
````yml
mvn -B verify sonar:sonar -Dsonar.projectKey=kuundyol -Dsonar.organization=kuundyol -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=b0aee3fdeec4cc391f38d1fac3b2107c7267f1c2  --file ./simple-api-student-main/pom.xml
````
Success!!

TP2 finished :)