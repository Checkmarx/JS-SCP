File Management
===============

The first precaution to take when handling files is to make sure the users are
not allowed to directly supply data to any dynamic functions: `require` is one
of such functions.

File uploads **SHOULD** only be restricted to authenticated users.
After guaranteeing that file uploads are only made by authenticated users,
another important aspect of security is to make sure that only accepted file
types can be uploaded to the server (_whitelisting_).
This check can be made using, for example, the [mmmagic package][1] - _An async
libmagic binding for Node.js for detecting content types by data inspection_.

```javascript
const mmm = require('mmmagic');
const Magic = mmm.Magic;

const magic = new Magic(mmm.MAGIC_MIME_TYPE);
magic.detectFile('node_modules/mmmagic/build/Release/magic.node', (err, result) => {
  if (err) throw err;

  console.log(result);
  // output on Windows with 32-bit node: application/x-dosexec
});
```

After identifying the file type, an additional step is required to validate the
file type against a whitelist of allowed file types:

```javascript
const allowedMimeTypes = [ 'image/png', 'image/jpeg' ];

if (allowedMimeTypes.indexOf(result) === -1) {
    throw new TypeError('file type not allowed');
}
```

Files uploaded by users should not be stored in the web context of the
application. Instead, files should be stored in a content server or in a
database. An important note is for the selected file upload destination not to
have execution privileges.

If the file server that hosts user uploads is \*NIX based, make sure to
implement safety mechanisms like chrooted environment or mounting the target
file directory as a logical drive.

In the case of dynamic redirects, user data should not be passed. If it is
required by your application, additional steps must be taken to keep the
application safe. These checks include accepting only properly validated data
and relative path URLs.

Additionally, when passing data into dynamic redirects, it is important to make
sure that directory and file paths are mapped to indexes of pre-defined lists
of paths and to use said indexes.

Never send the absolute file path to the user, always use relative paths.

Set the server permissions regarding the application files and resources to
`read-only`, and when a file is uploaded, scan the file for viruses and malwares.

[1]: https://github.com/mscdex/mmmagic