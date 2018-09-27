# Declarative Syntax Basics
## Setup the Workshop Environment
If you haven't already, follow the instructions in [Setup.md](Setup.md) before continuing with this exercise.  The Setup lesson will provide the essentials necessary to complete the exercises in this workshop.

## Setup to use Blue Ocean Pipeline Editor
Once you have completed the Setup exercise you will have your own Jenkins Master for the following exercises.  Ensure you are logged into the Jenkins cluster using the username/password you created during the Sign Up process.

**Note**: You need to have a Github personal access token ([Github-Personal-Access-Token.md](Github-Personal-Access-Token.md)) before proceeding.

We will setup the [Blue Ocean Pipeline Editor](https://jenkins.io/doc/book/blueocean/pipeline-editor/) so we can use the visual editor for many of following exercises:

1. If not already in Blue Ocean, click on the **Open Blue Ocean** link in the left navigation bar
2. Click on the **Create a new Pipeline** button (if you have already created a Pipeline, you will need to click the **New Pipeline** button in the upper right) <p><img src="img/1-0-create-a-new-pipeline.png" width=300/>
3. Click on one of the options in the **Where do you store your code?** section (**GitHub** for this course) <p><img src="img/1-0-select-scm.png" width=300/>
4. Enter your **GitHub token** (your GitHub Personal Access Token)
5. Next, select your GitHub user account (NOTE: Be sure to select the GitHub Organization where you created the empty repository for this workshop. The repository should not have an existing Jenkinsfile.) <p><img src="img/1-0-select-github-org.png" width=300/>
6. Under **Choose a repository** select the repository you created during setup and then click the **Create Pipeline** button <p><img src="img/1-0-choose-repo-create-pipeline.png" width=420/>

Next, the Blue Ocean editor will open - if the editor does not load after a minute or so, refresh your browser. 

## Basic Declarative Syntax Structure

In this exercise we will create a simple Declarative Pipeline using the Blue Ocen Pipeline editor.

Declarative Pipelines must be enclosed within a `pipeline` block and must contain a top-level `agent` declaration, and then contains exactly one `stages` block. The `stages` block must have at least one `stage` block but can have an unlimited number of additional stages. Each `stage` block must have exactly one `steps` block. The Blue Ocean editor takes care of much of this for you but we will need to add a `stage` and `steps`.

Using the Blue Ocean Pipeline editor we setup in the previous exercise, do the following:

1. You should see the **You don't have any branches that contain a Jenkinsfile** dialog, click on the **Create Pipeline** button (NOTE: If you already had a `Jenkinsfile` in your repository then the editor should open straight-away) <p><img src="img/1-1-create-pipeline-no-jenkinsfile.png" width=300/>
2. Click on the **+** icon next to the pipeline's **Start** node to add a `stage` <p><img src="img/1-basic-syntax-add-stage.png" width=400/>
3. Click into **Name your stage** and type in the text 'Say Hello'
4. Click on the **+ Add step** button <p><img src="img/1-basic-syntax-add-step.png" width=400/>
5. Click on the **Print Message** step and type in ***Hello World!*** as the **Message**
6. Click on the **<-** (arrow) next to the **'Say Hello / Print Message'** text to add another step <p><img src="img/1-1-print-message-then-add-step.png" width=300/>
7. Click on the **+ Add step** button
8. Click on the **Shell Script** step and enter `java -version` into the text area
9. Press the key combination `CTRL + S` to open the Blue Ocean free-form editor and you should see a Pipeline similar to the one below **(click anywhere outside the editor to close it)**:

```
pipeline {
   agent any
   stages {
      stage('Say Hello') {
         steps {
            echo 'Hello World!'   
            sh 'java -version'
         }
      }
   }
}
```

10. Click the **Save** button <p><img src="img/1-1-shell-step-save.png" width=350/>
11. Enter a commit message into the **Save Pipeline** pop up and click **Save & Run** <p><img src="img/1-basic-syntax-save-run.png" width=350/>

>Your Pipeline is actually being committed and pushed to your GitHub repository, and will run right away.

12. Click on your Pipeline to see the `steps` execute. <p><img src="img/1-1-click-to-see-pipeline-run.png" width=400/>
13. Expand the **'java -version — Shell Script' `step` and you should see the following: <p><img src="img/1-1-java-version-step-expanded.png" width=420/>
  
>NOTE: You may have noticed that your Pipeline GitHub repository is being cloned even though you didn't specify that in your Jenkinsfile. Declarative Pipeline checks out source code by default.

## Agent Labels

In this exercise we will update the pipeline we created in the previous exercise to use a specific `agent` using the `label` syntax. As you saw from the build logs of the previous exercise, the Java version of the `agent any` was 8. We want to update our pipeline to use a version 9 JDK by replacing the `any` parameter with a `label` declaration:

1. Click on the **pencil** icon in the top right to edit your Pipeline <p><img src="img/1-2-click-pencil.png" width=520/>
2. Use the key combination `CTRL + S` to open up the free-form editor
>NOTE: You could use the visual editor but it doesn't support the short syntax for adding a label and requires you to add a `node` block and then apply a `label`
3. Replace the `agent any` declaration with the following `agent` declaration:

>NOTE: Don't worry about formatting too much as the Blue Ocean editor will reformat your Jenkinsfile before it is committed to your repository

```
  agent {
    label 'jdk10'
  }
```

4. Click the **Update** button
5. Click the **Save** button, enter a commit message into the **Save Pipeline** pop up and click **Save & Run**. 

You should see the following output for the `sh` step after your Pipeline runs:

```
openjdk version "10.0.4"
OpenJDK Runtime Environment (build 10.0.4+12-Debian-4)
OpenJDK 64-Bit Server VM (build 10.0.4+12-Debian-4, mixed mode)
```

In addition to `any` and `label` you may also specify `none` and no global agent will be allocated for the entire Pipeline run and each `stage` section will need to contain its own `agent` section.

6. Now change the value of the `label` to **jdk8**

```
   agent {
     label 'jdk8'
   }
```

7. Click the **Save** button, enter a commit message into the **Save Pipeline** pop up and click **Save & Run**. 

You should see the following output for the `sh` step after your updated Pipeline runs:

```
openjdk version "1.8.0_151"
OpenJDK Runtime Environment (IcedTea 3.6.0) (Alpine 8.151.12-r0)
OpenJDK 64-Bit Server VM (build 25.151-b12, mixed mode)
```

Your Pipeline should look like the following before you move onto the next exercise:

```
pipeline {
   agent { label 'jdk8' }
   stages {
      stage('Say Hello') {
         steps {
            echo 'Hello World!'   
            sh 'java -version'
         }
      }
   }
}
```

## Environment Directive

For this exercise we are going to update our pipeline to demonstrate how to use the `environment` directive to set and use environment variables. We will also see how this directive supports a special helper method `credentials()` that allows access to pre-defined Jenkins Credentials based on their id.

### Simple Environment Variables

1. At the top of the pipeline insert the following code between the `agent` and `stages` blocks:  

```
   environment {
      MY_NAME = 'Mary'
   }
```

2. Then update the `echo 'Hello World!'` line to read `echo "Hello ${MY_NAME}!"`
>Notice the change from `''` to `""`.  Using double quotes will trigger extrapolation of environment variables.
3. Click the **Update** button
4. Click the **Save** button, enter a commit message into the **Save Pipeline** pop up and click **Save & Run**.
5. Click on your Pipeline to see the `steps` execute.

>NOTE: The Blue Ocean Editor will move global `environment` blocks after stages. The location does not matter as long as it is outside the `stages` block.

### Credentials

We can also use environmental variables to import credentials.

1. Add the following line to our `environment` block:

```TEST_USER = credentials('test-user')```

>NOTE: For Credentials which are of type "Standard username and password", the environment variable specified will be set to username:password and two additional environment variables will automatically be defined: `MYVARNAME_USR` and `MYVARNAME_PSW` respectively.
2. Add the following ```echo``` steps within the ```steps``` of the **Say Hello** ```stage```:

```
            echo "${TEST_USER_USR}"
            echo "${TEST_USER_PSW}"
```

3. Click the **Update** button
4. Click the **Save** button, enter a commit message into the **Save Pipeline** pop up and click **Save & Run**.  
5. Click on your Pipeline to see the `steps` execute. Look at the console output and make note of the fact that the credential user name and password are masked when output via the `echo` command.

## Parameters

In this exercise we will update our pipeline to accept external input in the form of a Jenkins job parameter.

1. Update your pipeline by inserting the following  after the ```environment``` block:

```
   parameters {
      string(name: 'Name', defaultValue: 'whoever you are', 
	     description: 'Who should I say hi to?')
   }
```

2. Update the ```echo "Hello ${MY_NAME}!'``` line to read ```echo "Hello ${params.Name}!"```
3. Click the **Save** button, enter a commit message into the **Save Pipeline** pop up and click **Save & Run**. 
>NOTE: The option to ***Build with parameters*** will not be available until you run your job at least once as parameters are Jenkins job properties that must be set in Jenkins. Using `parameters` in your Jenkinsfile updates the Jenking job configuration after the first run. 
4. Run the pipeline again by returning to the Blue Ocean activity view and then clicking on the **Branches** tab top right and then click the run button.<p><img src="img/1-param-run.png" width=520/>
5. After clicking the run button an **Input required** dialog will appear with the **Name** paramter. Replace **whoever you are** with your name and click the **Run** button.<p><img src="img/1-param-input-required.png" width=480/>
6. Now your top **Print Message** step should reflect your parameter value similar to this for a value of ***Beedemo Dev***: <p><img src="img/1-param-echo.png" width=550/>

## Next Exercises
You may proceed onto **[Declarative Advanced Syntax](./Exercise-02.md)** once your instructor tells you to.

Before you proceed you may want to check that your Pipeline looks like the following:

```
pipeline {
  agent {
    label 'jdk8'
  }
  stages {
    stage('Say Hello') {
      steps {
        echo "Hello ${params.Name}!"
        sh 'java -version'
        echo "${TEST_USER_USR}"
        echo "${TEST_USER_PSW}"
      }
    }
  }
  environment {
    MY_NAME = 'Mary'
    TEST_USER = credentials('test-user')
  }
  parameters {
    string(name: 'Name', defaultValue: 'whoever you are', description: 'Who should I say hi to?')
  }
}
```
