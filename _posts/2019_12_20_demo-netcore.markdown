---
layout: post
title: Demo Net core
description: Create a ASP.NET Core project then deploy to Heroku server via CircleCI continuous integration
date: 2019-12-20 10:59:00 +0700
categories: netcore
thumbnail: https://www.google.com/imgres?imgurl=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fdotnet%2Fimages%2Fhub%2Fnet-docs-dotnet-core.svg&imgrefurl=https%3A%2F%2Fdocs.microsoft.com%2Fvi-vn%2Fdotnet%2Fcore%2Fget-started&tbnid=RoisMD66yNId0M&vet=12ahUKEwjLtfq3s8PnAhV8x4sBHSAxDDsQMygBegUIARDTAQ..i&docid=GnYVx-Hp6bQG7M&w=800&h=280&q=net%20core&ved=2ahUKEwjLtfq3s8PnAhV8x4sBHSAxDDsQMygBegUIARDTAQ
---

#### Criteria
- Help you choose a solution modern, cross-platform, automation to build your project
- For small-scale applications, everything is free, so good for study and make a demo
- Your application can access online, you will feel realistic and can share your ideas

### Keywords
- ASP.NET Core
- Docker container
- Platform as a Service
- Heroku
- Continuous integration
- CircleCI

#### Before starting, you learn a little bit about concepts with these keywords. If you don't know it yet, please read some reference articles and discuss with friends
- ASP.NET Core: is a cross-platform, high-performance, open-source framework for building modern, cloud-based, Internet-connected applications.\
https://docs.microsoft.com/en-us/aspnet/core/?view=aspnetcore-2.2

- Docker container: is a platform for developers and sysadmins to build, share, and run applications with containers. The use of containers to deploy applications is called containerization.\
https://docs.docker.com/get-started/

- Platform as a Service: that provides a platform allowing customers to develop, run, and manage applications without the complexity of building and maintaining the infrastructure typically associated with developing and launching an app.\
https://en.wikipedia.org/wiki/Platform_as_a_service

- Heroku: is a container-based cloud *Platform as a Service (PaaS)*. Developers use Heroku (as a server/hosting) to deploy, manage, and scale modern apps.\
https://www.heroku.com/about

- Continuous integration (CI): is a practice that encourages developers to integrate their code into a master branch of a shared repository early and often.

- CircleCI: is a platform enables engineering teams with automation and *continuous integration*. CircleCI automates your software builds, tests, and deployments.\
https://circleci.com/docs/2.0/about-circleci/#section=welcome

#### Now, when you already know about the concepts, you will use Heroku to host and run your ASP.NET Core application via CircleCI continuous integration:
You will have Docker pack the web application inside of the container, and then use CircleCI to deploy the Docker container to Heroku Container Registry.


## Setup your local development

### 1. ASP.NET Core running on .NET Core 
- This demo is version 2.2, but you can use latest version.
- Download installers of SDK and Runtimes compatible with your operating system:
https://dotnet.microsoft.com/download/dotnet-core/2.2

### 2. Docker container
- Download docker desktop compatible with your operating system:\
https://docs.docker.com/docker-for-windows/install/\
https://docs.docker.com/docker-for-mac/install/

### 3. Your project idea
- Do you have any ideas? 
- Let's do it to leaning and creativily!

## Steps


### 1. You can get directly by using Fork the complete code of this article on GitHub to become your project that will be deployed:
https://github.com/duycs/demo-netcore

### 2. If don't fork the article's project, you can create your project
#### 2.1 Create new project ASP.NET Core
- Create a new folder for your project.
- At the folder, open Command/Terminal then run this command to create a template webapi project:
```
  dotnet new webapi
```
- The project's name is named as folder name, you can use Visual-Studio-Code tool to open the folder then start development. 

#### 2.2 Dockerize ASP.NET Core application
- To dockerize your application, you need to create new Dockerfile in your project folder.
- Here is a complete Dockerfile looks like:

```docker
# Grab an image with a small OS image made for .Net Core
FROM microsoft/dotnet:2.2-sdk AS build-env
# Working directory is /app
WORKDIR /app

# Copy csproj and restore as distinct layers
COPY *.csproj ./
RUN dotnet restore

# Copy everything else and build
COPY . ./
RUN dotnet publish -c Release -o out

# Build runtime image
FROM microsoft/dotnet:2.2-aspnetcore-runtime
WORKDIR /app
COPY --from=build-env /app/out .

# ENTRYPOINT ["dotnet", "demo-net-core.dll"]
# heroku uses the following
CMD ASPNETCORE_URLS=http://*:$PORT dotnet demo-net-core.dll
```
- At this article, project name is "demo-net-core", so you should change this name to your project name at Dockerfile

#### 2.3 Config CI/CD pipeline
- Create a folder .circleci in the root folder of your repository.
- Create a new file named config.yml inside that folder.
- Here is a complete config.yml looks like:

```
version: 2
jobs:
 build:
   machine: true
   steps:
     - checkout 

     # build image
     - run: |         
         docker info
         docker build -t aspnetapp -f Dockerfile .
     # deploy the image
     - run: |         
         docker login --username=$HEROKU_USERNAME --password=$HEROKU_API_KEY registry.heroku.com
         docker tag aspnetapp registry.heroku.com/$HEROKU_APP_NAME/web
         docker push registry.heroku.com/$HEROKU_APP_NAME/web                
         curl https://cli-assets.heroku.com/install.sh | sh
         heroku container:release web -a $HEROKU_APP_NAME
```

### 3. Create your Heroku application
- Log in to Heroku or Sign up to create a new account if you don’t have one:
https://id.heroku.com/login
- After that, create a new app, choose the name for your application and select the desired region.
- Save the name for later, we will use it with CircleCI.

### 4. Set up your project on CircleCI with Heroku
- Login to CircleCI with your Github:
https://circleci.com/vcs-authorize/
- Then grant authorization your Github Repository for CircleCI
- On CircleCI dashboard, click ADD-PROJECTS, chooe your project which you want to deploy on Heroku, click Set-Up-Project.
- At Set up project, choose Operating System is Linux then click Start-building
- After that is done, you will need to set up environment variables for your CircleCI project at Setting:
```
HEROKU_API_KEY: You can find your Heroku API key in your Heroku Account settings
HEROKU_USERNAME: Your username should be your email that you use to sign in
HEROKU_APP_NAME: The name that you used when you were creating new Heroku application.
```

### 5. Coding and push to master branch, you will see *Continuous-integration*
- Coding and push to master branch, after a few minutes, go to your Heroku application domain and enjoy your change!
- The article's app can be found at: https://duycs-demo-net-core.herokuapp.com/

# References
- https://circleci.com/docs/2.0/configuration-reference/
- https://devcenter.heroku.com/categories/reference
- https://developer.okta.com/blog/2019/09/18/build-a-simple-dotnet-core-app-in-docker
- https://softchris.github.io/pages/dotnet-dockerize.html#build-our-image-start-container
- https://codingblast.com/hosting-asp-net-core-on-heroku-with-dockercircleci-for-free/
---
