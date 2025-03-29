# airbnb-tf-infra
# steps to creating a pipline
# on the jenkins server, click new item
# select pipeline, then ok
# scroll down and click on sample pipeline, then select "hello world" save and build now
# you can use pipeline syntax if you want to perform an operation in your pipeline and you dont know how to write the step to use. You select which operation you want to perform, copy and paste scm repository link, input the branch of your repo, add credentials (use your scm username and password), then generate pipeline script (this also clones your your repo in ur jenkins server.) 

# in writting a pipiline, everying must be written in a pipline block with curly braces opening and closing it, there are stages within the block, which may be to build, test and deploy.

# actual command you would be running would be found inside the steps of a stage.

# STAGE1-SCM checkout
# allow access to jenkins to access your scm by providing credentials, url and branch. 

# STAGE 2-BUILD/ set up environment variables
# in using terraform, you need to configure terraform in your jenkins server. first enable terraform plug-ins on your jenkins server, then configure your pipeline.
# on jenkins dashboard, go to manage jenkins, tools, click on add terraform and give it a name, "Terraform", search for version, "41023 amazon linux", save.

# go to pipeline and configure, on your pipeline script add terraform as a tool below "agent any", type the tool which is Terraform, just as you saved it. add the verion. apply and build.

# create a new stage for terraform plan

#########################################################################################################


# Jenkins project - airbnb_tf_infrastructure
 
1. new item
2.
 
3. open pipeline syntax in a new tab to see how to generate a pipeline script to clone your repo to a jenkins server
for eg. - git branch: 'main', credentialsId: '3ccdcaa2-412b-4793-b15d-dfa4c1c08ee4', url: 'https://github.com/dainmusty/airbnb_tf_infrastructure.git'
 
 
# groovy language template or layout(jenkins file)
pipeline {
    agent any
 
    stages {
        stage('SCM Checkout') {  # this line defines the stage
            steps {                 # steps or commands come below this line.
                echo 'Hello World'  # use the pip syntax to help generate the equilvalent commands in pip
                git branch: 'main'   # this is command for calling your repo in a jenkins file
                sh 'ls'                 # command for using a shell script to list files
            }
        }
    }
}
 
# you can either use a pipeline script within jenkins or use a pipeline script from SCM (source code management)
# for automation - use pip script from SCM and specify branch
 # to run terraform commands, we need to setup a terraform environment on jenkins without neccesarily installing terraform on the ec2 instance.
 
 # to do that, go to jenkins dashboard, manage jenkins, plugins, available plugins, type terraform, install the only option that comes up. go back to the top of the page.
 
 # go back to your pip and configure, go to the stage indentation in your pip (line 21 above) and hit enter to start type the next stage's commands as seen below
 
 pipeline {
    agent any
 
    stages {
        stage('SCM Checkout') {
            steps {
                echo 'cloning repo with jenkins server'
                git branch: 'main', credentialsId: '3ccdcaa2-412b-4793-b15d-dfa4c1c08ee4', url: 'https://github.com/dainmusty/airbnb_tf_infrastructure.git'
                sh 'ls'
            }
        }
       
        stage('terraform init') {
            steps {
                sh 'terraform init'
            }
        }    
    }
}
 
# though terraform has been installed on jenkins, the pip will fail because it has not been customised for this particular pip
 
# below are the step to configure
# go to manage jenkins and then tools, add terraform, type Terraform with caps and select terraform 41023 linux amd64 and save
 
# now you need to configure terraform on your pip
# in your pip script, below agent, hit enter twice and type tools as seen below
 
pipeline {
    agent any
   
    tools {
        terraform 'Terraform'
    }
   
 
    stages {
        stage('SCM Checkout') {
            steps {
                echo 'cloning repo with jenkins server'
                git branch: 'main', credentialsId: '3ccdcaa2-412b-4793-b15d-dfa4c1c08ee4', url: 'https://github.com/dainmusty/airbnb_tf_infrastructure.git'
                sh 'ls'
            }
        }
       
        stage('terraform init') {
            steps {
                sh 'terraform init'
            }
        }    
    }
}
 
# once terraform init is successful, build terraform plan by going back to configure your pip as seen below
pipeline {
    agent any
   
    tools {
        terraform 'Terraform'
    }
   
 
    stages {
        stage('SCM Checkout') {
            steps {
                echo 'cloning repo with jenkins server'
                git branch: 'main', credentialsId: '3ccdcaa2-412b-4793-b15d-dfa4c1c08ee4', url: 'https://github.com/dainmusty/airbnb_tf_infrastructure.git'
                sh 'ls'
            }
        }
       
        stage('terraform init') {
            steps {
                sh 'terraform init'
            }
        }  
       
        stage('terraform plan') {
            steps {
                sh 'terraform plan'
            }
        }
    }
}
 
#if your instance profile has no admin privileges, it will fail. two options; 1. ues iam role 2. use environmnetal variables
 
# to use env variables
go to manage jenkins, crenditial, global, add crenditials
add your aws crendtials, access and secret access keys
for kind - choose secret
for id - enter AWS_ACCESS_KEY_ID
for description - enter AWS_ACCESS_KEY_ID
for secret - enter your aws access key id
 
#do same for secret access key setup
for id - enter AWS_SECRET_ACCESS_KEY_ID
for description - enter AWS_SECRET_ACCESS_KEY_ID
for secret - enter your aws secret access key id
 
# add the env variables to your pip environment
{
        //Credentials for Prod environment
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }
 
see below how your final script should look like
pipeline {
    agent any
   
    tools {
        terraform 'Terraform'
    }
   
    environment {
        //Credentials for Prod environment
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }
   
   
    stages {
        stage('SCM Checkout') {
            steps {
                echo 'cloning repo with jenkins server'
                git branch: 'main', credentialsId: '3ccdcaa2-412b-4793-b15d-dfa4c1c08ee4', url: 'https://github.com/dainmusty/airbnb_tf_infrastructure.git'
                sh 'ls'
            }
        }
       
        stage('terraform init') {
            steps {
                sh 'terraform init'
            }
        }  
       
        stage('terraform plan') {
            steps {
                sh 'terraform plan'
            }
        }
    }
}
 
# to parameterize your pip
under configure, stay on general, select 'the project is parameterized'
 
add parameter and select the parameter you need for eg. lets use the choice parameter in this eg.
 
give it a name, maybe 'action' and in the choices box,
 
type 'apply' on one line and on the next line type 'destroy'
 
scroll down to pipeline
insert another stage and name it 'terraform action' and under steps add the shell script
terraform plus the parameter called ${action} plus --auto-approve just as seen below
 
pipeline {
    agent any
   
    tools {
        terraform 'Terraform'
    }
   
 
    stages {
        stage('SCM Checkout') {
            steps {
                echo 'cloning repo with jenkins server'
                git branch: 'main', credentialsId: '3ccdcaa2-412b-4793-b15d-dfa4c1c08ee4', url: 'https://github.com/dainmusty/airbnb_tf_infrastructure.git'
                sh 'ls'
            }
        }
       
        stage('terraform init') {
            steps {
                sh 'terraform init'
            }
        }  
       
        stage('terraform plan') {
            steps {
                sh 'terraform plan'
            }
        }
       
        stage('terraform action to apply or destroy plan') {
            steps {
                sh 'terraform ${action} --auto-approve'
            }
        }
    }
}
 
# to add a trigger to automate your builds
under configure, in trigger, select github hook trigger for gitscm polling - so anytime there is
a git push, it will trigger a build
add the link to your github project
 
then go to the project repo you are working on in your github to see webhooks; under settings
select 'add webhook', enter your password
 
under payload URL, paste the link below and change your jenkins server ip
http://54.161.200.183:8080/github-webhook/
 
 
# now make changes in your main.tf to confirm the automation
do git add .
git commit -m "test webhook"
git push
 
# to add pipeline from an SCM say github;
instead of using the 'pipeline script' as your pipeline definition, use 'pipeline from SCM' as your pipeline definition
 
so choose your 'SCM' = git
provide your repo link, credentials and the branch details and save
 
now do git add ., git commit and git git push for the jenkinsfile to be picked up
 
 
 
 
# to be discussed later
post {
        always {
            echo 'Cleaning up...'
            sh 'terraform destroy --auto-approve'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
 
 
 ###########################################################################################

 Devops end 2 end
DEVOPS END 2 END STEPS.
Page1:
(1) Bounce a Jenkins server( EC2) with public IP enabled. and the following ports opened; port 22, port 80 and port 8080.

(2) Connect to the server via ssh or ssm. 

(3) Copy the public IP of Server and paste into a new tab and put Colon 8080. This would open Jenkins web that displays a path.

(4) Copy this path and Sudo cat and paste it on your Server terminal. This would display an initial Jenkins password.

(5) Copy the password and paste it in the Jenkins page and in click Continue.

(6) Jenkins plugins opens with 2 options. Choose install suggested plugins.

(7) Create first Admin user. NOTE: put admin for all fields and admin@gmail.com for email field. Save and continue. 

Page2:
(8)Once finished click start using Jenkins and this will open Jenkins dashboard.
(9) Click on manage jenkins to see all the configurations around Jenkins. 
Click on plugins and go to available plugins on the upper left of page. A Search bar with a list of available plugins will appear. You can search plugins
like docker, Kubernetes etc.
(10) Click on manage Jenkins again and go to tools .. Once a plugin is installed, it can be configured as a tool. Click on tools to see all the build tools.
(11) Click back on manage Jenkins and then to newl item. Name the item demo- pipeline. Click on freestyle. project and Scroll down to Build steps. Click on Add build step drop down. Select Execute shell.
(12) A Command field will display under Execute Shell. Type echo' Hello world, enter and type puid enter and type whoami. Click on save and go to Build Now on upper left. Click Build Now. Once build succeeds or fails, click on its drop dowen and select Console output to see the results.

Page3:
Note: every data input on the Jenkins Console is persistent also on the Jenkins instance. Hence, you can create a snapshot or an ami of that instance, take it to another region and bounce the same Jenkins server. (13) Go to your Jenkins terminal and cd to  / var/ lib/Jenkins. Now Is to see all directories. Cd to Workspace and pwd.

(14) Go to your github and create a new repository.
(15) Clone the repository in your vs code by first cd to the exact directory where you want the repo to be cloned.
(16) Clone the repository and cd into that directory. Note that since it's an empty repo that we created you must provision terraform into it. Provide all the necessan files such as providers.tf, main.tf and variables.tf .
(17)Now run terraform init, plan and apply.com

(18) Run terraform destroy to take down the instances. 
Open new browser and search for terraform git ignore file. Copy the git ignore file, go to vs code and create a git ignore file. Paste what you copied from the browser (.gitignore) 

Page4:
(20) Right click terraform hcl and copy path. Go to .gitignore file. Go to the last line, enter and paste the path there.
(21) Now on this path delete everything from the beginning up to infra.

(22) Now run git add, git commit and git push to Push the repo.
Server it becomes
(23) NOTE: When you stop and start your Sto slow so you must optimize it. The Java JDK has also changed so you must connect to the server and run the following commands:
Sudo yum remove java* -y
sudo yum install java- 17- amazon- corretto.x86_64- y Sudo Systemctl start jenkins.

(24)Go to Jenkins browser and refresh. Go to dashboard and then to manage jenkins. Click on system. Scroll down to where you see jenkins URL. Copy the public. IP from your server and replace that section in the enkins URL. Save changes and refresh.

Page:5
(25) Go to new item in Jenkins dashboard name the item and Select pipeline.
(26) Scroll down and click on to pipeline, Select pipeline Script and then click on try sample pipeline drop down I and select" Hello World". Save and build now. Go to Builds drop down and click on Console output- to see The output of the build.
(27) Go to Configure and scroll down to pipeline script and then click on pipeline Syntax. Right click on pipeline Syntax and open in new tab.

(28) Open pipeline syntax. Click on the drop down and select Git. Go to your github and copy your repo link that you want to use. Change The branch to main. Go to credentials and click add credentials.
Put your username and password of your github account so that Jenkins can be able to log into your github account.

(29) Select the credential which you just input and
click on generate pipeline script. NOTE: Do NOT forget give the credential an ID name. Once you generate the script it will create a credentials string in the field below.

Page6:
(30) Replace Hello on line 5 of the script with SCM checkout.

 (31) Go to Pipeline Syntax. Copy the Credentials String that was generated from the pipeline syntax.

(32) Go to the pipeline script and replace Hello World on line 7 with cloning Code base with jenkins server. Enter and paste the credentials string on the next line. Hit enter again and type sh'ls' or go to Pipeline syntax and Select shell script from Sample step. Put ls in the shell script field and generate Pipeline Script. Copy what has been generated and paste on line 9 of the pipeline script.

(33) Apply changes and dreplicate tab. Click on save in the earlier tab. Build now and go to Console output.

(34)Go to the other duplicate tab. Go to manage Jenkin and then go to plugins. Search for the plugin Terraform Select it and install. Click go back to top of page.

(35) Go to the pipeline and
perform a build. 

Page7:
(36) Click on Configure. Scroll down to the pipeline and a second stage called terraform init.

(37) Go to line 11 and enter twice. Put a new Stage Called terraform init. Open curly brace and hit enter. Type steps, open curly brace and hit enter. Now put sh ‘terraform init’ or go to Pipeline Syntax and type terraform init in The Shell Script field and then generate pipeline script. Copy the generated output and paste in the pipeline script on line 15.

(38) Save and build now. Check the consde output for the outcome of the build.

(39) Configure terraform for use within this pipeline. Go to to tools. Scroll down to manage Jenkins and go terraform installations and click on add terraform. Type Terraform in the name field and select the terraform version 41023( Terraform 41023 linux( amd64))

(40) Save it and go to pipeline, Go to the end of line 2 and hit enter twice. Type tools, open Curly brace and enter. Type terraform' Terraform. Save it and build now. Go to Console output and check the outcome.

Page8:
(41) Go to the pipeline and add a stage for terraform plan. On line 23 hit enter twice and put the new stage there. Save and build now. Check console output.

(42) Go to AS console and then to IAM. Create a role with EC2 full access. Call the role Jenkinsrole. Create the role and view it.

(43) Go to EC2 instance that is running jenkins. Select The server and go to actions. In the drop down, go to Security and Select modify IAM role. Search Jenkinsrole, select it and click update IAM role.

(44) Go back to jenkins pipeline and run build. Now go to Console output and check output.

(45) Go back now to AWS console. Go to IAM, search for the role and delete it.

(46) Go back to jenkins pipeline. Go to manage Jenkins and click credentials. Select global. Click on add credentials.

(47) Go to the drop down field for kind and Select Secret text.

Page9:
(48) Put AWS_ACCESS_KEY_ID in both the ID and description fields. Go into your folders and provide the actual value of AWS_ACCESS_KEY_ID into the secret field.

(49) Put AWS SECRET_ACCESS_KEY and do the same as in step 48.

(50) Go now to configure and then to the pipeline. After the tools on line 7, enter twice and put the enviroment variables there 

environment {
// Credentials for prod environment
AWS ACCESS_KEY_ID = Credentials( ‘AWS_ACCESS_KEY_ID’) AWS_SECRET_ACCESS_KEY = Credentials( ‘AWS_SECRET_ACCESS_KEY’
        
             }

(51) Apply and build now in the duplicate tab. view the Console output.

(52) Go to configure and select this project is Parameterized. Click on add parameter drop down and select choice parameter. Give the name of the Parameter as action. Type apply, enter and type destroy. In the description field type select the terraform action.

Page10:
(53) Scroll down to the pipeline and add a new stage. for Call the stage terraform action to apply or destroy plan Put sh ‘terraform ${ action} --auto- approve’ in the steps Save and build now. Click on build with parameters. Select the apply action and build. Go to Console output to view results. Once successful, check the provisioned resources in your AWS console.

(54) Go back to the pipeline and build a destroy action with parameter.

(55) Click on Configure. Scroll down to triggers and Selec Github hook trigger for GITScm polling. Now scroll up and Select Github project. Now go and copy the URL of your repository and paste in the project url field.

(56) Save the changes in Step 55. Now go to your repository and then to settings. Find webhooks. Click on Webhooks and then click add Webhooks. Now Put your github account password there.

Page11:
(57) The payload URL is the url of your jenkins server. Put this there "http: // < your pubic IP >: 8080/ github-webhook/" Click on Add webhook.

(58) Go to your Jenkins and refresh. Go now to vs code and get into the main.tf. Copy from line 13 to 20. Enter twice on line 20 and paste. Replace prod with dev for the resource name and tags.

(59)Run git add, git commit and git push operations.

(60) Go now to Jenkins pipeline and refresh. A new build Should be triggered already. Check the console output and also check your AWS Console for the new resources. 

(61) Go to the pipeline and build a destroy action.

(62) Go to your pipeline. Click on configure, go to the pipeline. Highlight everything in the pipeline and copy it.

(63) Go to vs code and create a new file called 
Jenkinsfile. Paste the pipeline script. Now go back to Configure pipeline. Click the pipeline script drop down and then select pipeline Script from SCM. Select Git as the SCM from the drop down. Go and copy your repository link and come and paste in the repository url. Select your credentials in the Credential field. Change the branch to your branch( main). Save the changes and go back to the VS Code.

Page12:
(64) Run git add, git commit and git push operations. Go now to your pipeline and see if the push automatically triggered the pipeline.

(65) Go back to VS code and add a new echo command on line 18 that should say," echo' testing CI Code base With Jenkins Server. Save changes and run git add, git commit and git push operations.

(66) Go back to Jenkins, locate the pipeline and refresh Jenkins to see if the pipeline was triggered by the changes in VS Code.
 
 
 
 
 
 
 