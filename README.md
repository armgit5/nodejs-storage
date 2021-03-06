[//]: # "This README.md file is auto-generated, all changes to this file will be lost."
[//]: # "To regenerate it, use `npm run generate-scaffolding`."
<img src="https://avatars2.githubusercontent.com/u/2810941?v=3&s=96" alt="Google Cloud Platform logo" title="Google Cloud Platform" align="right" height="96" width="96"/>

# [Google Cloud Storage: Node.js Client](https://github.com/googleapis/nodejs-storage)

[![release level](https://img.shields.io/badge/release%20level-general%20availability%20%28GA%29-brightgreen.svg?style&#x3D;flat)](https://cloud.google.com/terms/launch-stages)
[![CircleCI](https://img.shields.io/circleci/project/github/googleapis/nodejs-storage.svg?style=flat)](https://circleci.com/gh/googleapis/nodejs-storage)
[![AppVeyor](https://ci.appveyor.com/api/projects/status/github/googleapis/nodejs-storage?branch=master&svg=true)](https://ci.appveyor.com/project/googleapis/nodejs-storage)
[![codecov](https://img.shields.io/codecov/c/github/googleapis/nodejs-storage/master.svg?style=flat)](https://codecov.io/gh/googleapis/nodejs-storage)

> Node.js idiomatic client for [Cloud Storage][product-docs].

[Cloud Storage](https://cloud.google.com/storage/docs) allows world-wide storage and retrieval of any amount of data at any time. You can use Google Cloud Storage for a range of scenarios including serving website content, storing data for archival and disaster recovery, or distributing large data objects to users via direct download.


* [Cloud Storage Node.js Client API Reference][client-docs]
* [github.com/googleapis/nodejs-storage](https://github.com/googleapis/nodejs-storage)
* [Cloud Storage Documentation][product-docs]

Read more about the client libraries for Cloud APIs, including the older
Google APIs Client Libraries, in [Client Libraries Explained][explained].

[explained]: https://cloud.google.com/apis/docs/client-libraries-explained

**Table of contents:**

* [Quickstart](#quickstart)
  * [Before you begin](#before-you-begin)
  * [Installing the client library](#installing-the-client-library)
  * [Using the client library](#using-the-client-library)
* [Samples](#samples)
* [Versioning](#versioning)
* [Contributing](#contributing)
* [License](#license)

## Quickstart

### Before you begin

1.  Select or create a Cloud Platform project.

    [Go to the projects page][projects]

1.  Enable billing for your project.

    [Enable billing][billing]

1.  Enable the Google Cloud Storage API.

    [Enable the API][enable_api]

1.  [Set up authentication with a service account][auth] so you can access the
    API from your local workstation.

[projects]: https://console.cloud.google.com/project
[billing]: https://support.google.com/cloud/answer/6293499#enable-billing
[enable_api]: https://console.cloud.google.com/flows/enableapi?apiid=storage-api.googleapis.com
[auth]: https://cloud.google.com/docs/authentication/getting-started

### Installing the client library

    npm install --save @google-cloud/storage

### Using the client library

```javascript
// Imports the Google Cloud client library
const {Storage} = require('@google-cloud/storage');

// Your Google Cloud Platform project ID
const projectId = 'YOUR_PROJECT_ID';

// Creates a client
const storage = new Storage({
  projectId: projectId,
  keyFilename: 'keyFilename.json'
});

// The name for the new bucket
const bucketName = 'my-new-bucket';

// Creates the new bucket
storage
  .createBucket(bucketName)
  .then(() => {
    console.log(`Bucket ${bucketName} created.`);
  })
  .catch(err => {
    console.error('ERROR:', err);
  });
```

*Note: to get the json key file (https://medium.com/google-cloud/upload-images-to-google-cloud-storage-with-react-native-and-expressjs-61b8874abc49)
In your Google Cloud console (console.cloud.google.com), go to the API Manager.

1. Ensure the Google Cloud Storage JSON API is enabled.
2. Go to Credentials > Create Credentials > Service Account Key
3. Under service account, select create a new service account
4. Once that is created, then you can generate the JSON keyfile.
5. Save that keyfile to your Express project directory.

### Example how to upload a file with multer
1. npm install --save multer
2. Add gcsFileUpload.js in helpers folder as the middleware.
3. Add keyFilename.json in helpers foler as well.
```
const { Storage } = require('@google-cloud/storage');

const gcs = new Storage({
  projectId: projectId,
  keyFilename: './helpers/keyFilename.json'
});

const bucketName = 'my-new-bucket';
const bucket = gcs.bucket(bucketName);

function getPublicUrl(filename) {
  return 'https://storage.googleapis.com/' + bucketName + '/' + filename;
}

let ImgUpload = {};

// Upload a file
ImgUpload.uploadToGcs = (req, res, next) => {
  if (!req.file) return next();

  // Can optionally add a path to the gcsname below by concatenating it before the filename
  // https://cloud.google.com/nodejs/getting-started/using-cloud-storage
  const gcsname = new Date().toISOString() + '-' + req.file.originalname;
  const file = bucket.file(gcsname);

  const stream = file.createWriteStream({
    metadata: {
      contentType: req.file.mimetype
    }
  });

  stream.on('error', (err) => {
    req.file.cloudStorageError = err;
    next(err);
  });

  stream.on('finish', () => {
    req.file.cloudStorageObject = gcsname;
    req.file.cloudStoragePublicUrl = getPublicUrl(gcsname);
    next();
  });

  stream.end(req.file.buffer);
}

// Upload multiple files
ImgUpload.uploadManyToGcs = (req, res, next) => {
  if (!req.files) return next();

  // Can optionally add a path to the gcsname below by concatenating it before the filename
  // https://cloud.google.com/nodejs/getting-started/using-cloud-storage
  req.uploads = []; // Used to store uploads
  let count = 1; // Keeps track of numbers of files

  req.files.forEach(upload => {
    const gcsname = Date.now() + '-' + upload.originalname;
    const file = bucket.file(gcsname);

    const stream = file.createWriteStream({
      metadata: {
        contentType: upload.mimetype
      }
    });

    stream.on('error', (err) => {
      req.files.cloudStorageError = err;
      next(err);
    });

    stream.on('finish', () => {
      req.uploads.push({
        cloudStorageObject: gcsname,
        cloudStoragePublicUrl: getPublicUrl(gcsname)
      });

      // Wait until the last file uploaded
      if (count === req.files.length) next();
      count++;
    });

    // Send files to google cloud
    stream.end(upload.buffer);
    
  });

}

module.exports = ImgUpload;
```

4. In app.js file, add - 
* For uploading a file:
```
// Files upload
const Multer = require('multer');
const imgUpload = require('./helpers/gcsFileUpload');
const multer = Multer({
    storage: Multer.MemoryStorage,
    fileSize: 5 * 1024 * 1024
  });
app.use(multer.single("image"));
app.use(imgUpload.uploadManyToGcs);
```
* For uploading multiple files with maximum of 12 files:
```
// Files upload
const Multer = require('multer');
const imgUpload = require('./helpers/gcsFileUpload');
const multer = Multer({
    storage: Multer.MemoryStorage,
    fileSize: 5 * 1024 * 1024
  });
app.use(multer.array("images[]", 12));
app.use(imgUpload.uploadManyToGcs);
```

5. After creating a post request, in a post function, this will be returned from the middleware:
* For a single file, req.file will be returned:
```
const objectName = req.file.cloudStorageObject;
const fileUrl = req.file.cloudStoragePublicUrl;
```
* For multiple files, req.uploads array will be returned:
```
const uploads = req.uploads;
```

6. Set all users permission to view files (Please see https://stackoverflow.com/questions/40232188/allow-public-read-access-on-a-gcs-bucket).

## Samples

Samples are in the [`samples/`](https://github.com/googleapis/nodejs-storage/tree/master/samples) directory. The samples' `README.md`
has instructions for running the samples.

| Sample                      | Source Code                       | Try it |
| --------------------------- | --------------------------------- | ------ |
| ACL (Access Control Lists) | [source code](https://github.com/googleapis/nodejs-storage/blob/master/samples/acl.js) | [![Open in Cloud Shell][shell_img]](https://console.cloud.google.com/cloudshell/open?git_repo=https://github.com/googleapis/nodejs-storage&page=editor&open_in_editor=samples/acl.js,samples/README.md) |
| Buckets | [source code](https://github.com/googleapis/nodejs-storage/blob/master/samples/buckets.js) | [![Open in Cloud Shell][shell_img]](https://console.cloud.google.com/cloudshell/open?git_repo=https://github.com/googleapis/nodejs-storage&page=editor&open_in_editor=samples/buckets.js,samples/README.md) |
| Encryption | [source code](https://github.com/googleapis/nodejs-storage/blob/master/samples/encryption.js) | [![Open in Cloud Shell][shell_img]](https://console.cloud.google.com/cloudshell/open?git_repo=https://github.com/googleapis/nodejs-storage&page=editor&open_in_editor=samples/encryption.js,samples/README.md) |
| Files | [source code](https://github.com/googleapis/nodejs-storage/blob/master/samples/files.js) | [![Open in Cloud Shell][shell_img]](https://console.cloud.google.com/cloudshell/open?git_repo=https://github.com/googleapis/nodejs-storage&page=editor&open_in_editor=samples/files.js,samples/README.md) |
| Notifications | [source code](https://github.com/googleapis/nodejs-storage/blob/master/samples/notifications.js) | [![Open in Cloud Shell][shell_img]](https://console.cloud.google.com/cloudshell/open?git_repo=https://github.com/googleapis/nodejs-storage&page=editor&open_in_editor=samples/notifications.js,samples/README.md) |
| Requester Pays | [source code](https://github.com/googleapis/nodejs-storage/blob/master/samples/requesterPays.js) | [![Open in Cloud Shell][shell_img]](https://console.cloud.google.com/cloudshell/open?git_repo=https://github.com/googleapis/nodejs-storage&page=editor&open_in_editor=samples/requesterPays.js,samples/README.md) |

The [Cloud Storage Node.js Client API Reference][client-docs] documentation
also contains samples.

## Versioning

This library follows [Semantic Versioning](http://semver.org/).

This library is considered to be **General Availability (GA)**. This means it
is stable; the code surface will not change in backwards-incompatible ways
unless absolutely necessary (e.g. because of critical security issues) or with
an extensive deprecation period. Issues and requests against **GA** libraries
are addressed with the highest priority.

More Information: [Google Cloud Platform Launch Stages][launch_stages]

[launch_stages]: https://cloud.google.com/terms/launch-stages

## Contributing

Contributions welcome! See the [Contributing Guide](https://github.com/googleapis/nodejs-storage/blob/master/.github/CONTRIBUTING.md).

## License

Apache Version 2.0

See [LICENSE](https://github.com/googleapis/nodejs-storage/blob/master/LICENSE)

[client-docs]: https://cloud.google.com/nodejs/docs/reference/storage/latest/
[product-docs]: https://cloud.google.com/storage/docs
[shell_img]: https://gstatic.com/cloudssh/images/open-btn.png
