# recursive-readdir-async 

[![Build Status](https://travis-ci.org/GerHobbelt/recursive-readdir-async.svg?branch=master)](https://travis-ci.org/GerHobbelt/recursive-readdir-async)
[![Coverage Status](https://coveralls.io/repos/github/GerHobbelt/recursive-readdir-async/badge.svg?branch=master)](https://coveralls.io/github/GerHobbelt/recursive-readdir-async?branch=master)
[![NPM version](https://badge.fury.io/js/%40gerhobbelt%2Frecursive-readdir-async.svg)](https://badge.fury.io/js/%40gerhobbelt%2Frecursive-readdir-async)
[![Dependency Status](https://img.shields.io/david/GerHobbelt/recursive-readdir-async.svg)](https://david-dm.org/GerHobbelt/recursive-readdir-async)
[![GitHub license](https://img.shields.io/github/license/GerHobbelt/recursive-readdir-async.svg)](https://github.com/GerHobbelt/recursive-readdir-async/blob/master/LICENSE)
[![Average time to resolve an issue](http://isitmaintained.com/badge/resolution/GerHobbelt/recursive-readdir-async.svg)](http://isitmaintained.com/project/GerHobbelt/recursive-readdir-async "Average time to resolve an issue")
[![Percentage of issues still open](http://isitmaintained.com/badge/open/GerHobbelt/recursive-readdir-async.svg)](http://isitmaintained.com/project/GerHobbelt/recursive-readdir-async "Percentage of issues still open")

NPM Module to recursive read directory async (non blocking). Returns Promise. Configurable, with callback for extended filtering and progress status. Quiet, NO dependencies.
As non blocking module it is perfect to be used in any javascript based Desktop applications.
>This module uses Promises and can't be used in old javascript engines.
## Installation
For normal usage into a project, you must install as a NPM dependency. The next command will do all the work:
```
npm install --save recursive-readdir-async
```
After install, you can use the module using the *require* key:
```javascript
// Assign recursive-readdir-async to constant
const rra = require('recursive-readdir-async')
// use it
```
## Usage:
Example of basic usage:
```javascript
const rra = require('recursive-readdir-async');
const list = await rra.list('.');
console.log(list)
```
```javascript
const rra = require('recursive-readdir-async');
rra.list('.').then(function(list){
    console.log(list)
})
```
Example with full features:
```javascript
const rra = require('recursive-readdir-async');
const options = {
    mode: rra.LIST,
    recursive: true,
    stats: false,
    ignoreFolders: true,
    extensions: false,
    deep: false,
    realPath: true,
    normalizePath: true,
    include: [],
    exclude: []
}
const list = await rra.list('.', options, function (obj, index, total) {
    console.log(`${index} of ${total} ${obj.path}`)
    if(obj.name=="folder2")
        return true;// return true to delete item
})
if(list.error)
    console.error(list.error)
else
    console.log(list)
```
## Options
An options object can be passed to configure the module. The next options can be used:
* **mode (LIST | TREE)** : The list will return an array of items. The tree will return the items structured like the file system. *Default: list*
* **recursive (true | false)** : If true, files and folders of folders and subfolders will be listed. If false, only the files and folders of the select directory will be listed. *Default: true*
* **stats (true | false)** : If true a `stats` object ([with file information](https://nodejs.org/api/fs.html#fs_class_fs_stats)) will be added to every item. If false this info is not added. *Default: false*
* **ignoreFolders (true | false)** : If true and mode is LIST, the list will be returned with files only. If true and mode is TREE, the directory structures without files will be deleted. If false, all empty and non empty directories will be listed. *Default: true*
* **extensions (true | false)** : If true, lowercase extensions will be added to every item in the `extension` object property (`file.TXT` => `info.extension = ".txt"`). *Default: false*
* **deep (true | false)** : If true, folder depth information will be added to every item starting with 0 (initial path), and will be incremented by 1 in every subfolder. *Default: false*
* **normalizePath (true | false)** : Normalizes windows style paths by replacing double backslahes with single forward slahes (unix style). *Default: true*
* **realPath (true | false)** : Computes the canonical pathname by resolving `.`, `..` and symbolic links. *Default: true*
* **include (Array of String)** : Positive filter the items: only items which [DO](http://www.ietf.org/rfc/rfc2119.txt) (partially or completely) match one of the strings in the `include` array will be returned. *Default: []*
* **exclude (Array of String)** : Negative filter the items: only items which [DO NOT](http://www.ietf.org/rfc/rfc2119.txt) (partially or completely) match *any* of the strings in the `exclude` array will be returned. *Default: []*

### Notes
* Counter-intuitive to some folks, an *empty* `include` array is treated same as setting it to `null` / `undefined`: no include filter will be applied.
  Obviously, an *empty* `exclude` array acts similar: no `exclude` filter will be applied.
* The `include` and `exclude` options interact.
  
  **When `mode` is TREE**
  
  * Directories which [DO NOT](http://www.ietf.org/rfc/rfc2119.txt) match the `include` criteria themselves but contain items which [DO](http://www.ietf.org/rfc/rfc2119.txt) *are kept* in the returned items tree. I.e. *inclusion of the child* has precedence over *rejection of the parent*.
  * Directories which [DO](http://www.ietf.org/rfc/rfc2119.txt) match one of the `exclude` criteria themselves but contain items which [DO NOT](http://www.ietf.org/rfc/rfc2119.txt) *will not* be kept in the returned items tree. I.e. *exclusion of the parent* has precedence over *remaining of the child*.
  
  **When `mode` is LIST**

  As the directory tree is flattened into a list, directories and their children (subdirectories and files) are filtered through the `exclude` and `include` rules independently, hence `include` and `exclude` will only interact when an item matches *both* filters. See below: 

  **Common ground: `mode` is LIST or TREE**

  * `exclude` has precedence over `include`: exclusion rules are applied before the inclusion rules. Hence when an item matches both a string in the `include` array and a string in the `exclude` array, the item will be *excluded* (removed) from the list.
  
## Object structure
The function will return an object and never throw an error. All errors will be added to the returned object. The return object in LIST mode looks like this:
```json
[
    {
        "name":"item_name",
        "path":"/absolute/path/to/item",
        "fullname":"/absolute/path/to/item/item_name",
        "extension":"",
        "isDirectory": true,
        "stats":{

        }
    },
    {
        "name":"file.txt",
        "path":"/absolute/path/to/item/item_name",
        "fullname":"/absolute/path/to/item/item_name/file.txt",
        "extension":".txt",
        "isDirectory": false,
        "stats":{

        }
    },
    {
        "name":"UCASE.JPEG",
        "path":"/absolute/path/to/item/item_name",
        "fullname":"/absolute/path/to/item/item_name/UCASE.JPEG",
        "extension":".jpeg",
        "isDirectory": false,
        "stats":{

        }
    }
]
```
The same example for TREE mode:
```json
[
    {
        "name":"item_name",
        "path":"/absolute/path/to/item",
        "fullname":"/absolute/path/to/item/item_name",
        "isDirectory": true,
        "stats":{

        },
        "contents": [
            {
                "name":"file.txt",
                "path":"/absolute/path/to/item/item_name",
                "fullname":"/absolute/path/to/item/item_name/file.txt",
                "extension":".txt",
                "isDirectory": false,
                "stats":{

                }
            },
            {
                "name":"UCASE.JPEG",
                "path":"/absolute/path/to/item/item_name",
                "fullname":"/absolute/path/to/item/item_name/UCASE.JPEG",
                "extension":".jpeg",
                "isDirectory": false,
                "stats":{

                }
            }
        ]
    }
]
```
>`isDirectory` only exists if `stats`, `recursive` or `ignoreFolders` are `true` or `mode` is TREE

>`stats` only exists if `options.stats` is `true`

>`extension` only exists if `options.extensions` is `true`
## Errors handling
All errors will be added to the returned object. If an error occurs on the main call, the error will be returned like this:
```json
{
    "error":
        {
            "message": "ENOENT: no such file or directory, scandir '/inexistentpath'",
            "errno": -4058,
            "code": "ENOENT",
            "syscall": "scandir",
            "path": "/inexistentpath" 
        },
    "path":"/inexistentpath"
}
```
For errors with files and folders, the error will be added to the item like this:
```json
[
    {
        "name":"item_name",
        "path":"/absolute/path/to/item",
        "fullname":"/absolute/path/to/item/item_name",
        "error":{
            
        }
    }
    {
        "name":"file.txt",
        "path":"/absolute/path/to/item",
        "fullname":"/absolute/path/to/item/file.txt",
        "error":{

        }
    }
]
```
