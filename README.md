# url2img

Takes screenshot of a given page. This module correctly handles pages which dynamically load content making AJAX requests.
Instead of waiting fixed amount of time before rendering, we give a short time for the page to make additional requests.
This library correctly handles all childProcess spawning and lifecycle management.

**Usage from code:**

```javascript
var urlToImage = require('url2img');
urlToImage('http://google.com', 'google.png').then(function() {
    // now google.png exists and contains screenshot of google.com
}).catch(function(err) {
    console.error(err);
});
```

**Usage from command line:**

```bash
$ url2img http://google.com google.png
```

Sometimes it's useful to see requests, responses and page errors from PhantomJS:

```bash
$ url2img http://google.com google.png --verbose
-> GET http://google.com/
-> GET http://www.google.fi/?gfe_rd=cr&ei=xTYxVouuOeiA8QexyZ2QBw
<- 302 http://google.com/
-> GET http://ssl.gstatic.com/gb/images/b_8d5afc09.png

... quite a lot of requests ...

-> GET http://ssl.gstatic.com/gb/js/sem_32d9c4210965b8e7bfa34fa376864ce8.js
<- 200 http://ssl.gstatic.com/gb/js/sem_32d9c4210965b8e7bfa34fa376864ce8.js
Render screenshot..
Done.
```


For more options, see [CLI](#command-line-interface-cli) chapter.

## Install

    npm install url2img

PhantomJS is installed by using [Medium/phantomjs NPM module](https://github.com/Medium/phantomjs).

## API

```javascript
var urlToImage = require('url2img');
```

#### urlToImage(url, filePath, options)

This will run a PhantomJS script([url-to-image.js](./src/url-to-image.js)) which renders given url to an image.

**Parameters**

* `url` Url of the page which will be rendered as image. For example `http://google.com`.
* `filePath` File path where to save rendered image.
* `options` Options for page rendering.

    **Default values for options**

    ```javascript
    {
        // User agent width
        width: 1200,

        // User agent height
        height: 800,
		
		// The file type of the rendered image. By default, PhantomJS 
		// sets the output format automatically based on the file extension.
		// Supported: PNG, GIF, JPEG, PDF
		fileType: 'jpeg',
		
		// The file quality of the rendered image, represented as a percentage. 
		// This reduces the image size. By default, 100 percent is used.
		fileQuality: 100,
		
		// Sets the width of the final image (cropped from the User agent defined size)
		// By default, no cropping is done.
		cropWidth: false,
		
		// Sets the height of the final image (cropped from the User agent defined size)
		// By default, no cropping is done.
		cropHeight: false,
		
		//Sets the offset of where to begin the image cropping from the left margin 
		// of the page
		cropOffsetLeft: 0,
	
    	//Sets the offset of where to begin the image cropping from the top margin 
		// of the page
		cropOffsetTop: 0,

        // How long in ms do we wait for additional requests
        // after all initial requests have gotten their response
        // Note: this does NOT limit the amount of time individual request
        //       can take in time
        requestTimeout: 300,

        // How long in ms do we wait at maximum. The screenshot is
        // taken after this time even though resources are not loaded
        maxTimeout: 1000 * 10,

        // How long in ms do we wait for phantomjs process to finish.
        // If the process is running after this time, it is killed.
        killTimeout: 1000 * 60 * 2,

        // If true, phantomjs script will output requests and responses to stdout
        verbose: false

        // String of of phantomjs arguments
        // You can separate arguments with spaces
        // See options in http://phantomjs.org/api/command-line.html
        phantomArguments: '--ignore-ssl-errors=true'
    }
    ```

**Returns**

[Bluebird promise object](http://bluebirdjs.com/docs/api-reference.html).

**Detailed example**

```javascript
var urlToImage = require('url2img');

var options = {
    fileQuality:70,
    width: 1600,
    height: 900,
    cropWidth: 1600,
    cropHeight: 900,
    cropOffsetLeft: 0,
    cropOffsetTop: 0,     
    fileType: 'jpeg',
    requestTimeout: 300,
    maxTimeout: 1000 * 10,
    killTimeout: 1000 * 60 * 2
}

urlToImage('http://google.com', 'google.png', options)
.then(function() {
    // do stuff with google.png
})
.catch(function(err) {
    console.error(err);
});
```

## Command line interface (CLI)

The package also ships with a cli called `urltoimage`.

```
Usage: urltoimage <url> <path> [options]

<url>   Url to take screenshot of
<path>  File path where the screenshot is saved


Options:
  --width              Width of the viewport            [string] [default: 1280]
  --height             Height of the viewport            [string] [default: 800]
  --file-type          The file type of the rendered image. By default, 
		               PhantomJS sets the output format automatically based on 
					   the file extension. Supported: PNG, GIF, JPEG, PDF
					                                   [string] [default: false]
  --file-quality       The file quality of the rendered image, represented as a 
                       percentage. This reduces the image size. 
					   By default, 100 percent is used.
					                                     [string] [default: 100]
  --crop-width         Sets the width of the final image (cropped from the User 
                       agent defined size).
					                                   [string] [default: false]
  --crop-height        Sets the height of the final image (cropped from the User 
                       agent defined size).
					                                   [string] [default: false]
  --cropoffset-left    Sets the offset of where to begin the image cropping from 
                       the left margin of the page.
					                                   [string] [default: false]
  --cropoffset-top     Sets the offset of where to begin the image cropping from 
                       the top margin of the page.
					                                   [string] [default: false]
  --request-timeout    How long in ms do we wait for additional requests after
                       all initial requests have gotten their response
                                                         [string] [default: 300]
  --max-timeout        How long in ms do we wait at maximum. The screenshot is
                       taken after this time even though resources are not
                       loaded                          [string] [default: 10000]
  --kill-timeout       How long in ms do we wait for phantomjs process to
                       finish. If the process is running after this time, it is
                       killed.                        [string] [default: 120000]
  --phantom-arguments  Command line arguments to be passed to phantomjs
                       process.You must use the format
                       --phantom-arguments="--version".
                                  [string] [default: "--ignore-ssl-errors=true"]
  --verbose            If set, script will output additional information to
                       stdout.                        [boolean] [default: false]
  -h, --help           Show help                                       [boolean]
  -v, --version        Show version number                             [boolean]

Examples:
  url2img http://google.com google.png
```

## Test

    npm test

## Attribution

This module was inspired by

* [url-to-image](https://github.com/kimmobrunfeldt/url-to-image)

# License

MIT
