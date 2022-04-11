Task 5. CI-CD Pipeline
Create a CI-CD pipeline for a sample application using any CI-CD tool of your choice like
Jenkins, Azure DevOps, Gitlab, Github Actions, AWS CodePipeline or any other tool of your
choice. Include a code build and a docker build step in your pipeline
---

# CI-CD pipeline for a sample application using Google Cloud tools.

This doc describes how to set up and use a development, continuous integration (CI), and continuous delivery (CD) system using an integrated set of Google Cloud tools.

### Architecture overview

![Architecture](https://miro.medium.com/max/624/1*SQcTRfQ2Cqoq18yofRsvTQ.png
"Architecture")

- **Cloud Build** to build and test the application—the "CI" part of the pipeline.
- **Cloud Run** to develop and deploy container applications.
- **Google Cloud Deploy** to manage the deployment—the "CD" part of the pipeline.
- **Github** to be used as a repository

### Before you begin
1. Go to Google Cloud Console, on the project selector page, select or create a Google Cloud project. [^1]
2. In the Cloud Console, activate Cloud Shell. ![Cloud Shell](https://github.com/woodensofa/supreme-octo-fortnight/blob/main/Task%205%20CI-CD%20Pipeline/Screenshot%20(52).png
"Cloud Shell")
3. 

[^1]: Note: If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.
