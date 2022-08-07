This repository contains all the resources I've followed in these two months. I hope that these will atleast help anyone looking to get started.

## Celery
- [Task Queues](https://www.fullstackpython.com/task-queues.html)
- [Celery with RabbitMQ](https://www.digitalocean.com/community/tutorials/how-to-use-celery-with-rabbitmq-to-queue-tasks-on-an-ubuntu-vps)
- [Celery with Django](https://simpleisbetterthancomplex.com/tutorial/2017/08/20/how-to-use-celery-with-django.html)

## Django
- [Official Django Docs](https://docs.djangoproject.com/en/4.1/): It's quite long, but it's gold mine if you know what to search for.

## Django Rest Framework
- [Django: Building REST APIs](https://masnun.com/2017/05/13/django-building-rest-apis.html): One of the best that helped me to get started with DRF
- [DRF Permissions in Depth](https://nezhar.com/blog/django-rest-framework-permissions-in-depth/)
- [DRF Official Docs](https://www.django-rest-framework.org/tutorial/quickstart/): How could I miss this?

## Testing
- [Why Testing?](https://eev.ee/blog/2016/08/22/testing-for-people-who-hate-testing/): If you're someone like me who don't appreciate testing, this post will enlighten you.
- [Testing](https://www.fullstackpython.com/testing.html)

## All-in-one
Apart from those, I'll share my go-to site for any query regarding webdev in Python
- [Full Stack Python](https://www.fullstackpython.com/table-of-contents.html): All/most of the links above were from this site.


## Jenkins
This frustrated me a lot! Seriously the setup itself made me reconsider my decision as a backend developer. Let me go through "why"?
The Jenkins can be setup in two ways:
- From the Jenkins binary from the official website.
- From the Docker image from DockerHub.

I used the first option initially. It was all good. But since I learnt Docker by the time I'm doing this, I wanted to try the second method.
And every thing was good (Atleast until I tried using docker inside my pipelines). Yes! Using Docker commands inside my pipelines irritated me as it either gives:
- `docker command not found`
- `docker permission denied`

Boom! I went through every thread, every stackoverflow answer trying to solve this. Nah! the thing worked for them didn't work for me.
Then I saw this post:
- [Installing docker inside Jenkins container](https://faun.pub/how-to-install-docker-in-jenkins-container-4c49ba40b373)
Oh man, I wish I could thank him personally for writing this post. This solved my error that made me struggle for two complete days (Saturday and Sunday 🥲).

Ok. Enough of this! Let me go through the steps.

## Jenkins Documentation

1. On Jenkins dashboard, click on **Manage Jenkins** button to the left. 

2. Go to **Manage Credentials**. We need this step to store our credentials for GitHub (if we want to access private repos through Jenkins), DockerHub, ACR, SSH etc.

**Note**: Before this, we need to install **Credentials Plugin** from **Manage Plugins** tab.


3. Next, click on **Jenkins** button. This is basically means that the scope is all over your Jenkins account.


4. Next, select **Global credentials**.


5. Then, click **Add Credentials** to add your credentials.


6. There were many options under **Kind** dropdown to select from. For this case, We'll select **Username with password**. Fill in your Usnername and Password (the access token for GitHub). Then give it an **ID**, this should be unique for each credential and this is how we'll able to access our credentials inside our Jenkins pipeline.


7. For our case, we'll select Multibranch Pipeline that is best suited for projects with multiple branches. Also, don't forget to give this an unique name.


8. Next, We'll give it a more readable name under **Display Name** and a **Description**.


9. Under **Add Sources** dropdown, select **Git** as our SCM.


10. Provide the Git repo URL and if the repo is private, we need to provide the credentials we saved earlier to this **Credentials** dropdown.


11. This picture below shows that Jenkins searches for Jenkinsfile in the root of the repo by default. We can change the path if needed. **Scan Multibranch Pipeline Triggers**. This is an option that enables Jenkins to poll the Git repo for every specified time interval to detect any changes. We fixed it to do so for every 24 hours.


12. Upon clicking **Save**, this scans the repo and checks each branch for a **Jenkinsfile**.

These are the basic steps to setup a pipeline. Now let's look into the `Jenkinsfile`:

```groovy
pipeline {
    # agent any is analogous to saying "run this pipeline on any machine (Linux, Windows, MacOS)."
    agent any
    # We can specify the docker images to use inside pipeline. This enables to use python-related-stuff inside the pipeline.
    // agent {  
    //     docker { image 'python:3.8.10' }
    // }
    environment {
    # stores or defines the environment variables used for in the pipeline
    
        # ACR is the ID given to the acr credentials stored in the Credentials section in Jenkins.
        acr = credentials('ACR')

        registryName = 'counsel'
        registryUrl = 'counsel.azurecr.io'
        registryUsername = 'counsel'
        registryRepo = 'cai'
        DATE_TAG = java.time.LocalDate.now()
        TIME_TAG = java.time.LocalTime.now().toString().replace(':', '-')
    }

    stages {
    # Pipeline is a series of stages. One stages fails, whole pipeline is said to be failed.
    # Basically we can merge all the stages into a single stage. But that isn't ideal. 
    # So we'll divide the whole process into various stages and this division is completely dependent on the use case and the developer.
    
    
        stage('Setup') {
        # First stage.
        
        
            steps {
            # Each stage is a series of steps or series of commands.
            
                sh 'echo Installing requirements'
            }
        }
        stage('Test') {
            steps {
                sh 'echo Testing...'
            }
        }
        stage('Merge into Main')
        {
            steps {
                sh 'echo Merging...'
            }
        }
        stage('Done') {
            steps {
                sh 'echo Done mergning into main'
            }
        }
        stage('Build Image') {
            steps {
                // sh 'docker build -t article_generation_${DATE_TAG} .'
                // sh 'docker tag ${registryUrl}/${registryRepo}:cai_ds_article_gen_backend_marketing ${registryUrl}/${registryRepo}:cai_ds_article_gen_backend_marketing_${DATE_TAG}'
                sh 'docker build -f Dockerfile -t ${registryUrl}/${registryRepo}:cai_ds_article_gen_backend_marketing_${DATE_TAG}_${TIME_TAG} .'
            }
        }
        // stage('Start Pushing to Docker Hub') {
        //         steps {
        //         sh 'docker tag todo_app_jenkins $dockerhub_USR/sample_repo:todo_app_jenkins'
        //         sh 'echo $dockerhub_PSW | docker login -u $dockerhub_USR --password-stdin'

        //         sh 'docker push $dockerhub_USR/sample_repo:todo_app_jenkins'
        //         }
        // }
        stage('Pushing to acr') {
            steps {
                sh 'echo $acr_PSW | docker login -u $acr_USR --password-stdin $registryUrl'
                sh 'docker push ${registryUrl}/${registryRepo}:cai_ds_article_gen_backend_marketing_${DATE_TAG}_${TIME_TAG}'
            }
        }
    }
}
```
