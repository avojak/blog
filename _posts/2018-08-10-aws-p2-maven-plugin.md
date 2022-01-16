---
title:  "AWS p2 Maven Plugin"
description: "A Maven plugin for deploying a p2 update site to an AWS S3 bucket"
author: avojak
image: https://images.unsplash.com/photo-1532785907815-0afcbb413aa5
tags:
  - software
  - app
  - aws
  - evergreen
---

This project stemmed directly from another side project which is currently in progress ([Hydrogen](https://avojak.com/projects/hydrogen), an Eclipse plugin). During testing, I realized that it would be far easier to test via the update site rather than manually installing from a .zip file with each change pushed to Git. Unfortunately, there aren't many options (especially free ones) for hosting a [p2](https://www.eclipse.org/equinox/p2/) update site. Thanks to the generous AWS free tier, I tried hosting the update site in an [AWS S3](https://aws.amazon.com/s3/) bucket with static web hosting enabled. It worked perfectly. At that point it only made sense to learn about writing Maven plugins to automate the process!

## Process

There is a fair amount of existing documentation for creating a Maven plugin, however I very quickly ran into issues with dependency versions despite creating the project via the [Maven Plugin Archetype](https://maven.apache.org/archetypes/maven-archetype-plugin/). After a little trial and error with test dependencies (Especially `maven-compat` which I eventually removed altogether), I landed on the following:

```xml
<dependencies>
    <!-- Compile dependencies -->
    <dependency>
        <groupId>com.amazonaws</groupId>
        <artifactId>aws-java-sdk-s3</artifactId>
        <version>1.11.343</version>
    </dependency>
    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>25.1jre</version>
    </dependency>
    <dependency>
        <groupId>org.apache.maven</groupId>
        <artifactId>maven-core</artifactId>
        <version>3.5.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.maven</groupId>
        <artifactId>maven-plugin-api</artifactId>
        <version>3.5.2</version>
    </dependency>
    <dependency>
        <!-- As of Maven 3.1.0, slf4j-simple is packaged with Maven -->
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.25</version>
    </dependency>
    <!-- Provided dependencies -->
    <dependency>
        <groupId>org.apache.maven.plugin-tools</groupId>
        <artifactId>maven-plugin-annotations</artifactId>
        <version>3.5.1</version>
        <scope>provided</scope>
    </dependency>
    <!-- Test dependencies -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>2.13.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>uk.org.lidalia</groupId>
        <artifactId>slf4j-test</artifactId>
        <version>1.2.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

Once the dependencies were sorted out, it didn't take long to create a working version.

There is only a single [Mojo](https://maven.apache.org/developers/mojo-api-specification.html) to perform the deployment, and it is automatically bound to the deploy [lifecycle phase](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html):

```java
@Mojo(name = "deploy", defaultPhase = LifecyclePhase.DEPLOY, requiresOnline = true)
public class AWSP2Mojo extends AbstractMojo {
    // ...
}
```

The rest of the process is simple and doesn't need much explanation: upload the contents of the `${project.build.directory}/repository` directory to an S3 bucket via the AWS Java SDK. The [SDK](https://aws.amazon.com/documentation/sdk-for-java/) is incredibly easy to use.

The bulk of the complexity (explained below) came from adding a landing page which a user would see if they attempted to paste the URL into their web browser instead of Eclipse. The result is similar to what Eclipse does with their update sites [^1].

## Lessons Learned

### Eventual Consistency

After I started work on the plugin, I began studying for the *AWS Certified Developer - Associate* certification. This was very timely, because in the S3 portion of my studying I learned that S3 provides "eventual consistency for overwrite PUTS and DELETES [^2]." My initial approach for building the landing page was to:

0. Upload all update site content
1. Query the bucket contents (given a prefix) for the new update site.
2. Using the contents of the [S3ObjectSummary](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/s3/model/S3ObjectSummary.html) from the query response, we can easily build a URL for each file in the update site.
3. Build and upload the HTML file.

This was a perfect example of how eventual consistency could haunt me. In the event that the update site which was just deployed is overwriting an existing site (e.g. a new SNAPSHOT deployment), we would be performing an overwrite PUT. When we turn around and query the contents in step 1, we may receive old object data because the changes have not fully propagated.

To combat this, I had to model the file/directory hierarchy and build the URLs ahead of time.

### The Trie

To model the file and directory hierarchy [^3], I decided to use a [trie](https://en.wikipedia.org/wiki/Trie). Each node of the trie would represent either a directory, or a file, and because we want the value of the node to represent the URL of an actual object, we don't allow directory nodes to hold a value.

<figure class="constrained" markdown="1">

![trie](https://s3.amazonaws.com/blog.avojak.ghost/2018/07/trie.png)

<figcaption>A simplified - and slightly altered - trie representing an example update site</figcaption>
</figure>

Because of the potential problems with eventual consistency, it was simplest to build the trie while the update site content is being uploaded. The object key for each file is split into a file path and used to traverse the nodes of the trie until a leaf node is reached. At this point either a new `DirectoryTrieNode` or `FileTrieNode` is created.

Now when it's time to build the HTML for the landing page, we simply traverse the trie!

### Object Listings

One nuance that I ran into was different object listing results depending on whether objects were uploaded via the AWS console, or the SDK [^4].

The AWS console provides an easy interface for interacting with objects in S3, as well as creating "directories". I use quotes because S3 doesn't actually have the concept of directories - only objects. Under the hood, S3 is creating zero-sized objects for the "intermediate" object key. So even though we've only actually uploaded one file, we've created multiple objects.

When uploading via the SDK, this is all done in one PUT request, where the full file path is part of the object key. The result is a single object - not one for each "directory".

This ultimately did not affect any development since files are only uploaded via the SDK, but I did find it interesting, and it's definitely something to keep in mind and account for when creating a client to read objects from S3.

## Finished Product
![aws-p2-maven-plugin](https://s3.amazonaws.com/blog.avojak.ghost/2018/07/aws-p2-maven-plugin.gif)

- GitHub: [aws-p2-maven-plugin](https://github.com/avojak/aws-p2-maven-plugin)
- Maven Central: [com.avojak.mojo:aws-p2-maven-plugin](http://mvnrepository.com/artifact/com.avojak.mojo/aws-p2-maven-plugin)

---

[^1]: [http://download.eclipse.org/eclipse/updates/4.7/](http://download.eclipse.org/eclipse/updates/4.7/)
[^2]: [https://docs.aws.amazon.com/AmazonS3/latest/dev/Introduction.html](https://docs.aws.amazon.com/AmazonS3/latest/dev/Introduction.html) (See: "Amazon S3 Data Consistency Model")
[^3]: Technically S3 does not have directories, but using delimiters in the object key you can achieve the same effect
[^4]: [https://stackoverflow.com/questions/48069409/different-s3-object-listing-when-uploading-via-java-sdk-and-aws-console](https://stackoverflow.com/questions/48069409/different-s3-object-listing-when-uploading-via-java-sdk-and-aws-console)