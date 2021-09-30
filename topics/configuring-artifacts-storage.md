[//]: # (title: Configuring Artifacts Storage)
[//]: # (auxiliary-id: Configuring Artifacts Storage)

The __Project Settings | Artifacts Storage__ tab displays artifact storages configured in this project as well as the storages inherited from parents. 

By default, the built-in TeamCity artifacts storage is displayed and marked as active. You can activate a different storage using the corresponding link.

### Built-in Artifacts Storage

TeamCity stores [artifacts](build-artifact.md) produced by builds on the file system accessible by the TeamCity server. The default artifacts directory location is `<[TeamCity Data Directory](teamcity-data-directory.md)>/system/artifacts` but it can be [redefined](teamcity-configuration-and-maintenance.md).

### External Artifacts Storage 

TeamCity provides a pluggable API to enable external storage for TeamCity build artifacts. Support for different storages can be implemented as an external plugin to TeamCity: the details are provided in the [external storage implementation guide](https://plugins.jetbrains.com/docs/teamcity/external-storage-implementation-guide.html).

Note that when an external storage for artifacts is enabled, the TeamCity [internal](build-artifact.md#Hidden+Artifacts) artifacts (including build logs) will still be published to the TeamCity server and stored in the TeamCity Data Directory in the built-in artifacts' storage.

The same applies to the metadata about artifacts mappings, which will be published to the [artifacts directory](teamcity-configuration-and-maintenance.md) of the TeamCity Data Directory. When restoring from a backup, make sure they are restored for the external artifact plugin to work properly.

### Amazon S3 Support

TeamCity comes bundled with [Amazon S3 Artifact Storage](https://plugins.jetbrains.com/plugin/9623-aws-s3-artifact-storage) plugin which allows storing build artifacts in an Amazon S3 bucket.

It is possible to replace the TeamCity built-in artifacts' storage with [AWS S3](https://aws.amazon.com/s3/) at the project level. When S3 artifact storage is configured, it:
* allows uploading to, downloading and removing artifacts from S3
* handles resolution of artifact dependencies as well as clean-up of artifacts
* displays artifacts located externally in the TeamCity web UI

To enable external artifact storage in an AWS S3 bucket:
1. Navigate to the __Project Settings | Artifacts Storage__ tab. The built-in TeamCity artifacts storage is displayed by default and marked as active.
2. Click __Add new storage__. S3 Storage is selected as the storage type (provided there are no other external storage plugins installed).
3. Provide an optional name for your storage.
4. Select the AWS environment and provide the required settings.
5. Provide your AWS Security Credentials.
6. Specify an existing S3 bucket to store artifacts.
7. Save your settings. 
8. The configured S3 storage will appear on the __Artifacts Storage__ page. Make it active using the corresponding link.

Now new artifacts produced by builds of this project with its subprojects and build configurations will be stored in the specified AWS S3 bucket.

#### Path prefix

<anchor name="pathPrefix"/>

Since TeamCity 2020.2, you can set an S3 [path prefix](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/using-folders.html). This allows using the same S3 bucket for all TeamCity projects and configure prefix-based permissions.

#### Virtual host addressing

<anchor name="forceVirtualHostAddressing"/>

You can enable the virtual host addressing for S3 buckets. Currently, both hosted-style and path-style requests are supported by TeamCity. Note that Amazon [stopped supporting path-style access](https://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html#path-style-access) for new buckets since September 2020.

#### Permissions

When the "Use Pre-Signed URLs for upload" option is enabled, the provided AWS credentials or IAM role on the TeamCity server should have permissions: `DeleteObject, ListAllMyBuckets, GetBucketLocation, GetObject, ListBucket, PutObject`.

When the "Use Pre-Signed URLs for upload" option is disabled:
* the provided AWS credentials or IAM role on the TeamCity server should have permissions: `DeleteObject, ListAllMyBuckets, GetBucketLocation, GetObject`
* either AWS credentials should be specified and have `ListBucket, PutObject` permissions, or IAM role on all the TeamCity agents should have permissions: `ListBucket, PutObject`

### Other External Artifact Storage Plugins

#### Azure Artifact Storage

[Azure Artifact Storage](https://plugins.jetbrains.com/plugin/9617-azure-artifact-storage) is an experimental plugin by JetBrains which allows replacing the TeamCity built-in artifacts' storage by Azure Blob storage. 

#### Google Cloud Artifact Storage

 Google Cloud Artifact Storage is implemented as a [plugin](https://plugins.jetbrains.com/plugin/9634-google-artifact-storage) by JetBrains.

#### Multipart Upload
<anchor name="multipartUpload"/>

To optimize the [upload of large files](https://aws.amazon.com/premiumsupport/knowledge-center/s3-upload-large-files/) to S3, you can initiate [multipart upload](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html) instead of regular upload. To do this, set the multipart upload threshold in the _Connection Settings_ block. The minimum allowed value is `5MB`. Supported suffixes: `KB`, `MB`, `GB`, `TB`. If you leave this field empty, multipart upload will be initiated automatically for all files larger than 8 MB (`8MB` is the default value).

Additionally, you can configure the maximum allowed size of each uploaded file part. The minimum value is `5MB`. If left empty, TeamCity will use `8MB` as the default value.

>We recommend that you configure a [bucket lifecycle policy](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpu-abort-incomplete-mpu-lifecycle-config.html) to prevent incomplete multipart uploads.