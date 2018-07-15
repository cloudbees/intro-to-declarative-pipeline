# Distributed Pipelines with CloudBees

## Checkpoints

In this exercise we are going to update our pipeline to demonstrate how **[Checkpoints](https://go.cloudbees.com/docs/cloudbees-documentation/cje-user-guide/#workflow-sect-checkpoint)** work and how end users can interact with Checkpoints once a job as been built. 

1. Add the following stage after `stage('Testing')`:

```
      stage('Checkpoint') {
         agent none
         steps {
            checkpoint 'Checkpoint'
         }
      }
      stage('Deploy') {
         agent none
         steps {
            echo 'Deploying....'
         }
      }
```

2. **Save & Run** your pipeline.
3. The UI to interact with a `checkpoint` is not availbe in Blue Ocean yet, so you will need to exit to the classic Jenkins UI for the next step.
4. When the job has completed running you will see a **Resume** icon in the build's **Stage View**. Clicking on the **Resume** icon gives you the ability to:
  - * **Delete** - Delete the cached artifacts and configuration for that build;
  - * **Restart** - Restart the build from the checkpoint.
5. Click **Restart** and your pipeline will run from your `checkpoint` forward - skipping over the **Say Hello** stage

>**NOTE**: Deleting a checkpoint doesn't make the **Resume** icon vanish.

## Cross Team Collaboration
In this exercise we are going to set-up two Pipeline jobs (using the Jenkins classic UI) that demonstrate CloudBee's Cross Team Collaboration feature. We will need two separate Pipelines - one that publishes an event - and another that is triggered by an event.

### Master Events

For the first part of **Cross Team Collaboration** we will create an event that is only published on your master.

#### Publish Event

First you have to publish an event from a Pipeline - any other Pipeline may set a trigger to listen for this event. Create a Pipeline job named `notify-event` with the following content:

```
pipeline {
    agent none
    stages {
        stage('Publish Event') {
            steps {
                publishEvent simpleEvent('beeEvent')
            }
        }
    }
}
```

Replace `beeEvent` with your `{username}Event` so my event would be `beedemo-devEvent`.

#### Event Trigger

Next, create a Pipeline job name `notify-trigger` and set a `trigger` to listen for the event you created above with the following content:

```
pipeline {
    agent none
    triggers {
        eventTrigger simpleMatch('beeEvent')
    }
    stages {
        stage('Event Trigger') {
            when {
                expression { 
                    return currentBuild.rawBuild.getCause(com.cloudbees.jenkins.plugins.pipeline.events.EventTriggerCause)
                }
            }
            steps {
                echo 'triggered by published event'
            }
        }
    }
}
```

After creating both of these Pipeline jobs you will need to run the **Event Trigger** job once so that the trigger is registered (similar to what was necessary for job parameters). Once that is complete, click on **Build Now** to run the **Publish Event** job. Once that job has completed, the **Event Trigger** job will be triggered after a few seconds. The logs will show that the job was triggered by an `Event Trigger` and the `when` expression will be true.

>**NOTE**:  If your *trigger* job does not fire, you may need to enable the **Cross Team Collaboration** feature on your master.  Navigate to the top level of your master, select **Manage Jenkins** and then **Configure Notification**.  Next, select **Enable** and choose **Local only** and then click **Save**.

### Cross-Master Events

For the second part of **Cross Team Collaboration** we will create an event that will be published **across Team Masters** via CloudBees Operations Center. The Cross Team Collaboration feature has a configurable router for routing events and we will change the router used for this exercise. You will need to select a partner to work with - one person will be the notifier and the other person will update their **event-trigger** job to be triggered when the notifier's job is run.

1. First you need to update the **Notification Router Implementation** to use the **Operations Center Messaging** router by clicking on the **Manage Jenkins** link - on the left side at the root of your Team Master (classic ui).
2. Next, scroll down and click on **Configure Notification** link.
3. Under **Notification Router Implementation** select the **Operations Center Messaging** option as opposed to the currently selected **Local only** option.
4. Now, the trigger job of the second partner needs to be updated to listen for the notifier's `event` string - ask your partner what their event string is - and then run the job once to register the trigger.
5. Next, the notifier will run their `notify-event` job and you will see the `notify-trigger` job get triggered.

## Jenkins Kubernetes Agents

In this exercise we will get an introduction to the [Jenkins Kubernetes plugin](https://github.com/jenkinsci/kubernetes-plugin) and use the `container` block to run a set of steps inside a Docker container set-up in a Jenkins Kubernetes Agent template. So far we have used different `labels` to specify different JDK versions. But now we want to use Maven and our Jenkins Administrator has configured a Jenkins Kubernetes plugin template to include access to a maven container.

>**NOTE**: The 'container' declaration does not currently work with the Blue Ocean Pipeline Editor. So the changes for this exercise will need to be done directly on your file in source control. See https://help.github.com/articles/editing-files-in-your-repository/ 

1. Replace the **Testing** stage with the following stage:

```
      stage('Testing') {
        parallel {
          stage('Java 8') {
            agent { label 'jdk9' }
            steps {
              container('maven8') {
                sh 'mvn -v'
              }
            }
          }
          stage('Java 9') {
            agent { label 'jdk8' }
            steps {
              container('maven9') {
                sh 'mvn -v'
              }
            }
          }
        }
      }
```
  You see above that the `container` block runs all of the steps within that block in the specified container based on the **name** of the **Container Template** - so it doesn't matter what version of the JDK is running in the agent.
2. Commit your change to the master branch.
3. Compare the logs for the **Java 8** and **Java 9** stages.

## GitHub Organization Project

In this exercise we are going to create a Jenkins GitHub Organization project from a newly forked repository and the repository you created in **[Setup - Create a GitHub Organization](./Setup.md#create-a-github-organization)**.

First, let's add your GitHub credentials to the Jenkins' Credentials manager:

1. Navigate back to your personal folder in Jenkins
2. Click on **Credentials**
3. Click on **[YourFolderName]** under **Stores Scoped to [YourFolderName]**
4. Click on **Global Credentials (Unrestricted)**
5. Click on **Add Credentials**
6. Fill out the form (**Username with password**)
  - **Username**: The Github organization name
  - **Password**: Your Github personal access token
  - **ID**: Create an ID for your credentials (something like **yourorg-id**)
  - **Description**: Can be left blank if you want
7. Click on **OK**

Now let's create the Github Organization project:

1. Click on **New Item**
2. Enter your organization name as the **Item Name**
3. Select **GitHub Organization**
4. Click **Ok**
5. Select the credentials you created above from the **Credentials** drop down
6. Select **All** from the **Strategy** drop down under **Discover Branches**
7. Click **Save**

Once you click on save, Jenkins will search your organization for any projects with Jenkinsfiles in them, import those projects as Multibranch projects, and begin building each branch with a Jenkinsfile in it.

When the project was created it also should have created a webhook in Github. Verify that the webhooks were created in Github by checking **Webhooks** within your Organization's Github **Settings**.

## Custom Marker Files

In the following exercise we are going to demonstrate how you can use the **[Custom Marker feature](https://go.cloudbees.com/docs/cloudbees-documentation/cje-user-guide/#pipeline-custom-factories)** of CloudBees Jenkins Enterprise to assign pipeline to a job based on an arbitrary file name like pom.xml.

In order to complete the following exercise you will need to fork the following repositories into the Github Organization you created in **[Setup - Create a GitHub Organization](./Setup.md#create-a-github-organization)**:

* https://github.com/PipelineHandsOn/maven-project
* https://github.com/PipelineHandsOn/custom-marker-files

Once that repository is forked:

1. Click on the Github organization project you created in the **[GitHub Organization Project exercise](./Exercise-03.md#github-organization-project)**.
2. Click on **Configure**
3. Under **Project Recognizers** select **Custom Script**
4. In **Marker file** type ```pom.xml```
5. Under **Pipeline** - **Definition** select **Pipeline script from SCM**
6. Select **Git** from **SCM**
7. In **Repository URL** enter: ```https://github.com/{your_org_name}/custom-marker-files```
8. Select the credentials you created in the previous exercise.
9. In **Script path** enter: ```Jenkinsfile-pom```
10. Click on **Save**
11. Click on **Scan Organization Now**

When the scan is complete your **Github Organization** project should now have both the original [empty repository you created in setup](./Setup.md#create-an-empty-github-repository-for-your-first-example-pipeline) and the **maven-project**.

> **NOTE:** The ***custom-marker-files*** repository does not get added to your **Github Organization** project since in doesn't contain a matching marker file: `Jenkinsfile` or `pom.xml`.

## Next Exercises
You may move onto **[Distributed Pipelines with Pipeline As Code](./Exercise-04.md)** once your instructor tells you to.
