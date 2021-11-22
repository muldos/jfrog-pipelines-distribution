# JFrog Distribution Pipeline Sample

This repository shows an example of a [JFrog Pipeline](https://www.jfrog.com/confluence/display/JFROG/JFrog+Pipelines) that gets triggered by the upload of an artifact to a local [JFrog Artifactory](jfrog.com/confluence/display/JFROG/JFrog+Artifactory) repository and that distributes such artifact to a [JFrog Edge](https://www.jfrog.com/confluence/display/JFROG/JFrog+Artifactory+Edge) node and/or to another JFrog Artifactory instance. 

<br/>

   <img src="https://github.com/lsilvapvt/jfrog-pipelines-distribution/raw/main/images/DistributionPipeline.png" alt="Resource Bundle Creation Pipeline" width="100%" style="margin: 20px;"/>

<br/>

The `pipeline.yaml` definition file defines two connected pipeline instances:

1. one to create and sign a Release Bundle for the uploaded artifact  
   
   Each run of this pipeline is executed serially to avoid a current known issue with resources state across runs.   
      
   <img src="https://github.com/lsilvapvt/jfrog-pipelines-distribution/raw/main/images/pipeline01.png" alt="Resource Bundle Creation Pipeline" width="100%" style="margin: 20px;"/>
   

2. the other one to distribute the release bundle to the targeted edge node (defined by the `DistributionRule` resource)   
   
   This pipeline is triggered by the `debSignedBundle` resource output created by the previous pipeline. If nodes are available, this pipeline can have multiple runs executed in parallel.   
     
   <img src="https://github.com/lsilvapvt/jfrog-pipelines-distribution/raw/main/images/pipeline02.png" alt="Distribution Pipeline" width="300px" style="margin: 20px;"/>


---

### How to configure this pipeline 


1. Fork this repository

2. Define an [Incoming Webhook](https://www.jfrog.com/confluence/display/JFROG/Incoming+Webhook+Integration) under "Admin > Pipelines > Integrations" with a name that matches the value of `webhookName` of resource `debWebhook` (e.g. `acmeDebDistribute`)    
  
    This is the mechanism that triggers the pipeline when a new artifact is uploaded into an Artifactory repository.   
  
   <img src="https://github.com/lsilvapvt/jfrog-pipelines-distribution/raw/main/images/webhook01.png" alt="WebhookIntegration1" width="500px" style="margin: 20px;"/>
  

3. Define the [generic webhook](https://www.jfrog.com/confluence/display/JFROG/Webhooks) that triggers the pipeline  

   This webhook invokes the URL of the "Incoming Webhook" defined in the previous step.   
   Notice the "authorization" field added as a custom header, which should match the `authorization` value configured for the incoming webhook above. Use the `Test` button to make sure that the integration works.    

   <img src="https://github.com/lsilvapvt/jfrog-pipelines-distribution/raw/main/images/webhook03.png" alt="WebhookIntegration2" width="600px" style="margin: 20px;"/>


4. Update custom fields in `pipelines.yaml` to match your environment  

   - Update the `query` parameter of the `debAql` resource to match the repository and the file path pattern of the artifacts to be included in the created resource bundle. Ideally, such files should match the ones that trigger the webhook in the step above.   

   - Update resource `debDistributionRules` with your targeted edge nodes to distribute the release bundles to


5. Add your git repo from (1) as a [pipeline source](https://www.jfrog.com/confluence/display/JFROG/Pipelines+Step-By-Step#PipelinesStep-By-Step-add-pipeline-sourceAddaPipelineSource)  

   Once the new pipeline source is successfully resolved, your pipelines should be listed under "Pipelines > My Pipelines" in Artifactory.  



6. Trigger the first pipeline by uploading an artifact with the expected file name pattern to the repository that will trigger the webhook defined in step (3)    

   The pipeline will create and sign a release bundle for the uploaded artifact and then will trigger the second pipeline to distribute the release bundle to the targeted edge nodes. 

---   




