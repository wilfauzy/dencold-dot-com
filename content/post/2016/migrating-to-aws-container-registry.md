---
author: "dennis"
date: "2016-05-14T11:53:28-08:00"
draft: false
title: "Migrating to AWS Container Registry"
tags: ["aws", "docker"]
image: "/images/content/2016/aws-ecr-cover.jpg"
share: true
---

I use [Docker](https://www.docker.com/) to manage deployments for my applications and, up until April of 2016, I was using [Docker Hub](https://hub.docker.com) as the registry for all of my images. If you have public images, I recommend this route. However, if you have images that need to be kept private (as I do for my client work) you'll need to pay for the service and it can get [quite pricey](https://hub.docker.com/account/billing-plans/) for many repositories. Thankfully, Docker's registry is [open-sourced](https://github.com/docker/distribution) and the Docker organization has done a fantastic job of documenting the process to host your own registry. But, if you deploy to AWS, there is an even easier way...

### AWS Container Registry announced

In December of 2015 AWS [announced](https://aws.amazon.com/blogs/aws/ec2-container-registry-now-generally-available/) that their Container Registery (ECR) was generally available (in AWS terms, "generally" means "us-east-1"). In March of 2016 they [opened up the service to us-west-2](https://aws.amazon.com/about-aws/whats-new/2016/03/amazon-ec2-container-registry-available-in-us-west-oregon/). This is the region that I deploy my applications on, so I was finally able to make the switch to a managed registry. AWS's [pricing](https://aws.amazon.com/ecr/pricing/) for this service is crazy cheap:

![AWS ECR Pricing](/images/content/2016/aws-ec2-container-registry-pricing.png)

Obviously, YMMV, but I've never had these costs add up to more than a dollar, per-month.

### Some terms...

AWS provisions a registry for you when you create your first ECR repository. If you are asking yourself, "wait, what's the difference between a registry and a repository" like I was, [this stackoverflow post](http://stackoverflow.com/questions/34004076/difference-between-docker-registry-and-repository) is for you. So, the AWS *registry* is analagous to Docker Hub and *repositories* are how your images (and their tags) are organized within the registry. Here is the list of [my public repositories](https://hub.docker.com/r/dencold/) on Docker Hub:

![Docker Hub Repositories](/images/content/2016/docker-hub-repositories.png)

If I wanted to migrate all of those over to AWS, I'd need to explicitly create a repository for each before making a push to the registry. This is different behavior from Docker Hub, which will automatically create a repository for you the first time you push an image.

### Step-by-step instructions

Okay, enough background. Let's get your first repository setup. First, I prefer to do my AWS provisioning using their excellent [aws-cli](https://aws.amazon.com/cli/) tool. If you haven't done so already, make sure to [install the tool](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) via instructions on their site. Additionally, make sure you've run the `aws configure` command and provided an access key that has admin privilages.

For this example, I'm going to show you how I would take my public [pgcli repository](https://hub.docker.com/r/dencold/pgcli/) and bring it over to my private AWS instance.

0. Create the registry using the ecr `create-repository` subcommand (note that the repository name is set to "pgcli", you should change this to be the name of your repository).

	```json
	$ aws ecr create-repository --region us-west-2 --repository-name pgcli
	{
	    "repository": {
	        "registryId": "000042290000",
	        "repositoryName": "pgcli",
	        "repositoryArn": "arn:aws:ecr:us-west-2: 000042290000:repository/pgcli",
	        "repositoryUri": "000042290000.dkr.ecr.us-west-2.amazonaws.com/pgcli"
	    }
	}
	```

0. Make special note of the `repositoryUri` in the json response. At this point I like to take the hostname and store it in an env variable because it's pretty unwieldy (and sadly, AWS does not currently have a way to CNAME/Alias the host). Make sure to replace `000042290000` with your registryId.

	```bash
	$ export AWS_ECR_HOST=000042290000.dkr.ecr.us-west-2.amazonaws.com
	```

0. Find out your registry login(note that `<base64-encoded-auth>` will be a very long byte sequence).

	```bash
	$ aws ecr get-login --region us-west-2
	docker login -u AWS -p <base64-encoded-auth> -e none https://000042290000.dkr.ecr.us-west-2.amazonaws.com
	```
	
	If that looks okay, you can eval it directly to log yourself in via docker:
	
	```bash
	$ $(aws ecr get-login --region us-west-2)
	WARNING: login credentials saved in /Users/coldwd/.docker/config.json
	Login Succeeded
	```
	
0. Tag the latest local version of your image with the full AWS repository name (docker requires this for pushes to private registries).

	```bash
	$ docker tag dencold/pgcli:latest $AWS_ECR_HOST/pgcli:latest
	```
	
0. Finally, you should be able to now push the image up to the repository on your new registry:

	```bash
	$ docker push $AWS_ECR_HOST/pgcli:latest
	The push refers to a repository [000042290000.dkr.ecr.us-west-2.amazonaws.com/pgcli]
	5f70bf18a086: Layer already exists
	15a88e76afed: Layer already exists
	97315b41b490: Pushed
	c57b1a499184: Layer already exists
	cff209dfde52: Layer already exists
	5439b1ec108e: Layer already exists
	61a73490eb04: Layer already exists
	a2e0f03e8793: Layer already exists
	65f3b0435c42: Pushed
	latest: digest: sha256:63216aaa1cd7aa940cc719ffa8b18cb249567906e7c6ba6baa7b1781d6a9d3fd size: 10312
	```

You're done! If you login to your AWS console and click on the "Repositories" section of Amazon ECS, you should see it listed:

![AWS Repositories](/images/content/2016/aws-ecr-repositories.png)

If you drill into the detail page by clicking on the repository name you should see that the hash checksum matches the output from our push:

![AWS Repository Detail](/images/content/2016/aws-ecr-repositories-detail.png)

### Pulling images

You can pull down images from your registry from any host. Only prequisites:

1. Docker is installed.
2. aws-cli is installed.
3. aws-cli is configured with your access key.
4. You've run docker login via the same `$(aws ecr get-login --region us-west-2)` command you ran above.

To pull down that image we pushed in the last step, you'd just need to run:

```bash
$ docker pull 000042290000.dkr.ecr.us-west-2.amazonaws.com/pgcli:latest
```

### Closing thoughts

One disapointment is that AWS gives you no way to name your registry (e.g. `000042290000.dkr.ecr.us-west-2.amazonaws.com`) into something a little friendlier, like `registry.yourdomain.com`. Since there is no way to associate your own TLS certificate with the registry, you'll get certificate errors if you try to CNAME the host:

```bash
$ docker login -u AWS -p REDACTED -e none https://registry.mydomain.com
Error response from daemon: invalid registry endpoint https://registry.mydomain.com/v0/: unable to ping registry endpoint https://registry.mydomain.com/v0/
v2 ping attempt failed with error: Get https://registry.mydomain.com/v2/: x509: certificate is valid for *.dkr.ecr.us-west-2.amazonaws.com, not registry.mydomain.com
v1 ping attempt failed with error: Get https://registry.mydomain.com/v1/_ping: x509: certificate is valid for *.dkr.ecr.us-west-2.amazonaws.com, not registry.mydomain.com
```

This has been raised on the following [AWS form post](https://forums.aws.amazon.com/thread.jspa?messageID=697917&#697917). Hopefully it is something AWS will support in the future.
