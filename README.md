# BitbucketPipelines-ECS
A Bitbucket Pipelines yaml for deploying to AWS ECS developed at Sahaj Software

This repo was built for use with a Medium article. For a full explanation of the project, please read [Using Bitbucket Pipeline For AWS ECS Deployments](https://medium.com/inspiredbrilliance/using-bitbucket-pipeline-for-aws-ecs-deployments-1a914960f050)

## Setting up AWS access

Bitbucket Pipelines has a setup that makes working with AWS rather straight forward. All you have to do is set up three environment variables and it will automatically run all CLI commands on your account. Go to Setting -> Pipelines -> Environment variables. You’ll have to set **AWS_ACCESS_KEY_ID, AWS_DEFAULT_REGION, AWS_SECRET_ACCESS_KEY**. We will additionally setup **AWS_REGISTRY_URL** since we are using ECS. This covers all that is needed to start deploying to our main account (for us this is dev since most deployments will go to dev). For our purposes, we had to create these variables for the production account as well.

## The Script

The main parts that are done in the script are as follows:

**AWS ECR login**: This will retrieve the login command to use to authenticate your Docker client to your registry.

**Export Enviornment Variables**: Here you will set the service, cluster, task, execrole, and build ID to be used throughout the pipeline.

**Docker Build, Push**: These are your standard Docker commands to push the image to your ECR repository you’ve logged into.

**Stop Current Task**: We export the current running task and then stop it so that we can run a new task. This is done in my build specifically because I only allow 1 task for my service. This part can be changed based on your needs.

**Register New Task**: Register the new task you will build. My task is built with many of the standards to include port 5000 for Python Flask and deploys it’s docker logs to CloudWatch. Before it can deploy this way you must have gone to CloudWatch and created a new log group of the same name.

**Update Service**: This deploys the new task to the service you’ve listed above and begins the deployment of a new image to the EC2 instance.

Once these steps have finished running through Bitbucket Pipelines, the Target Group associated with the ECS Service will drain, run initial health checks, then be fully accessible.

## Final Steps

The final step after you’ve figured out how to properly run this for one script is to repeat it four times. In my case, I additionally added in a Blue/Green strategy for my development environment. Note that once you do complete this, you will want to save the code as bitbucket-pipelines.yml at the root level so that Pipelines will pick it up. Then, enable pipelines on Bitbucket under Settings-> Pipelines -> Settings

Now every time you push your code to dev it will automatically build and deploy both your dashboard and your API. Every time you merge a pull request to master it will build and deploy your code to your production environment.

## Enjoy!
It’s an open source project. If you add improvements, contribute! 
