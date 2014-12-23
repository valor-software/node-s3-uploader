AWS S3 Image Uploader (EXTENDED)
=====================

Based on:

[![NPM](https://nodei.co/npm/s3-uploader.png?downloads=true)](https://www.npmjs.org/package/s3-uploader)


Flexible and efficient resize, rename, and upload images to Amazon S3 disk
storage. Uses the official [AWS Node SDK](http://aws.amazon.com/sdkfornodejs/)
and [GM](https://github.com/aheckmann/gm) for image processing.

## Package.json

```javascript
  "dependencies": {
    // [...]
    "node-s3-uploader": "git+https://github.com/valor-software/node-s3-uploader",
    // [...]
  }
```

## Install

```
npm install node-s3-uploader --save
```

## Requirements

* Node.JS >= v0.10
* imagemagic
* AWS credentials environment variables
  * `AWS_ACCESS_KEY_ID`
  * `AWS_SECRET_ACCESS_KEY`

```
npm install imagemagic --save
npm install aws-sdk --save
```

```javascript
  var AWS = require('aws-sdk');
  var AWS_ACCESS_KEY_ID = 'AWS_ACCESS_KEY_ID';
  var AWS_SECRET_ACCESS_KEY = 'AWS_SECRET_ACCESS_KEY';
  var AWS_REGION = 'AWS_REGION';

  AWS.config.update({accessKeyId: AWS_ACCESS_KEY_ID, secretAccessKey: AWS_SECRET_ACCESS_KEY, region: AWS_REGION});
```

## API

```javascript
var Upload = require('s3-uploader');
```

### new Upload(`string` awsBucketName, `object` opts)

* `string` awsBucketName - name of Amazon S3 bucket
* `object` opts - global upload options
  * `string` awsBuckeRegion - region for you bucket; default "us-east-1"
  * `string` awsBucketPath - path within your bucket (ex. "/images")
  * `string` awsBucketAcl - default ACL for uploded images
  * `number` awsMaxRetries - max number of retries; default 3
  * `number` awsHttpTimeout - inactive time (ms) beofre timing out; default 10000
  * `number` resizeQuality - default resize quallity
  * `boolean` returnExif - return exif data for original image
  * `string` tmpDir - directory to store temporary files
  * `number` asyncLimit - limit number of async workers
  * `function` getRandomPath - function for generating filepath (for s3)
  * `object[]` versions - versions to upload to S3
    * `boolean` original - if this is the original image
    * `string` suffix - this is appended to the file name
    * `number` quality - resized image quality
    * `string` formatSuffix - suffix for format (e.g. jpg.orig)
    * `string` format - new format fro image (e.g. jpg, png, etc)
    * `number` maxWidth - max width for resized image
    * `number` maxHeight - max height for resized image

#### Example

```javascript
var client = new Upload('my_s3_bucket', {
  awsBucketRegion: 'us-east-1',
  awsBucketPath: 'images/',
  awsBucketAcl: 'public-read',

  versions: [{
    original: true
  },{
    suffix: '-large',
    quality: 80,
    maxHeight: 1040,
    maxWidth: 1040,
  },{
    suffix: '-medium',
    maxHeight: 780,
    maxWidth: 780
  },{
    suffix: '-small',
    maxHeight: 320,
    maxWidth: 320
  }]
});
```

### #upload(`string` src, `object` opts, `function` cb)

* `string` src - absolute path to source image to upload
* `object` opts - local upload config options (overwrites global config)
  * `object` file - original image object (e.g. for getRandomPath function where original filename could be used)
* `function` cb - callback function (`Error` err, `object[]` versions, `object` meta)
  * `Error` err - `null` if everything went fine
  * `object[]` versions - original and resized images with path/location
  * `object` meta - metadata for original image

#### Example

```javascript
client.upload('/some/file/path.jpg', {}, function(err, images, meta) {
  if (err) {
    console.error(err);
  } else {
    for (var i = 0; i < images.length; i++) {
      console.log('Thumbnail with width %i, height %i, at %s', images[i].width, images[i].height, images[i].url);
    }
  }
});
```

#### Algorithm

```
A
+-- B
    `-- C
    `-- D
    `-- E

Where A is the original image uploaded by the user. An mpc image is created, B,
which is used to crate the thumbnails C, D, and E.
```

## Lisence

[MIT](https://github.com/Turistforeningen/node-s3-uploader/blob/master/LICENSE)
