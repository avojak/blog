---
title:  "AWS p2 Repository"
description: "A web application to host p2 repositories backed by AWS S3"
author: avojak
image: https://images.unsplash.com/photo-1501004318641-b39e6451bec6
tags:
  - software
  - aws
  - app
  - evergreen
---

*I promise that this is the latest project related to AWS and p2 for a while!*

Not too long ago I [posted](https://blog.avojak.com/2018/08/10/aws-p2-maven-plugin/) about a [Maven](https://maven.apache.org/) plugin that I had created to deploy a [p2](https://www.eclipse.org/equinox/p2/) repository to [AWS S3](https://aws.amazon.com/s3/). My goal with that plugin was to host the repository for free on AWS, and make it accessible to the public. The drawback that I hadn't considered was that I couldn't easily provide a static URL from which to retrieve the *latest* version of the software. Even with a custom domain name setup, you would still need to know the exact version that you are looking for:

`https://p2.avojak.com/hydrogen/releases/1.0.0`

Ideally, I would be able to provide a one-stop URL that will always fetch the latest version:

`https://p2.avojak.com/hydrogen/releases/latest`

This got me thinking - it would also be nice to provide a site to view all available software hosted on the bucket. When you visit `https://p2.avojak.com`, you would expect to see all content hosted there.

I ultimately decided to play around with [Spring Boot](https://spring.io/projects/spring-boot) to create a web application that would serve up content hosted in an S3 bucket. The web application would be able to take requests for `/latest` (or a particular version), inpsect the bucket content, and return the content that is the latest version available. Furthermore, it would be simple to add a minimal UI with [Thymeleaf](https://www.thymeleaf.org/) to view the bucket content.

## Process

The first step was to setup the Spring Boot project with a few modules for the different application layers:

* `aws-p2-repository-dataaccess`: Data-access layer to retrieve content from S3
* `aws-p2-repository-model`: To hold models shared by the various application layers
* `aws-p2-repository-service`: To hold any "business logic" between the REST requests and the data-access layer
* `aws-p2-repository-webapp`: To define the REST API and provide the UI templates

### Data REST Controller

To handle requests for content, I created a `DataController` class to setup REST endpoints under `/content`. The primary method retrieves a [`Resource`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/Resource.html) for files:

```java
@GetMapping(value = "/{project}/{qualifier}/{version}/**",
            produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
public ResponseEntity<Resource> getResource(@PathVariable("project") final String project,
                                            @PathVariable("qualifier") final String qualifier,
                                            @PathVariable("version") final String version,
                                            final HttpServletRequest request) {
    ...
}
```

If the `version` URL parameter matches "latest", I simply lookup the most recent version for the qualifier ("snapshots" or "releases"), and proceed with a discrete version.

### Identifying Projects and Versions

Objects in S3 are identified uniquely by keys. If you use `/` (or any other deliminator) in the key, and create objects of size 0, you can effectively create a directory structure with files and directories. If you were to then query for all of the keys in a bucket, you would get a LOT of results:

```
/hydrogen/
/hydrogen/snapshots/
/hydrogen/snapshots/1.0.0-SNAPSHOT/
/hydrogen/snapshots/1.0.0-SNAPSHOT/features/
/hydrogen/snapshots/1.0.0-SNAPSHOT/features/...
/hydrogen/snapshots/1.0.0-SNAPSHOT/plugins/
/hydrogen/snapshots/1.0.0-SNAPSHOT/plugins/...
/hydrogen/snapshots/1.0.0-SNAPSHOT/artifacts.jar
/hydrogen/snapshots/1.0.0-SNAPSHOT/artifacts.xml.xz
/hydrogen/snapshots/1.0.0-SNAPSHOT/content.jar
/hydrogen/snapshots/1.0.0-SNAPSHOT/content.xml.xz
/hydrogen/snapshots/1.0.0-SNAPSHOT/p2.index
```

...and that's just for ONE SNAPSHOT version! You can see how quickly that list of keys can grow. That poses a bit of a challenge when we want to determine a simple list of projects and available versions.

In order to do this, I perform the following steps:

1. Query the bucket for all keys
2. Filter to only keys which contain `p2.index`

And that's it! This is based on the assumption that a valid p2 repository will necessarily contain a `p2.index` file, which is used by Eclipse to locate the repository content (the content and artifacts archives).

With those two, simply steps, the very large list of keys dwindles quickly to a much shorter list:

```
/hydrogen/snapshots/1.0.0-SNAPSHOT/p2.index
/hydrogen/snapshots/1.1.0-SNAPSHOT/p2.index
/hydrogen/releases/1.0.0/p2.index
/foobar/snapshots/1.0.0/p2.index
```

Perfect - one key per version per project! Now by simply dropping the `/p2.index` from the key, we have a key prefix to locate all content for that project version.

## The Web UI

To be perfectly honest, I didn't spend much time working on the UI. I just wanted something simple that had key bits of information:

* The projects availble
* Project versions
* Project description
* Last-updated timestamps
* Licensing information

When you reach the "home" page (the `/browse` endpoint), you will see some basic details, including service up-time, and the S3 bucket which is backing the repository:

![Screenshot-from-2019-03-31-12.25.58](https://s3.amazonaws.com/blog.avojak.ghost/2019/03/Screenshot-from-2019-03-31-12.25.58.png)

To view available projects, click on the "Projects" link in the nav-bar:

![Screenshot-from-2019-03-31-12.26.21](https://s3.amazonaws.com/blog.avojak.ghost/2019/03/Screenshot-from-2019-03-31-12.26.21.png)

When you select a project, you will see the name, description (if available), the list of installable unit groups that are a part of the project, and versioning information:

![Screenshot-from-2019-03-31-12.26.37](https://s3.amazonaws.com/blog.avojak.ghost/2019/03/Screenshot-from-2019-03-31-12.26.37.png)

On the Eclipse side, when you provide the `/content` URL for the latest version:

![Screenshot-from-2019-03-31-12.32.46](https://s3.amazonaws.com/blog.avojak.ghost/2019/03/Screenshot-from-2019-03-31-12.32.46.png)

### Metadata

To retrieve repository metadata I leveraged my previous project ([p2-inspector](https://github.com/avojak/p2-inspector)) to query the p2 repository.

## Lessons Learned

I had used Spring Boot and the AWS SDK before working on this project, however Thymeleaf was completely new to me. This was a bit of a nusance, although once I figured out how to create reusable partials, my UI templates got a bit cleaner. I'm still not completely happy with my templates, as there is a lot of logic performed in the template (e.g. conditionally displaying messages for no content). Ideally, this would all be performed server-side and the UI would be "dumb".

## Future Work

The one key feature that I did not implement, but would be nice to have, is the ability to upload repositories via the web dashboard. There's an issue for it in GitHub, so maybe at some point I'll tackle it.

## Finished Product

- GitHub: [aws-p2-repository](https://github.com/avojak/aws-p2-repository)
- Docker Hub: [avojak/aws-p2-repository](https://hub.docker.com/r/avojak/aws-p2-repository)