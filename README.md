# Welcome to the DriveTime Jenkins DSL Library

#What is Jenkins DSL?
Here is a great video from January 2014: [Adoption of the Job DSL Plugin at Netflix](https://www.youtube.com/watch?v=FeSKrBvT72c) (20 minutes).  It's fairly old, but will give you a quick look into what Jenkins DSL is.

A good summary is, they came up with a way to put the definition of jenkins jobs into code.  The code gets run as a "seed job" that runs your custom DSL code, which then generates your jenkins builds that you end up using in your normal work.

These seed jobs can run over and over again and will update the final jobs.

#What is the in house library for Jenkins DSL?
Since DSL is really just programming, we can use good design patterns for our build definitions to allow for teams to re-use functionality and easily extend it when needed.  

Instead of a very verbose ```job { }``` block in DSL, you may have a simple ```new DriveTimeNugetBuild()``` and only have to specify a few simple parameters.

#How is this library distributed?
Currently the library exists in the [Builds.Jenkins](https://github.com/DriveTimeInc/Builds.Jenkins) repository and is copied down by [SeedBuildDefinition](https://github.com/DriveTimeInc/Jenkins.Builds/blob/master/Builds/SeedBuildDefinition.groovy) job's.  

This is done somewhat haphazardly by copying the code from the Builds.Jenkins repo on top of the code of subsequent seed builds.  This means that the Builds.Jenkins source code files were copied into the build for Builds.QA.  This allows the Builds.QA to reference the latest version of the library without having to go through a maven package or something more elaborate.  This is a less than ideal situation because you are unable to proprely develop in an IDE due to the way files are copied back and forth between workspaces.  

The end result, however, is that we are able to share code from the root library located in Builds.Jenkins, out to other builds.

#What are the contents of the library?

```
Builds.Jenkins
│
└───Batch Files
│/* re-usable scripts */
│
└───Builds
│/* re-usable build definitions */
│
└───CSS					    
│/* CSS for styling build pages */
│
└───Framework / LifeCycle   
│/* contains extensibility points across all builds */
│
└───Scm
│/* contains common connections we support (Tfs, Tfs git, GitHub, etc)*/
│
│   jenkins_seed.groovy     
 /* list of jobs to be created */   

```

// TO DO: Generate or create documentation for using the build definitions and scm connectors

#How do I get started using it?

##Make your own seed job

###Point to your seed job from the root seed job
You need to make your own seed job from within the Jenkins.Builds seed job.  

To get this, add a new seed job to the root dsl script, [Jenkins.Builds/jenkins_seed.groovy](https://github.com/DriveTimeInc/Builds.Jenkins/blob/master/jenkins_seed.groovy).  Currently this looks like so
```
ICreateableJob[] buildsToCreate = [
	  
	new SeedBuildDefinition(
  		jobName: 'YourTeamsName Seed',   
		scmConnector: new GitHubConnector(repoName: 'Builds.YourTeamsName'),
		groovyFileToRun: 'jobs.groovy',
		descriptionText: 'html read from an html file'
    ),
	
	// other build definitions

];
```
###Create your seed job in your repo
You'll now need to create your groovy file that you specified in the ```groovyFileToRun``` proprety, and put it in the repository that you pointed to in the ```scmConnector```.  Here is a good template

```
import Builds.*;
import Builds.Steps.MsBuildCall;
import Scm.*;

def rootLocation = "QAAutomation";

folder(rootLocation) {
	description "folder description pulled from an html file";
}

ICreateableJob[] buildsToCreate = [
  
	// your builds here

];

buildsToCreate.each { buildToCreate -> buildToCreate.create(delegate, this, out) }
```

###Fill your seed job with builds
You'll want to start adding jobs.  Hopefully you will be re-using an existing job definition.  You can find documentation on those below.  

If you want to write your own definition, feel free to do so and you will be able to use it just like the other build definitions above.  Please refer to them for examples on how to write your own, for now.

