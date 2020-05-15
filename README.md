# CF S3 Demo

>> This repo is cloned from cloudfoundry samples. gradle-wrapper.properties has been changed from the original repo replacing http with https in the distributionUrl else I was getting a 403 error

This is a simple example of using Amazon S3 (or a different S3-compatible service) for asset storage. It is an image catalog to which you can upload images and see them on the main page.

## IMPORTANT INSTRUCTIONS., DONT MISS

* The app uses a Gradle distribution version that is not compatible with Java 11. This means you will need Java 8 installed on your machine to build the app. I tried it with Java 11 and upgraded the Gradle version to 5.1.1 resulting into weird errors, which I didn't feel like troubleshooting, so stuck onto Java 8 and the included Gradle wrapper distribution.

* You will not need Gradle installed system wide, as Gradle wrapper is included with appropriate settings. 

* You will need a pre-configured S3 bucket with List only public access. Go the S3 bucket's permission on the console > Access control lists > Everyone > List objects (don't need write permissions for public)

## Running Locally

* Create a file called `application.yml` in `src/main/resources`. It should have the following structure (replace the values with those appropriate for your environment):

```yaml
s3:
  aws_access_key: your-aws-access-key
  aws_secret_key: your-aws-secret-key
  region: the-region-where-your-buckets-are
  bucket: the-bucket-name-you-want-to-use
  endpoint: s3-compatible-endpoint (optional)
  base-url: public-base-url-for-uploaded-objects (optional)
  path-style-access: true-or-false (optional, default: false)
mysql:
  driver: com.mysql.jdbc.Driver
  url: jdbc:mysql://localhost:3306/mysql_db
  username: mysql_db
  password: mysql_pw</code></pre>
```

* Assemble the App

```
$ ./gradlew assemble
```

* Run it.

```
$ java -jar build/libs/cf-s3-demo.jar
```

* Browse to `http://localhost:8080`

## DEPLOYING ON CLOUD FOUNDRY

The app uses the concept of services that are managed by Cloud Foundry platform. It uses a brokered MySQL service that is created on the platform itself as a Virtual Machine and a user provided service that is used to inject S3 variables into the application via VCAP variables. Note that on PWS, you can avail a free plan of ClearDB and while on self-hosted platforms you will need to install the MySQL tile. The app works with Spring Cloud Profiles and the Cloud profile automatically injects these variables while binding the app to these two services

## CREATING MYSQL SERVICE

* Running on Cloud Foundry (PIVOTAL WEB SERVICES)

Assuming you already have an account at http://run.pivotal.io:

* Create a ClearDB service.

```
$ cf create-service cleardb spark mysql-service
```

* SELF HOSTED TANZU APPLICATION SERVICES ON ANY IAAS INCLUDING VSPHERE, AWS, GCP, etc..

Download the MySQL service tile from Pivotal Network. Install the tile and configure it to create a Service broker (Note: there are pre-requisites for ASGs). Create the service either via CLI or via Apps Manager in the respective organization space (below command assumes that you have a plan created "db-small")

```
cf create-service p.mysql db-small mysql-service
```

### CREATE A USER PROVIDED SERVICE FOR S3 (SAME FOR PWS AND SELF-HOSTED)

* Create a user-provided service, making sure its name begins with "s3". It should have the following credentials (assign values appropriate for your environment):
    * `accessKey`
    * `secretKey`
    * `region`
    * `bucket`
    * `endpoint` (optional)
    * `baseUrl` (optional)
    * `pathStyleAccess` (optional, default: false)
```
$ cf create-user-provided-service s3-service -p '{"accessKey":"1234","secretKey":"5678","region":"us-west-1","bucket":"cf-s3-bucket"}'
```

* Compile the app.
```
$ ./gradlew assemble
```

* Push it to Pivotal Cloud Foundry. It will fail because services are not bound yet.

```
$ cf push cf-s3-123 -p build/libs/cf-s3-demo.jar
```

* Bind services to the app.

```
$ cf bind-service cf-s3-123 mysql-service
$ cf bind-service cf-s3-123 s3-service
```

* Restage the app.

```
$ cf restage cf-s3-123
```

* Browse to the given URL (e.g. `http://cf-s3-123.cfapps.io` or to the URL provided for your platform).
