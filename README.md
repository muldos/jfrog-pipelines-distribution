# JFrog Distribution Pipeline Sample

This repository shows an example of a [JFrog Pipeline](https://www.jfrog.com/confluence/display/JFROG/JFrog+Pipelines) that gets triggered just by adding a given property to an artifact stored in a [JFrog Artifactory](jfrog.com/confluence/display/JFROG/JFrog+Artifactory) repository, and that distributes such artifact to a [JFrog Edge](https://www.jfrog.com/confluence/display/JFROG/JFrog+Artifactory+Edge) node and/or to another JFrog Artifactory instance. 

A real use case for this could be to produce a release, when someone with limited tech skills just update a release notes document.

<br/>

   <img src="https://github.com/muldos/jfrog-pipelines-distribution/raw/main/images/rb-pipeline-overview.png" alt="Resource Bundle Creation Pipeline" width="100%" style="margin: 20px;"/>

<br/>

The `pipeline.yaml` definition file defines two connected pipeline instances:

1. One to create and sign a Release Bundle when a given property is added to an artifact.
   
   Each run of this pipeline is executed serially to avoid a current known issue with resources state across runs.   
      
   <img src="https://github.com/muldos/jfrog-pipelines-distribution/raw/main/images/pipeline01.png" alt="Resource Bundle Creation Pipeline" width="100%" style="margin: 20px;"/>
   

2. the other one to distribute the release bundle to the targeted edge node (defined by the `DistributionRule` resource)   
   
   This pipeline is triggered by the `droAppSignedBundle` resource output created by the previous pipeline. If nodes are available, this pipeline can have multiple runs executed in parallel.   
     
   <img src="https://github.com/muldos/jfrog-pipelines-distribution/raw/main/images/pipeline02.png" alt="Distribution Pipeline" width="300px" style="margin: 20px;"/>


---

### How to configure this pipeline 


1. Fork this repository

Then create the following Integrations 

  <img src="https://github.com/muldos/jfrog-pipelines-distribution/raw/main/images/integrations.png" alt="AllIntegrations" width="400px" style="margin: 20px;"/>

- dro_github is a [Github Intergation](https://www.jfrog.com/confluence/display/JFROG/Github+Integration), that will be used to interact and sync your pipeline YAML defintion from your github repository
- droDistribution is a [Distribution integration](https://www.jfrog.com/confluence/display/JFROG/Distribution+Integration) 
- droSolengRT is an [Artifactory integration](https://www.jfrog.com/confluence/display/JFROG/Artifactory+Integration), used by the AQL resources to know which artifactory instance to use to perform the query.
- triggerDroBundle is an [IncomingWebhook integration](https://www.jfrog.com/confluence/display/JFROG/Incoming+Webhook+Integration) which provides an url to trigger pipeline #1.


1.2. Define an [Incoming Webhook](https://www.jfrog.com/confluence/display/JFROG/Incoming+Webhook+Integration) under "Admin > Pipelines > Integrations" with a name that matches the value of `webhookName` of resource `droDocWebhook` (e.g. `triggerDroBundle`)
  
    This is the mechanism that will be used to trigger the pipeline when a new artifact is uploaded into an Artifactory repository, or in our case when a property is added to an artifact.
  
 
   <img src="https://github.com/muldos/jfrog-pipelines-distribution/raw/main/images/webhook01.png" alt="WebhookIntegration1" width="500px" style="margin: 20px;"/>

  
1.3. Define the [generic webhook](https://www.jfrog.com/confluence/display/JFROG/Webhooks) that triggers the pipeline  

   This webhook invokes the URL of the "Incoming Webhook" defined in the previous step.   
   Notice the "authorization" field added as a custom header, which should match the `authorization` value configured for the incoming webhook above. Use the `Test` button to make sure that the integration works.    

   <img src="https://github.com/muldos/jfrog-pipelines-distribution/raw/main/images/webhook02.png" alt="WebhookIntegration1" width="500px" style="margin: 20px;"/>
   
   Now, you should be in this state 
   
   <img src="https://github.com/muldos/jfrog-pipelines-distribution/raw/main/images/pipeline-trigger.png" alt="WebhookIntegration1" width="500px" style="margin: 20px;"/>

1.4. Define an Artifactory Integration, and a Distribution Integration, as described in our documentation.

   <img src="https://github.com/muldos/jfrog-pipelines-distribution/raw/main/images/artifactory-int.png" alt="ArtifactoryIntegration" width="500px" style="margin: 20px;"/>
   <img src="https://github.com/muldos/jfrog-pipelines-distribution/raw/main/images/distribution-int.png" alt="DistributionIntegration" width="500px" style="margin: 20px;"/>

They will be used respectively by the AQL Resource and the DistributionRule resource


4. Update custom fields in `pipelines.yaml` to match your environment  

   - Update the `query` parameter of the `propsSelectorQuery` resource to match the repository and the file path pattern of the artifacts to be included in the created resource bundle. Ideally, such files should match the ones that trigger the webhook in the step above.   

   - Update resource `droBundleDistributionRules` with your targeted edge nodes to distribute the release bundles to


5. Add your git repo from (1) as a [pipeline source](https://www.jfrog.com/confluence/display/JFROG/Pipelines+Step-By-Step#PipelinesStep-By-Step-add-pipeline-sourceAddaPipelineSource)  

   Once the new pipeline source is successfully resolved, your pipelines should be listed under "Pipelines > My Pipelines" in Artifactory.  



6. Trigger the first pipeline by uploading an artifact to the expected repository, and then add a property to it, with the name pattern expected in step 1    

   The pipeline will create and sign a release bundle for all the artifacts with the same property of other repositories, and then will trigger the second pipeline to distribute the release bundle to the targeted edge nodes. 

---   




