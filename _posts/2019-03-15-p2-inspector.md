---
title:  "p2 Inspector"
description: "A headless Eclipse plugin which exposes a REST interface for inspecting and retrieving the contents of a remote P2 repository"
author: avojak
image: https://images.unsplash.com/photo-1525119257764-35ca8b725677
tags:
  - software
  - evergreen
  - app
---

The p2 Inspector was born of necessity. While working on another project, I found that I needed to retrieve various information about [p2](https://www.eclipse.org/equinox/p2/) repositories (e.g. the name, description, license information, etc.). After all, the layout of p2 repositories is pretty simple, and the metadata is just XML; how hard could that be?

```xml
<?xml version='1.0' encoding='UTF-8'?>
<?metadataRepository version='1.1.0'?>
<repository name='Eclipse Project Repository for Oxygen.3' 
            type='org.eclipse.equinox.internal.p2.metadata.repository.LocalMetadataRepository' 
            version='1.0.0'>
  <properties size='2'>
    <property name='p2.timestamp' value='1522460979529'/>
    <property name='p2.compressed' value='true'/>
  </properties>
  <units size='1159'>
    <unit id='org.eclipse.ui.ide.application' version='1.2.0.v20170512-1452'>
      <update id='org.eclipse.ui.ide.application' range='[0.0.0,1.2.0.v20170512-1452)' severity='0'/>
      <properties size='8'>
        <property name='df_LT.Plugin.providerName' value='Eclipse.org'/>
        <property name='df_LT.Plugin.name' value='Eclipse IDE UI Application'/>
        <property name='org.eclipse.equinox.p2.name' value='%Plugin.name'/>
        <property name='org.eclipse.equinox.p2.provider' value='%Plugin.providerName'/>
        <property name='org.eclipse.equinox.p2.bundle.localization' value='plugin'/>
        <property name='maven-groupId' value='org.eclipse.ui'/>
        <property name='maven-artifactId' value='org.eclipse.ui.ide.application'/>
        <property name='maven-version' value='1.2.0-SNAPSHOT'/>
      </properties>
...
```

As it turns out, very hard. Mostly because [there isn't a spec](https://stackoverflow.com/a/12647896/3300205) that defines the layout. Thus began the journey...

## Process

In my hubris I thought that I could figure out some key fields to pull from the XML, and sort of "reverse engineer" the layout of the XML. I quickly came back down to reality and realized that the simplest approach would be to use classes provided by Eclipse: [`IMetadataRepositoryManager`](http://help.eclipse.org/neon/index.jsp?topic=%2Forg.eclipse.platform.doc.isv%2Freference%2Fapi%2Forg%2Feclipse%2Fequinox%2Fp2%2Frepository%2Fmetadata%2FIMetadataRepositoryManager.html) and [`IArtifactRepositoryManager`](http://help.eclipse.org/neon/index.jsp?topic=%2Forg.eclipse.platform.doc.isv%2Freference%2Fapi%2Forg%2Feclipse%2Fequinox%2Fp2%2Frepository%2Fartifact%2FIArtifactRepositoryManager.html).

Armed with those two classes, I yet again thought this would project would be a simple task. And again, I was wrong. The implementations of these classes require a running [OSGi](https://www.osgi.org/) container, and setting this up manually isn't trivial.

### Headless Eclipse

After some research, I came across a project on GitHub [^1] which is a headless Eclipse plugin that uses the two repository classes above to compare the contents of p2 repositories. Bingo! By implementing a headless Eclipse plugin, I no longer had to worry about manually handling an OSGi container.

In order to get instances of the `IMetadataRepositoryManager` and the `IArtifactRepositoryManager` you need an instance of `IProvisioningAgent`. Getting the provisioning agent is slightly tricky, but with the `BundleContext` provided via the plugins `BundleActivator`, it can be accomlished rather simply [^2]:

```java
public class ProvisioningAgentProvider {

    private final BundleContext bundleContext;
    
    public ProvisioningAgentProvider(final BundleContext bundleContext) {
        this.bundleContext = checkNotNull(bundleContext, "bundleContext cannot be null");
    }
    
    public IProvisioningAgent getAgent(final URI location) throws ProvisionException {
        IProvisioningAgent result = null;
        ServiceReference<?> providerRef = bundleContext.getServiceReference(IProvisioningAgentProvider.SERVICE_NAME);
        if (providerRef == null) {
            throw new RuntimeException("No IProvisioningAgentProvider service reference is available"); //$NON-NLS-1$
        }
        IProvisioningAgentProvider provider = (IProvisioningAgentProvider) bundleContext.getService(providerRef);
        if (provider == null) {
            throw new RuntimeException("No IProvisioningAgentProvider service is available"); //$NON-NLS-1$
        }
        result = provider.createAgent(location);
        bundleContext.ungetService(providerRef);
        return result;
    }
    
}
```

Then, to get the `IMetadataRepositoryManager`:

```java
final IMetadataRepositoryManager metadataManager;
try {
    metadataManager = (IMetadataRepositoryManager) agentProvider.getAgent(null)
            .getService(IMetadataRepositoryManager.SERVICE_NAME);
} catch (final ProvisionException e) {
    throw new RuntimeException(e);
}
```

The final challenge was figuring out how to access this headless Eclipse plugin from the other project that needs the p2 repository information.

### Jetty Server

I ultimately settled on using an embedded [Jetty](http://www.eclipse.org/jetty/) server to create a simple REST interface. The decision to use Jetty was largely due to the fact that it's available directly from Eclipse via their update sites [^3]. This makes it very easy to consume within an Eclipse plugin project via a [target definition](https://wiki.eclipse.org/PDE/Target_Definitions):

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?><?pde version="3.8"?>
<target includeMode="feature" name="com.avojak.webapp.p2.inspector.target">
    <locations>
        <location includeAllPlatforms="false" 
                  includeConfigurePhase="true" 
                  includeMode="planner" 
                  includeSource="true" 
                  type="InstallableUnit">
            <unit id="org.eclipse.equinox.sdk.feature.group" version="3.13.4.v20180322-2228"/>
            <repository location="http://download.eclipse.org/releases/oxygen/201804111000"/>
            <unit id="org.eclipse.sdk.feature.group" version="4.7.3.v20180330-0919"/>
        </location>
    </locations>
</target>
```

The server is started when the Eclipse plugin starts up:

```java
public class Application implements IApplication {
    ...
    @Override
	public Object start(final IApplicationContext applicationContext) throws Exception {
		final Server server = serverFactory.create();
		server.start();
		server.join();
		return IApplication.EXIT_OK;
	}
    ...
}
```

### Docker Deployment

With the plugin functional, the remaining challenge was figuring out the best way to deploy it. The result of the [Tycho](https://www.eclipse.org/tycho/) build is a handful of platform-specific executables, so you could easily just run whichever one is applicable to your runtime environment. However, I thought this would be a good opportunity to get some hands-on experience with Docker.

Fortunately all we really need for the Docker container is one of the executables, so the `Dockerfile` is rather simple:

```
FROM openjdk:8-jre

# Set the working directory
RUN mkdir -p /opt/app/
WORKDIR /opt/app/

# Install unzip to extract the application archive
RUN apt-get update && apt-get upgrade -y
RUN apt-get install unzip -y

# Define variables
ARG BUILD_DIR=com.avojak.webapp.p2.inspector.packaging/target
ARG PRODUCT_ID=com.avojak.webapp.p2.inspector.product
ARG TARGET_PLATFORM=linux.gtk.x86_64

# Copy and unzip the Linux executable
COPY ${BUILD_DIR}/products/${PRODUCT_ID}-${TARGET_PLATFORM}.zip .
RUN unzip ${PRODUCT_ID}-${TARGET_PLATFORM}.zip && rm ${PRODUCT_ID}-${TARGET_PLATFORM}.zip

# Set the command to run the executable
CMD ["./p2-inspector"]
```

### Hosting on Heroku

I'm a huge fan of Heroku due to its simplicity and their pipelines, so I opted to host this application there. Unfortunately, getting the Docker build to publish to Heroku was mildly complicated. I followed a handful of guides from Travis CI and Heroku, but still couldn't get everything working. Fortunately I came across an AMAZING blog post [^4] by [@javierfernandes](https://medium.com/@javierfernandes) that solved all of my problems. The only catch was that it wasn't in English. At least the code samples and screenshots didn't need translating!


<figure class="constrained" markdown="1">

![Screenshot-from-2019-03-15-16.07.57](https://s3.amazonaws.com/blog.avojak.ghost/2019/03/Screenshot-from-2019-03-15-16.07.57.png)

<figcaption>Looks like I wasn't the only person who found the post helpful!</figcaption>
</figure>

## Lessons Learned

### Executable type

One of the strangest and challenging (to me) issues that I ran into was with the base image for the Docker container. Initially I was using Alpine Linux, however the executable created from the Tycho build (an ELF 64-bit LSB executable) was not compatible with Alpine [^5]. This manifested as a cryptic "executable not found" error.

### Target Platform

This was the first time that I had created an Eclipse target platform configuration for a project. It required a lot of tweaking to locate exactly which artifacts to pull from the various sources, but in the end it allowed much easier access to additional dependencies which are not part of the base Eclipse SDK. This included Mockito which is invaluable for testing.

## Finished Product

- GitHub: [p2-inspector](https://github.com/avojak/p2-inspector)
- Docker Hub: [avojak/p2-inspector](https://hub.docker.com/r/avojak/p2-inspector)

---

[^1]: [irbull/p2diff](https://github.com/irbull/p2diff)
[^2]: [https://eclipsesource.com/blogs/tutorials/eclipse-p2-tutorial-managing-metadata/](https://eclipsesource.com/blogs/tutorials/eclipse-p2-tutorial-managing-metadata/)
[^3]: [Eclipse Oxygen plugins](http://download.eclipse.org/releases/oxygen/201804111000/plugins/?d)
[^4]: [Continuous Deployment con Docker + Travis + Heroku](https://medium.com/@javierfernandes/continuous-deployment-con-docker-travis-heroku-c24042fb830b)
[^5]: [Executable in Docker image not found](https://stackoverflow.com/questions/54281646/executable-in-docker-image-not-found)