# React Native AWS3

React Native AWS3 is a module for uploading files to S3. Unlike other libraries out there, there are no native dependencies.

```
npm install --save react-native-aws3
```

## Note on S3 user permissions

The user associated with the `accessKey` and `secretKey` you use must have the appropriate permissions assigned to them. My user's IAM policy looks like:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1458840156000",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:GetObjectAcl",
                "s3:GetObjectVersion",
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:PutObjectVersionAcl"
            ],
            "Resource": [
                "arn:aws:s3:::my-bucket/uploads/*"
            ]
        }
    ]
}
```

## Example

```javascript
import { RNS3 } from 'react-native-aws3';

let file = {
  // `uri` can also be a file system path (i.e. file://)
  uri: "assets-library://asset/asset.PNG?id=655DBE66-8008-459C-9358-914E1FB532DD&ext=PNG",
  name: "image.png",
  type: "image/png"
}

let options = {
  keyPrefix: "uploads/",
  bucket: "your-bucket",
  region: "us-east-1",
  accessKey: "your-access-key",
  secretKey: "your-secret-key",
  successActionStatus: 201,
  metadata: {
    latitude: '123.506239',  // Becomes x-amz-meta-latitude onec in S3
    longitude: '-23.045293',
    photographer: 'John Doe'
  }
}

RNS3.put(file, options).then(response => {
  if (response.status !== 201)
    throw new Error("Failed to upload image to S3");
  console.log(response.body);
  /**
   * {
   *   postResponse: {
   *     bucket: "your-bucket",
   *     etag : "9f620878e06d28774406017480a59fd4",
   *     key: "uploads/image.png",
   *     location: "https://your-bucket.s3.amazonaws.com/uploads%2Fimage.png"
   *   }
   * }
   */
});
```

## Usage

### put(file, options)

Upload a file to S3.

Arguments:

1. `file`
  * `uri` **required** - File system URI, can be assets library path or `file://` path
  * `name` **required** - The name of the file, will be stored as such in S3
  * `type` **required** - The mime type, also used for `Content-Type` parameter in the S3 post policy
2. `options`
  * `acl` - The [Access Control List](http://docs.aws.amazon.com/AmazonS3/latest/dev/acl-overview.html) of this object. Defaults to `public-read`.
  * `keyPrefix` - Prefix, or path to the file on S3, i.e. `uploads/` (note the trailing slash).
  * `bucket` **required** - Your S3 bucket
  * `region` **required** - The region of your S3 bucket
  * `accessKey` **required** - Your S3 `AWSAccessKeyId`
  * `secretKey` **required** - Your S3 `AWSSecretKey`
  * `successActionStatus` - HTTP response status if successful, defaults to 201.
  * `serverSideEncryption` - Server-side encryption parameters
    * `encryptionType` - `AES256` for SSE-S3, `aws:kms` for SSE-KMS
    * `kmsKeyId` - KMS key ID for SSE-KMS, format: `arn:aws:kms:region:acct-id:key/key-id`
  * `metadata` - Custom metadata to attach to your object

Returns an object that behaves like a promise. It also has a `progress` method on it which accepts a callback and will invoke the callback with the upload progress.

Example of using the response promise with `progress`:

```javascript
RNS3.put(file, options)
  .then(/* ... */)
  .catch(/* ... */)
  .progress((e) => console.log(e.loaded / e.total));
```

## Server-Side Encryption

Amazon S3 encrypts your data at the object level as it writes it to disks in its data centers and decrypts it for you when you access it. You have three options depending on how you choose to manage the encryption keys:
  1. Use Server-Side Encryption with Amazon S3-Managed Keys (SSE-S3)
  2. Use Server-Side Encryption with AWS KMS-Managed Keys (SSE-KMS) 
  3. Use Server-Side Encryption with Customer-Provided Keys (SSE-C)

Currently, only Server-Side Encryption with SSE-S3 and SSE-KMS are supported in this library.

View the [Amazon Simple Storage Service documentation: Protecting Data Using Server-Side Encryption](http://docs.aws.amazon.com/AmazonS3/latest/dev/serv-side-encryption.html) for more documentation.

## TODO

- [ ] Support `DeleteObject` and (authenticated) `GetObject` operations.


## License

[MIT](https://github.com/benjreinhart/react-native-aws3/blob/master/LICENSE.txt)
