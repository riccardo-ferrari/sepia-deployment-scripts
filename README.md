# Sepia Deployment Scripts

Scripts to setup a continuous deployment pipeline on AWS.

## Prerequisites

- [Dockerhub account]()
- [AWS account]()
- [Github Account]()
- [Travis Account]()

 
## Initial setup 
 
Sample repositories that can be used to set up a sample pipeline:

Fork these repositories:

- [Sample Liferay application](https://github.com/liferay-labs/liferay-game)
- [Deployment definitions for the sample application](https://github.com/liferay-labs/liferay-game-deployment)

Then, add your forked liferay-game repository to your Travis account and make 
sure you set the following environment variables (keep them not visible):

| Variable              | Definition                                                  |
|-----------------------|-------------------------------------------------------------|
| AWS_ACCOUNT_ID        | The ACCOUNT ID of your AWS account                          |
| AWS_ACCESS_KEY_ID     | Your AWS ACCESS KEY ID                                      |
| AWS_SECRET_ACCESS_KEY | Your AWS SECRET ACCESS KEY                                  |
| DOCKER_ORG            | Your public organization in Dockerhub                       |
| DOCKER_USER           | Your Dockerhub user                                         |
| DOCKER_PWD            | Your Dockerhub password                                     |
| DOCKER_AUTH_TOKEN     | Your Docker Auth Token                                      |
| GITHUB_TOKEN          | A Github personal access token with repo access to your liferay-game-deployment repository |
| GITHUB_USER           | The Github user of your liferay-game-deployment repository  |

To obtain the value for DOCKER_AUTH_TOKEN follow the steps described in [Using Images from a Private Repository](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker.container.console.html#docker-images-private).
The value you need to enter is referred on that link as `auth_token`.
If you are running Docker on MAC, in order to get the right value of the token make sure you have configure Docker NOT to store docker logins in macOS keychain.
To do so, got to Docker > Preferences and uncheck the option "Securely store docker logins in macOS keychain".
On a terminal execute:

```
docker logout
docker login -u<DOCKER_USER> -p<DOCKER_PWD>
```

The value of the DOCKER_AUTH_TOKEN can be extracted from ~/.docker/config.json
```
{
  "auths" : {
    "https://index.docker.io/v1/" : {
      "auth" : "auth_token"
    }
  }
}
```

To obtain the value for `GIHUB_TOKEN` follow these steps: [Creating a token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/#creating-a-token).

## Creating and updating the pipeline

In Travis, launch a first build of your liferay-game repository. Apart from 
running the tests, it will automatically build and publish a Docker image to 
your Dockerhub organization. Your liferay-game-deployment repository will be 
updated with the version of the recently generated Docker image. Then, all the 
elements of your continuous delivery pipeline will be automatically generated in 
your AWS account. Finally, deployments and tests of your Docker image will 
execute on each environment of your pipeline, until it is ready for production.
 
Now perform any change in the code of the liferay-game and push it to your
Github master branch. The same process is repeated, except for the generation of
the delivery pipeline in AWS, which is skipped because it was created during the
first build.

## Creating and updating a cloudformation stack

To create or update a stack just execute the command

```
aws/deployment-cloudformation/create-or-update-stack.sh -c CONFIG_DIR
```

where ```CONFIG_DIR``` is the absolute path to a directory that should follow the structure:

```
   - CONFIG_DIR
     - params
          *.json
     - templates
          template.json
```

where the json files located under the ```params``` directory should follow the
 [structure](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html):

```
[
   {
     "ParameterKey": "string",
     "ParameterValue": "string",
     "UsePreviousValue": true|false,
     "ResolvedValue": "string"
   }
   ...
 ]
```

A new stack will be created for every ```params/*.json``` file with the name ```stack-CONFIG_DIR_NAME-JSON_FILE_NAME``` and the content of the templates/template.json file.

To know more about to how write a AWS stack, please read the
 [AWS Cloudformation documentation](https://aws.amazon.com/es/cloudformation/)