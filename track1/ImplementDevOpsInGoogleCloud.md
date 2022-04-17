# Implement DevOps in Google Cloud: Challenge Lab
> Created and Managed by [Shivam Vishwakarma](https://www.github.com/svshiva) 


_Before starting the challenge labs,
Go to Kubernets Cluster and check if jenkins-cd is present or not._

## **Task 1: Configure a Jenkins pipeline for continuous deployment to Kubernetes Engine**


_Open Cloud console and run following commands_

    gcloud config set compute/zone us-east1-b

    gcloud container clusters get-credentials jenkins-cd

    kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)

This will get credentials for your jenkins

Now you start the jenkins at port 8080,

    export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
    
    kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &

The below command will give you password for your jenkins

    printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo

Now in console click on web preview button, and then click show on port 8080

![Image](img/web_preview.png)

-   Now enter username : **admin**
-   and enter Password that you got from **previous command.**


---

### Adding your service account credentials

1. In the Jenkins user interface, click Manage Jenkins in the left navigation then click Manage Credentials.

2. Click Jenkins

3. Click **Global credentials (unrestricted).**

4. Click **Add Credentials** in the left navigation.

5. Select Google Service Account from metadata from the Kind drop-down and click OK.

![](img/gsp330-jenkins-add-gcp-credentials.webp)


Now run following command to create the Kubernetes namespace to logically isolate the deployment:

    
    kubectl create ns production

---

### Cloning the sample-app
-   goto VM Intances 
-   SSH on kraken-jumphost
-   Run the following commands in SSH window
        
        gcloud config set compute/zone us-east1-b

-   Now clone the sample app repo (Change Your-Project-ID)

        gcloud source repos clone sample-app --project=<YOUR-PROJECT-ID>

-   Now run following commands

        cd sample-app
        kubectl create ns production



### Configure the Jenkins job

1.  Click Jenkins to return the welcome page.

2.  Click New Item in the left navigation.
![](img/newItem.png)

3.  Name the project sample-app, then choose the Multibranch Pipeline option and click OK.
![](img/1_-L9qFTwNWaEKnp1XnyjxbA.png)

4.  On the next page, in the Branch Sources section, click Add Source and select git.
5.  Paste the HTTPS clone URL of your sample-app repo in Cloud Source Repositories into the Project Repository field. Replace [PROJECT_ID] with your Project ID.

        https://source.developers.google.com/p/<YOUR-PROJECT-ID>/r/sample-app

6. Select the service account for your GCP project from the Credentials dropdown list.

> after doing so, it should look like this

![](img/gsp330-jenkins-sample-app-job-branch-sources-config.webp)

7.  Check **Periodically if not otherwise run** in the Scan Multibranch Pipeline Trigger section, and then select **1 minute** for the interval.

![](img/gsp330-jenkins-sample-app-job-build-and-trigger-config.webp)

8. Scroll to the bottom and click on Save.

---
_If you correctly configured the pipeline, you will see Finished: SUCCESS at the end of the Scan Multibranch Pipeline Log._

![](img/gsp330-jenkins-sample-app-scan-multibranch-pipeline-log.webp)

---

### Examine the pipeline

-   Click on sample-app » master and wait until it gives you a green tick.
-   When it gets completed go back to your lab and check if task 1 is completed.

---
## Task 2: Push an update to the application to a development branch

>Now you have to do required changes to your website and push them in a development branch.

- Goto SSH Window
- Make sure you are in sample-app directory

- Create a new development branch using folloing command 

        git checkout -b new-feature

- now you have to change 2 files
> main.go and html.go, for this use any editor you like, vi or nano. I am giving instructions for nano.

-   type
        
        nano main.go
- **Search** for line

        const version string = "1.0.0"
- **Change** it to

        const version string = "2.0.1"

> To save it press Ctrl+X and then press Y and hit Enter.

-   Open the html.go file and change the two instances of `<div class="card blue">` to `<div class="card green">`

-   Now change html.go file

        nano html.go
- **Find 2 instances** of line 

        <div class="card blue">
-   **Change it to** 

        <div class="card green">

> **Remember you have to change 2 lines in html.go**

> To save it press Ctrl+X and then press Y and hit Enter


Now run following commands to push the code.

    git config --global user.email "you@example.com"
    git config --global user.name "Your Name"
    git add .
    git commit -m "Version 2.0.1"
    git push origin new-feature

-   Go to the Jenkins user interface, and check if new build is running for your new branch wait till it gets completed.

-   Go and check your progress.

## Task 3: Push a Canary deployment to the production namespace

1.  Go back to the SSH window, run the following command to create a canary branch in the sample-app directory.
    
        git checkout -b canary

2.Merge the change from the development branch:

        git merge new-feature

3. Push the canary to the Git server:

        git push origin canary

4.  Go to the Jenkins user interface, and check if new build is running for your **canary** branch wait till it gets completed.

---
## Task 4: Promote the Canary Deployment to production

1. Go back to the SSH window, run the following commands to merge the canary branch, and push it to the Git server.

        git checkout master
        git merge canary
        git push origin master

2.  Go to the Jenkins user interface, and check if new build is running for your **canary** branch wait till it gets completed.



**Congratulations! You completed this challenge lab.**


> Created and Managed by [Shivam Vishwakarma](https://www.github.com/svshiva) 
