Files
=====

User input data can take the form of uploaded files, and they should be
subject to strict validation.

[Express web framework for Node.js][1] recommends [multer middleware][2] for
handling `multipart/form-data`.
In this section, we will cover the most important security best practices
regarding files uploads, pointing you to the middleware documentation page
which is full of usage examples.

## Store Uploaded Files on a Specific Location

Uploaded files should be stored on a specific destination other than the
location where your application is running.

Files should be stored without execution privileges and directory listing
should be disabled.

Regarding storage location, [multer middleware][2] supports two types:
[DiskStorage][3] and [MemoryStorage][4]. Remember to specify which you want to
use, as the default one, `MemoryStorage`, is not appropriate for large
files or multiple small ones, as the available memory can get exhausted.

Uploaded files should be renamed only using safe characters. [multer allows
you to provide a function to compute the file][3] name, otherwise uploaded files
are given random names without file extension.

If the uploaded file original metadata (e.g. original file location) is
required, it should be stored on another storage, such as a database.

## Limit What Users Are Allowed to Upload

You should enforce what users are allowed to upload - if you're requesting, for
example, an avatar image, it does not make sense to allow users to upload
executable files. **DO NOT** rely on file extension to determine the file type.

Similar to that, restrictions on file sizes should be applied. Otherwise, big
files may exhaust available resources (memory and storage space), causing a
Denial of Service.

[multer][2] allows you to do both with passing simple configurations; see the
[limits][5] and [fileFilter][6] configurations.

## DO NOT Include or Execute User Uploaded Content

Only in really, very special cases you will need to include or execute user
uploaded content. If this is your case, do it in a sandboxed environment.

There are several projects with different features and security capabilities.
Below you will find an incomplete list; you should see which best fits your
needs:

* [jailed][7] - Jailed is a small JavaScript library for running untrusted code
  in a sandbox. The library is written in vanilla-js and has no dependencies.
* [sandbox][8] - A nifty JavaScript sandbox for Node.js.

Other guidelines and recommendations about file manipulation and management 
are available in the [File Management section][9].

[1]: https://expressjs.com/
[2]: https://expressjs.com/en/resources/middleware/multer.html
[3]: https://expressjs.com/en/resources/middleware/multer.html#diskstorage
[4]: https://expressjs.com/en/resources/middleware/multer.html#memorystorage
[5]: https://expressjs.com/en/resources/middleware/multer.html#limits
[6]: https://expressjs.com/en/resources/middleware/multer.html#filefilter
[7]: https://github.com/asvd/jailed
[8]: https://github.com/gf3/sandbox
[9]: ../../file-management/README.md
