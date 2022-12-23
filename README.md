In this tutorial, we will look at deploying our app at Spring Azure app. Spring Azure App is an Azure offering to seamlessly deploy Spring boot microservices at Azure. For this article, we hope you have an active Azure subscription. At the time of writing this article, Azure provides a one-year subscription free of cost.

# Agenda

* Create a spring boot app and add a hello endpoint
    
* Create Resource Group, Service and App at Azure Portal
    
* Deploy Application at Azure using Command Line (Azure cli)
    
* Use GitHub Action to deploy on Azure
    

If you want to follow the video tutorial of the app you can use it at Youtube below.

%[https://www.youtube.com/watch?v=uliQ37J8jU4] 

### Create a spring boot app and add a hello endpoint

We will use [https://start.spring.io](https://start.spring.io) . We are using maven as the build tool and 2.7.6 as version . We will use Java 17 as our Java version. Add web dependency and hit download.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671804920603/9e68acdb-b822-4073-9a44-67aa3a0f7572.png align="center")

Open the project in your favourite IDE, We are using IntelliJ. Add the Hello endpoint and run the project.

```java
package com.kp.springdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class SpringdemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringdemoApplication.class, args);
	}
	@GetMapping("hello/{name}")
	public  String hello(@PathVariable ("name")  String name){
		return "Hello "+name +" from Azure";
	}
}
```

Now we can run the project and verify if everything is working as expected. We can check with http://localhost:8080/hello/Kumar in browser , it should return `Hello Kumar from Azure`

Build the project using `mvn clean package` , and it will keep generate the war file at the target folder inside your project.

## Create Resource Group, Service and App at Azure Portal

Login to the azure portal and click on resource group, add a resource group.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671805568130/1223fbed-0e1b-41d7-920b-81c060e3abbf.png align="center")

If you wish you can use Azure cli to do same . Use the following command.

```bash
az group create --resource-group <YOUR_RESOURCE_GROUP_NAME>  --location eastus
```

Once created we will add Spring Azure Service inside this resource group so that we can create Spring application inside the service.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671805845945/5d501afc-7e6c-42a8-9c58-d38e6d677568.png align="center")

Alternatively following command can be used at Azure cli.

```bash
az spring spring create   --resource-group  <YOUR_RESOURCE_GROUP_NAME>  --name <YOUR_SPRING_AZURE_SERVICE_NAME> 
```

Now we can create a Spring App inside the service. Azure will deploy a sample Spring app. We will replace the app with thw one we have coded.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671806386691/a7ba4f73-3598-4d88-a028-a92d91c6f2d2.png align="center")

Once deployment is complete it will have a app as shown in picture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671808704360/5cd45681-e8ab-434d-a18c-9fe88012eec8.png align="center")

Alternatively, we can create this all using Azure CLI.

```bash
az spring app create <YOUR_RESOURCE_GROUP_NAME>  --name <YOUR_SPRING_AZURE_SERVICE_NAME>  --name <YOUR_SPRING_AAPP_NAME>  --assign-endpoint true
```

## Deploy Application at Azure using Command Line (Azure cli)

  
Now let us go to the target folder where our jar file is created while building via maven. We will run the following command to deploy the app. It will deploy our app rather the sample spring azure app.

```bash
az spring app deploy -n springdemo -g codingsaint -s codingsaintsvc --artifact-path springdemo-0.0.1-SNAPSHOT.jar
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671809116479/1e989fb7-f449-4125-af96-1bfaa0b6f7ef.png align="center")

We can go to the assigned endpoint and check the deployed app. At present the URL (if assigned , you can see "Assign URL" link when you login to Azure and check your app) gets created as https://YOUR\_SERVICENAME-YOUR\_APPNAME.azuremicroservices.io .

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671809166025/41b0c323-dc7f-425f-90f1-0ce304d3cfba.png align="center")

## Use GitHub Action to deploy on Azure

It is not ideal to run deploy command every time we deploy. Let's use Github Action to deploy

Create .github folder inside source code , inside this folder create a workflow folder. Add Azure deployment yaml inside it.

```yaml
name: AZSpringDemo
on: push
env:
  ASC_PACKAGE_PATH: ${{ github.workspace }}
  JAVA_VERSION: 17
  AZURE_SUBSCRIPTION: 29f6fde1-6736-4a36-b820-c879ba4868d1

jobs:
  deploy_to_production:
    runs-on: ubuntu-latest
    name: deploy to production with artifact
    steps:
      - name: Checkout Github Action
        uses: actions/checkout@v2

      - name: Set up JDK ${{ env.JAVA_VERSION }}
        uses: actions/setup-java@v1
        with:
          java-version: ${{ env.JAVA_VERSION }}

      - name: maven build, clean
        run: |
          mvn clean package -DskipTests
      - name: Login via Azure CLI
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: deploy to production with artifact
        uses: azure/spring-cloud-deploy@v1
        with:
          azure-subscription: ${{ env.AZURE_SUBSCRIPTION }}
          action: Deploy
          service-name: codingsaintsvc
          app-name: springdemo
          use-staging-deployment: false
          package: ${{ env.ASC_PACKAGE_PATH }}/**/*.jar
```

The YAML has name of the app and the service under which it is deployed. It also has a command under `run` to build the application. We need to add AZURE\_CREDENTIAL so that whenever we push code to git, it logs in to the Azure portal and deploys our app.

Run the following command at azure cli to create Authentication to be used for github.  

```bash
az ad sp create-for-rbac --name "CICD" --role contributor \ --scopes /subscriptions/{SUBSCRIPTION_ID/resourceGroups/{YOUR_RESOURCE_GROUP_NAME} \ --sdk-auth
```

This will generate the JSON. Copy it.

Log in to Github , and create a repository to push our code. Go to Setting Tab. Under Settings go to secret and Add AZURE\_CREDENTIAL inside it.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671809859340/5d88eb7c-dbd4-4715-aaf4-b795554433f1.png align="center")

Now we can save to JSON generated while running the command. Now we can commit our code to it. As the YAML for pipeline suggest it should trigger the build for you.

You can verify the build under Action tab.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671810046139/00e10c7d-d4f3-4410-9205-6699b2fd310d.png align="center")

If everything goes fine it will deploy the app at azure.
