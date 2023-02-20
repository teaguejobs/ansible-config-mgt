**Configuring Ansible For Jenkins Deployment**

- In previous projects, you have been launching Ansible commands manually from a CLI. Now, with Jenkins, we will start running Ansible from Jenkins UI.

To do this,

1. Navigate to Jenkins URL

2. Install & Open Blue Ocean Jenkins Plugin

3. Create a new pipeline

![alt text](./images.png/blue%20ocean.PNG)

4. Select GitHub
![alt text](./images.png/select%20github.PNG)

5. Connect Jenkins with GitHub

![alt text](./images.png/connect%20jenkins%20to%20github.PNG)

6. Login to GitHub & Generate an Access Token
![alt text](./images.png/login%20github.PNG)

7. Copy Access Token

![alt text](./images.png/copy%20access%20token.PNG)

8. Paste the token and connect

![alt text](./images.png/paste%20token.PNG)

9. Create a new Pipeline

- At this point you may not have a Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give you some guidance to create one. But we do not need that. We will rather create one ourselves. So, click on Administration to exit the Blue Ocean console.

![alt text](./images.png/admin.PNG)

- Let us create our Jenkinsfile
Inside the Ansible project, create a new directory deploy and start a new file Jenkinsfile inside the directory.

![alt text](./images.png/create%20jenkins%20file%202.PNG)

- Add the code snippet below to start building the Jenkinsfile gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage

`pipeline {`
    `agent any`

  `stages {`
    `stage('Build') {`
      `steps {`
        `script {`
          `sh 'echo "Building Stage"'`
         `}`
       `}`
    `}`
    `}`
`}`

10. Now go back into the Ansible pipeline in Jenkins, and select configure

11. Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/Jenkinsfile
![alt text](./images.png/jenkins%20file%20location.PNG)

12. Back to the pipeline again, this time click "Build now"

- This will trigger a build and you will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.

![alt text](./images.png/build2.PNG)

- To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.
![alt text](./images.png/build3.PNG)

- Notice that this pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.

- Let us see this in action.

1. Create a new git branch and name it feature/jenkinspipeline-stages

2. Currently we only have the Build stage. Let us add another stage called Test. Paste the code snippet below and push the new changes to GitHub.

 `pipeline {`
    `agent any`

  `stages {`
    `stage('Build') {`
      `steps {`
        `script {`
          `sh 'echo "Building Stage"'`
        `}`
      `}`
    `}`
   `stage('Test') {`
      `steps {`
        `script {`
          `sh 'echo "Testing Stage"'`
       `}`
      `}`
    `}`
    `}`
`}`

3. To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.

Click on the "Administration" button

![alt text](./images.png/admin%20button.PNG)

4. Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.

![alt text](./images.png/both%20branch.PNG)

5. In Blue Ocean, you can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch.
![alt text](./images.png/blue%20ocean%20test%20stage.PNG)

6. Create a pull request to merge the latest code into the main branch

7. After merging the PR, go back into your terminal and switch into the main branch.

8. Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an echo command like we have in build and test stages)

- package
- deploy
- clean up

![alt text](./images.png/package.PNG)

9. Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch

10.
6. Eventually, your main branch should have a successful pipeline like this in blue ocean

![alt text](./images.png/successful%20pipeline.PNG)

**RUNNING ANSIBLE PLAYBOOK FROM JENKINS**

- Now that we have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by

1. Installing Ansible on Jenkins

`sudo yum install ansible`
`ansible --version`

- Install ansible dependecies
`yum install python3 python3-pip wget unzip git -y`
`python3 -m pip install --upgrade setuptools`
`python3 -m pip install --upgrade pip`
`python3 -m pip install PyMySQL`
`python3 -m pip install mysql-connector-python`
`python3 -m pip install psycopg2==2.7.5 --ignore-installed`

![alt text](./images.png/ansible%20version.PNG)

2. Installing Ansible plugin in Jenkins UI

- Manage Jenkins>Global tool configuration>add ansible
- copy the path to ansible ,use the command `which ansible`

![alt text](./images.png/configure%20ansible.PNG)

3. Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)

Note: Ensure that Ansible runs against the Dev environment successfully.

**Possible errors to watch out for:**

1. Ensure that the git module in Jenkinsfile is checking out SCM to main branch instead of master (GitHub has discontinued the use of Master due to Black Lives Matter. You can read more here)

![alt text](./images.png/git%20module.PNG)

2. Jenkins needs to export the ANSIBLE_CONFIG environment variable. You can put the .ansible.cfg file alongside Jenkinsfile in the deploy directory. This way, anyone can easily identify that everything in there relates to deployment. Then, using the Pipeline Syntax tool in Ansible, generate the syntax to create environment variables to set.

![alt text](./images.png/ansible%20cfg.PNG)

![alt text](./images.png/environment%20path.PNG)

- Update our role path on the jenkinsfile as thus:

![alt text](./images.png/roles%20path.PNG)

- We generate our steps for ansible playbook on our jenkins file using the pipeline syntax tool

![alt text](./images.png/manage%20credential.PNG)

- copy and paste  the private key
-

![alt text](./images.png/credentials.PNG)

![alt text](./images.png/invoke%20ansible.PNG)

- Generate pipeline script

![alt text](./images.png/pipeline%20script.PNG)

**NOTE:**

- We already updated our inventory/dev.yml with the private IP of our instances

![alt text](./images.png/inventory%20dev.PNG)

- Now lets scan repository on our jenkins
![alt text](./images.png/ansible%20build%20success1.PNG)

- If everything goes well it means, the Dev environment has an up-to-date configuration. But what if we need to deploy to other environments?

- Are we going to manually update the Jenkinsfile to point inventory to those environments? such as sit, uat, pentest, etc.
- Or do we need a dedicated git branch for each environment, and have the inventory part hard coded there.

- Manually updating the Jenkinsfile is definitely not an option. And that should be obvious to you at this point. Because we try to automate things as much as possible.

**Parameterizing Jenkinsfile For Ansible Deployment**

To deploy to other environments, we will need to use parameters

1. Update sit inventory with new servers

![alt text](./images.png/sit.PNG)

2. Update Jenkinsfile to introduce parameterization. Below is just one parameter. It has a default value in case if no value is specified at execution. It also has a description so that everyone is aware of its purpose.
`pipeline {`
    `agent any`

    `parameters {
string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy`configuration)`
    `}`

![alt text](./images.png/parameters.PNG)

3. In the Ansible execution section, remove the hardcoded inventory/dev and replace with `${inventory}
From now on, each time you hit on execute, it will expect an input.

![alt text](./images.png/inventory%20variable.PNG)

- Notice that the default value loads up, but we can now specify which environment we want to deploy the configuration to. Simply type sit and hit Run
![alt text](./images.png/build%20sit.PNG)

**CI/CD PIPELINE FOR TODO APPLICATION**

- Our goal here is to deploy the application onto servers directly from Artifactory rather than from git. If you have not updated Ansible with an Artifactory role, simply use this guide to create an Ansible role for Artifactory

**Phase 1 – Prepare Jenkins**

1. Fork the repository below into your GitHub account
`https://github.com/darey-devops/php-todo.git`

2. On you Jenkins server, install PHP, its dependencies and Composer tool

`yum module reset php -y`
`yum module enable php:remi-7.4 -y`
`yum install -y php  php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd    php-fpm php-json`
`systemctl start php-fpm`
`systemctl enable php-fpm`

3. Install Jenkins plugins

- plot plugin - We will use plot plugin to display tests reports, and code coverage information.
- artifactory plugin - plugin will be used to easily upload code artifacts into an Artifactory server.

4. create and configure an ubuntu 20.04 instance for our artifactory server and copy the private IP to our ci inventory
![alt text](./images.png/sit.PNG)

- Run our ansible playbook from jenkins using the ci inventory on our build with parameters,using the artifactory roles

![alt text](./images.png/build%20para.PNG)

![alt text](./images.png/build%20artifactory.PNG)

5. In Jenkins UI configure Artifactory
![alt text](./images.png/jenkins%20ui.PNG)

6. Configure the server ID, URL and Credentials, run Test Connection

![alt text](./images.png/jenkins%20jfrog.PNG)

**Phase 2 – Integrate Artifactory repository with Jenkins**

1. Create a dummy Jenkinsfile in the repository

![alt text](./images.png/dummy%20jenkins.PNG)

2. Using Blue Ocean, create a multibranch Jenkins pipeline

3. On the database server, create database and user

`Create database homestead;`
`CREATE USER 'homestead'@'%'` `IDENTIFIED BY 'sePret^i';`
`GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';`

4. Update the database connectivity requirements in the file .env.sample

![alt text](./images.png/env%20file.PNG)

5. Update Jenkinsfile with proper pipeline configuration

`pipeline {`
    `agent any`

  `stages {`

     stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

    stage('Checkout SCM') {
      steps {
            git branch: 'main', url: 'https://github.com/darey-devops/php-todo.git'
      }
    }

    stage('Prepare Dependencies') {
      steps {
             sh 'mv .env.sample .env'
             sh 'composer install'
             sh 'php artisan migrate'
             sh 'php artisan db:seed'
             sh 'php artisan key:generate'
      }
    }
  }
}

![alt text](./images.png/pipeline%20todo.PNG)

- Notice the Prepare Dependencies section

- The required file by PHP is .env so we are renaming .env.sample to .env
- Composer is used by PHP to install all the dependent libraries used by the application
- php artisan uses the .env file to setup the required database objects – (After successful run of this step, login to the database, run show tables and you will see the tables being created for you)

![alt text](./images.png/show%20tables.PNG)

4. Update the Jenkinsfile to include Unit tests step
`stage('Execute Unit Tests') {`
      steps {
             `sh './vendor/bin/phpunit'
      }
  }

![alt text](./images.png/unit%20test.PNG)

**Phase 3 – Code Quality Analysis**

- For PHP the most commonly tool used for code quality analysis is phploc
- The data produced by phploc can be ploted onto graphs in Jenkins.

1. Add the code analysis step in Jenkinsfile. The output of the data will be saved in build/logs/phploc.csv file

`stage('Code Analysis') {`
  `steps {`
        `sh 'phploc app/ --log-csv build/logs/phploc.csv'`

  `}`
`}`

2. Plot the data using plot Jenkins plugin.

- This plugin provides generic plotting (or graphing) capabilities in Jenkins. It will plot one or more single values variations across builds in one or more plots. Plots for a particular job (or project) are configured in the job configuration screen, where each field has additional help information. Each plot can have one or more lines (called data series). After each build completes the plots’ data series latest values are pulled from the CSV file generated by phploc.

`stage('Plot Code Coverage Report') {`
      `steps {`

            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)                          ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of code', yaxis: 'Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Directories,Files,Namespaces', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'B - Structures Containers', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Average Class Length (LLOC),Average Method Length (LLOC),Average Function Length (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'C - Average Length', yaxis: 'Average Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'D - Relative Cyclomatic Complexity', yaxis: 'Cyclomatic Complexity by Structure'      
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Classes,Abstract Classes,Concrete Classes', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'E - Types of Classes', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'F - Types of Methods', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Constants,Global Constants,Class Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'G - Types of Constants', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Test Classes,Test Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'I - Testing', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'AB - Code Structure by Logical Lines of Code', yaxis: 'Logical Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Functions,Named Functions,Anonymous Functions', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'H - Types of Functions', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'BB - Structure Objects', yaxis: 'Count'

      }
    }`

NOTE: in order for the pipeline to run successfully we need to install some dependecies for phploc

- `sudo dnf --enablerepo=remi install php-phpunit-phploc`
- `wget -O phpunit https://phar.phpunit.de/phpunit-7.phar`
- `chmod +x phpunit`
- `sudo yum  install php-xdebug`

- If you're getting the error "Code coverage needs to be enabled in php.ini by setting 'xdebug.mode' to 'coverage'", it means that the Xdebug extension for PHP is not configured correctly to generate code coverage reports.

- Locate your php.ini file: The location of the php.ini file can vary depending on your system. You can use the command php --ini to find the location of the php.ini file that is used by your PHP installation.
Edit the php.ini file: Open the php.ini file in a text editor and add the following line at the end of the file:
xdebug.mode = coverage
 Restart your web server: After making changes to the php.ini file, you'll need to restart your web server for the changes to take effect.

- Verify the configuration: You can use the command php -i | grep xdebug to check the current configuration of xdebug. Make sure that the xdebug.mode setting is set to 'coverage'.

- Re-run your phpunit test: Re-run your phpunit test and verify that the code coverage report is generated correctly.

![alt text](./images.png/plot%20coverage%20report.PNG)

3. Bundle the application code for into an artifact (archived package) upload to Artifactory

`stage ('Package Artifact') {`
    `steps {`
            `sh 'zip -qr php-todo.zip ${WORKSPACE}/*'`
     `}`
    `}`

- Ensure that zip is installed
`sudo yum install zip -y`

4. Publish the resulted artifact into Artifactory

`stage ('Upload Artifact to Artifactory') {`
          `steps {`
            `script {`
                 `def server = Artifactory.server` `'artifactory-server'`
                 `def uploadSpec = """{`
                    `"files": [`
                      `{`
                       `"pattern": "php-todo.zip",`
                       `"target":` `"<name-of-artifact-repository>/php-todo",`
                       `"props": "type=zip;``status=ready"`

                       }
                    ]
                 }""" 

                 server.upload spec: uploadSpec
               }
            }

        }

![alt text](./images.png/upload%20artifact.PNG)

5. Deploy the application to the dev environment by launching Ansible pipeline

`stage ('Deploy to Dev Environment') {`
    `steps {`
    `build job: 'ansible-project/main', parameters: [[$class: 'StringParameterValue', name: 'env', value:` `'dev']], propagate: false, wait: true`
    `}`
  `}`

![alt text](./images.png/php%20todo%20pipeline.PNG)

![alt text](./images.png/deploy%20to%20env.PNG)

- The build job used in this step tells Jenkins to start another job. In this case it is the ansible-project job, and we are targeting the main branch. Hence, we have ansible-project/main. Since the Ansible project requires parameters to be passed in, we have included this by specifying the parameters section. The name of the parameter is env and its value is dev. Meaning, deploy to the Development environment.

- Even though we have implemented Unit Tests and Code Coverage Analysis with phpunit and phploc, we still need to implement Quality Gate to ensure that ONLY code with the required code coverage, and other quality standards make it through to the environments.

- To achieve this, we need to configure SonarQube – An open-source platform developed by SonarSource for continuous inspection of code quality to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities.

**SONARQUBE INSTALLATION**

*Install SonarQube on Ubuntu 20.04 With PostgreSQL as Backend Database*

- We can automate our installation by creating an ansible role for sonarqube,or by downloading a role from the community as thus

![alt text](./images.png/install%20sonar%20roles.PNG)

- We will make some Linux Kernel configuration changes to ensure optimal performance of the tool we will increase vm.max_map_count, file discriptor and ulimit.
![alt text](./images.png/sonar%20limit.PNG)

- Update our inventory file with private i.p of our sonarqube server in our ci environment

![alt text](./images.png/sonar%20private%20ip.PNG)

- Set our parameter in jenkins to ci then run ansible playbook in jenkins

![alt text](./images.png/sonar%20playbook.PNG)

![alt text](./images.png/sonar%20playbook2.PNG)

**Access SonarQube**

- To access SonarQube using browser, type server’s IP address followed by port 9000

`http://server_IP:9000 OR http://localhost:9000`

- Login to SonarQube with default administrator username and password – admin

![alt text](./images.png/sonar%20success%20page.PNG)

**CONFIGURE SONARQUBE AND JENKINS FOR QUALITY GATE**

1. In Jenkins, install SonarScanner plugin

2. Navigate to configure system in Jenkins. Add SonarQube server as shown below:

![alt text](./images.png/sonar%20config.PNG)

3. Generate authentication token in SonarQube

`User > My Account > Security > Generate Tokens`

![alt text](./images.png/gen%20auth%20token.PNG)

4. Configure Quality Gate Jenkins Webhook in SonarQube – The URL should point to your Jenkins server

`http://{JENKINS_HOST}/sonarqube-webhook/`

`Administration > Configuration > Webhooks > Create`

![alt text](./images.png/sonar%20webhook.PNG)

5. Setup SonarQube scanner from Jenkins – Global Tool Configuration

`Manage Jenkins > Global Tool Configuration`

![alt text](./images.png/sonarscanner%20installation.PNG)

**Update Jenkins Pipeline to include SonarQube scanning and Quality Gate**

`cd php-todo`
Below is the snippet for a Quality Gate stage in Jenkinsfile.

 `stage('SonarQube Quality Gate') {`
        `environment {`
            `scannerHome = tool 'SonarQubeScanner'`
        `}`
        `steps {`
            `withSonarQubeEnv('sonarqube') {`
                `sh "${scannerHome}/bin/``sonar-scanner"`
            `}`

        }
    }

- Add this stage before 'package artifact' .

![alt text](./images.png/stage%20add.PNG)

- NOTE: The above step will fail because we have not updated `sonar-scanner.properties

- Configure sonar-scanner.properties – From the step above, Jenkins will install the scanner tool on the Linux server. You will need to go into the tools directory on the server to configure the properties file in which SonarQube will require to function during pipeline execution.

`cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/conf/`

- Open sonar-scanner.properties file

`sudo vi sonar-scanner.properties`

- Add configuration related to php-todo project

`sonar.host.url=http://<SonarQube-Server-IP-address>:9000/sonar`
`sonar.projectKey=php-todo`
`#----- Default source code encoding`
`sonar.sourceEncoding=UTF-8`
`sonar.php.exclusions=**/vendor/**`
`sonar.php.coverage.reportPaths=build/logs/clover.xml`
`sonar.php.tests.reportPath=build/logs/junit.xml`

- Install npm dependency on our jenkins server - `sudo yum install npm -y`


- HINT: To know what exactly to put inside the sonar-scanner.properties file, SonarQube has a configurations page where you can get some directions

![alt text](./images.png/sonar%20paths.PNG)

- A brief explanation of what is going on the the stage – set the environment variable for the scannerHome use the same name used when you configured SonarQube Scanner from Jenkins Global Tool Configuration. If you remember, the name was SonarQubeScanner. Then, within the steps use shell to run the scanner from bin directory.

- To further examine the configuration of the scanner tool on the Jenkins server – navigate into the tools directory

`cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/bin`

List the content to see the scanner tool sonar-scanner. That is what we are calling in the pipeline script.

`ls -latr`

![alt text](./images.png/sonar%20ls.PNG)

- Our pipeline should look like this

![alt text](./images.png/end%20to%20end%20pipeline.PNG)


But we are not completely done yet!

- The quality gate we just included has no effect. Why? Well, because if you go to the SonarQube UI, you will realise that we just pushed a poor-quality code onto the development environment.

- Navigate to php-todo project in SonarQube

![alt text](./images.png/poor%20quality%20code.PNG)

There are bugs, and there is 0.0% code coverage. (code coverage is a percentage of unit tests added by developers to test functions and objects in the code)

If you click on php-todo project for further analysis, you will see that there is 6 hours’ worth of technical debt, code smells and security issues in the code.

![alt text](./images.png/further%20analysis.PNG)

In the development environment, this is acceptable as developers will need to keep iterating over their code towards perfection. But as a DevOps engineer working on the pipeline, we must ensure that the quality gate step causes the pipeline to fail if the conditions for quality are not met.

**Conditionally deploy to higher environments**

In the real world, developers will work on feature branch in a repository (e.g., GitHub or GitLab). There are other branches that will be used differently to control how software releases are done. You will see such branches as:

- Develop
- Master or Main
(The * is a place holder for a version number, Jira Ticket name or some description. It can be something like Release-1.0.0)
- Feature/*
- Release/*
- Hotfix/*

- There is a very wide discussion around release strategy, and git branching strategies which in recent years are considered under what is known as GitFlow

- Assuming a basic gitflow implementation restricts only the develop branch to deploy code to Integration environment like sit.

Let us update our Jenkinsfile to implement this:

- First, we will include a When condition to run Quality Gate whenever the running branch is either develop, hotfix, release, main, or master

`when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP"}`

- Then we add a timeout step to wait for SonarQube to complete analysis and successfully finish the pipeline only when code quality is acceptable.

` timeout(time: 1, unit: 'MINUTES') {`
        `waitForQualityGate abortPipeline: true`
    `}`

The complete stage will now look like this:

`stage('SonarQube Quality Gate') {`
      `when { branch pattern: "^develop*|^hotfix*|``^release*|^main*", comparator: "REGEXP"}`
        `environment {`
            `scannerHome = tool 'SonarQubeScanner'`
        `}`
        `steps {`
            `withSonarQubeEnv('sonarqube') {`
                `sh "${scannerHome}/bin/``sonar-scanner -Dproject.``settings=sonar-project.``properties"`
            `}`
            `timeout(time: 1, unit: 'MINUTES') {`
                `waitForQualityGate ``abortPipeline: true`
            `}`
        `}`
    `}`

- To test, create different branches and push to GitHub. You will realise that only branches other than develop, hotfix, release, main, or master will be able to deploy the code.

- If everything goes well, you should be able to see something like this:

![alt text](./images.png/skip%20dev.PNG)

- Notice that with the current state of the code, it cannot be deployed to Integration environments due to its quality. In the real world, DevOps engineers will push this back to developers to work on the code further, based on SonarQube quality report. Once everything is good with code quality, the pipeline will pass and proceed with sipping the codes further to a higher environment.


