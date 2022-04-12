Task 5. CI-CD Pipeline
Create a CI-CD pipeline for a sample application using any CI-CD tool of your choice like
Jenkins, Azure DevOps, Gitlab, Github Actions, AWS CodePipeline or any other tool of your
choice. Include a code build and a docker build step in your pipeline
---

# CI-CD pipeline for a sample application using Google Cloud tools and Gitlab.

This doc describes how to set up and use a development, continuous integration (CI), and continuous delivery (CD) system using an integrated set of Google Cloud tools and Gitlab.

### Architecture overview

![Architecture](https://miro.medium.com/max/624/1*SQcTRfQ2Cqoq18yofRsvTQ.png
"Architecture")

- **Cloud Build** to build and test the application—the "CI" part of the pipeline.
- **Cloud Run** to develop and deploy container applications.
- **Google Cloud Deploy** to manage the deployment—the "CD" part of the pipeline.
- **Gitlab** to be used as a repository and version control.

### Build and Deploy the Application by Hand

1. Make a working project repo in Gitlab, and take note of project SSH URL. I have used sample "cicd2" Express project.
1. Start up your Google Cloud Shell.
2. Clone the project using ```git clone {your project SSH URL}```.
3. We can’t use the docker commands inside the CI/CD pipeline inside GitLab. Instead we’ll use Google Cloud Build to build the container image. 
Before building the pipeline, we’ll do everything by hand first to make sure it is going to work.
4. Change to the project directory with ```cd cicd2```.
5. Instead of “docker build …” and “docker push …” use “gcloud builds …” as follows: ```gcloud builds submit --tag gcr.io/PROJECT-ID/cicd2```
You will have to replace PROJECT-ID with your actual GCP project ID.
6. You can test the image as follows: ```docker run --rm --env PORT=8080 -it -p 8080:8080 gcr.io/PROJECT-ID/cicd2``` *or* 
Test the app using “Web Preview” in the Cloud Shell environment. When you are satisfied that everything is working, stop the container with *CTRL-C*.
7. Finally, deploy the app to Cloud Run as follows: ```gcloud run deploy cicd2 --image gcr.io/PROJECT-ID/cicd2 --platform managed --region us-west1 --allow-unauthenticated```
The command will output the URL for the deployed app. Test it out in your browser.[^1]
[^1]: (if command throws error) Here region is "us-west1" in GCP, gitlab region must also be same.

### Build the GitLab CI/CD Pipeline

The next step is the to automate the build and deploy steps with a Gitlab CI/CD pipeline.

1. Create a ".gitlab-ci.yml" file in the project directory that defines the pipeline as follows:

```Ignore List
default:
    image: google/cloud-sdk:alpine
    before_script:
        - gcloud config set project PROJECT-ID

build:
    stage: build
    script:
        - gcloud builds submit --tag gcr.io/PROJECT-ID/cicd2

deploy:
    stage: deploy
    script:
        - gcloud run deploy cicd2 --image gcr.io/PROJECT-ID/cicd2 --platform managed --region us-west1 --allow-unauthenticated
```

Don’t forget to replace PROJECT-ID with your actual GCP project ID.

The defualt image identifies the default container image to use for the jobs in the pipeline. In this case, we are using the google/cloud-sdk:alpine image because it contains the gcloud command.

The default “before_script” contains the list of commands to execute before each job. In this case, we just set the default GCP project.

This pipleine has two jobs named “build” and “deploy”. The “build” job is run during the “build” stage and the “deploy” job is run during the “deploy” stage. “script” lists the commands to be run for the job.

Stage, commit, and push the changes as follows:

git add .
git commit -m "define initial CI/CI pipeline"
git push
If you look at the GitLab log for the “build” stage of the pipeline, you will see that it failed with a message similar to this:

 $ gcloud builds submit --tag gcr.io/cpsc-2650/cicd2
ERROR: (gcloud.builds.submit) You do not currently have an active account selected.
Please run:
  $ gcloud auth login
to obtain new credentials, or if you have already logged in with a
different account:
  $ gcloud config set account ACCOUNT
to select an already authenticated account to use.
Running after_script
00:02
Uploading artifacts for failed job
00:01
ERROR: Job failed: exit code 1
When we built the app in the Cloud Shell environment, we never had to supply authentication credentials because we were already logged into the GCP environment. Because GitLab is external to Google Cloud Platform, we are going to have supply credentials to allow GitLab CI/CD to access GCP resources.

Using GCP Credentials in a GitLab CI/CD Pipeline
First, use the GCP Console to create a service account with the appropriate permissions to to build and deploy an app that we can use in the GitLab CI/CD Pipeline.

From the GCP menu, choose “IAM & Admin | Service Accounts”. Click on the “CREATE SERVICE ACCOUNT” link. For “Service account name”, type “gitlab”. The “Service account ID” field should get filled in automatically. For “Service account description”, enter “GitLab CI/CD”. Click on the “Create” button. For “Service account permisisons”, add the following roles:

Project | Viewer
Cloud Build | Cloud Build Service Account
Cloud Run | Cloud Run Admin
Service Management | Cloud Run Service Agent
Click on the “CONTINUE” button. Click on the “CREATE KEY” button. Make sure “JSON” is selected and click on the “CREATE” button. The private key will be downloaded to your computer, probably in the “Downloads” folder. Click on the “DONE” button.

Open up the downloaded private key in a text editor.

Go to the “Settings | CI/CD” page for the project in GitLab. Expand the “Variables” section. Click on the “Add Variable” button. For the “Key” field, enter “GCP_SERVICE_CREDS”. For the “Value” field, pasted the contents of the private key file. For “Type”, choose “File”. Under “Flags”, check “Protect” variable. Click on the “Add Variable” button.

Now we have to back to the CI/CD pipeline definition and incorporate the service credentials. In the “before_script” section of the .gitlab-ci.yml file, add:

- gcloud auth activate-service-account --key-file $GCP_SERVICE_CREDS
Stage, commit, and push the changes as follows:

git add .
git commit -m "add service account credentials to pipeline"
git push
Both jobs should complete successfully now. Check that the application is still running. Make a visible change to the views/index.ejs file. Stage, commit, and push the change. Confirm that the change appears on the running application.
