## Introduction
Download website to a local directory (including all css, images, js, etc.)

[![Build Status](https://img.shields.io/travis/s0ph1e/node-website-scraper/master.svg?style=flat)](https://travis-ci.org/s0ph1e/node-website-scraper)
[![Build status](https://ci.appveyor.com/api/projects/status/s7jxui1ngxlbgiav/branch/master?svg=true)](https://ci.appveyor.com/project/s0ph1e/node-website-scraper/branch/master)
[![Test Coverage](https://codeclimate.com/github/s0ph1e/node-website-scraper/badges/coverage.svg)](https://codeclimate.com/github/s0ph1e/node-website-scraper/coverage)
[![Code Climate](https://codeclimate.com/github/s0ph1e/node-website-scraper/badges/gpa.svg)](https://codeclimate.com/github/s0ph1e/node-website-scraper)
[![Dependency Status](https://david-dm.org/s0ph1e/node-website-scraper.svg?style=flat)](https://david-dm.org/s0ph1e/node-website-scraper)

[![Version](https://img.shields.io/npm/v/website-scraper.svg?style=flat)](https://www.npmjs.org/package/website-scraper)
[![Downloads](https://img.shields.io/npm/dm/website-scraper.svg?style=flat)](https://www.npmjs.org/package/website-scraper)
[![Gitter](https://badges.gitter.im/s0ph1e/node-website-scraper.svg)](https://gitter.im/s0ph1e/node-website-scraper?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

[![NPM Stats](https://nodei.co/npm/website-scraper.png?downloadRank=true&stars=true)](https://www.npmjs.org/package/website-scraper)

You can try it in [demo app](https://scraper.nepochataya.pp.ua/) ([source](https://github.com/s0ph1e/web-scraper))

**Note:** dynamic websites (where content is loaded by js) may be saved not correctly because `website-scraper` doesn't execute js, it only parses http responses for html and css files.


## Installation
```
npm install website-scraper
```

## Usage
```javascript
var scrape = require('website-scraper');
var options = {
  urls: ['http://nodejs.org/'],
  directory: '/path/to/save/',
};

// with promise
scrape(options).then((result) => {
	/* some code here */
}).catch((err) => {
	/* some code here */
});

// or with callback
scrape(options, (error, result) => {
	/* some code here */
});
```

## options
* [urls](#urls) - urls to download, *required*
* [directory](#directory) - path to save files, *required*
* [sources](#sources) - selects which resources should be downloaded
* [recursive](#recursive) - follow anchors in html files
* [maxDepth](#maxdepth) - maximum depth for dependencies
* [request](#request) - custom options for for [request](https://github.com/request/request)
* [subdirectories](#subdirectories) - subdirectories for file extensions
* [defaultFilename](#defaultfilename) - filename for index page
* [prettifyUrls](#prettifyurls) - prettify urls
* [ignoreErrors](#ignoreerrors) - whether to ignore errors on resource downloading
* [urlFilter](#urlfilter)
* [filenameGenerator](#filenamegenerator)
* [httpResponseHandler](#httpresponsehandler)
 
Default options you can find in [lib/config/defaults.js](https://github.com/s0ph1e/node-website-scraper/blob/master/lib/config/defaults.js).

#### urls
Array of objects which contain urls to download and filenames for them. **_Required_**.
```javascript
scrape({
  urls: [
    'http://nodejs.org/',	// Will be saved with default filename 'index.html'
    {url: 'http://nodejs.org/about', filename: 'about.html'},
    {url: 'http://blog.nodejs.org/', filename: 'blog.html'}
  ],
  directory: '/path/to/save'
}).then(console.log).catch(console.log);
```

#### directory
String, absolute path to directory where downloaded files will be saved. Directory should not exist. It will be created by scraper. **_Required_**.

#### sources
Array of objects to download, specifies selectors and attribute values to select files for downloading. By default scraper tries to download all possible resources.
```javascript
// Downloading images, css files and scripts
scrape({
  urls: ['http://nodejs.org/'],
  directory: '/path/to/save',
  sources: [
    {selector: 'img', attr: 'src'},
    {selector: 'link[rel="stylesheet"]', attr: 'href'},
    {selector: 'script', attr: 'src'}
  ]
}).then(console.log).catch(console.log);
```

#### recursive
Boolean, if `true` scraper will follow anchors in html files. Don't forget to set `maxDepth` to avoid infinite downloading. Defaults to `false`.

#### maxDepth
Positive number, maximum allowed depth for dependencies. Defaults to `null` - no maximum depth set.

#### request
Object, custom options for [request](https://github.com/request/request#requestoptions-callback). Allows to set cookies, userAgent, etc.
```javascript
scrape({
  urls: ['http://example.com/'],
  directory: '/path/to/save',
  request: {
    headers: {
      'User-Agent': 'Mozilla/5.0 (Linux; Android 4.2.1; en-us; Nexus 4 Build/JOP40D) AppleWebKit/535.19 (KHTML, like Gecko) Chrome/18.0.1025.166 Mobile Safari/535.19'
    }
  }
}).then(console.log).catch(console.log);
```

#### subdirectories
Array of objects, specifies subdirectories for file extensions. If `null` all files will be saved to `directory`.
```javascript
/* Separate files into directories:
  - `img` for .jpg, .png, .svg (full path `/path/to/save/img`)
  - `js` for .js (full path `/path/to/save/js`)
  - `css` for .css (full path `/path/to/save/css`)
*/
scrape({
  urls: ['http://example.com'],
  directory: '/path/to/save',
  subdirectories: [
    {directory: 'img', extensions: ['.jpg', '.png', '.svg']},
    {directory: 'js', extensions: ['.js']},
    {directory: 'css', extensions: ['.css']}
  ]
}).then(console.log).catch(console.log);
```

#### defaultFilename
String, filename for index page. Defaults to `index.html`.

#### prettifyUrls
Boolean, whether urls should be 'prettified', by having the `defaultFilename` removed. Defaults to `false`.

#### ignoreErrors
Boolean, if `true` scraper will continue downloading resources after error occured, if `false` - scraper will finish process and return error. Defaults to `true`.

#### urlFilter
Function which is called for each url to check whether it should be scraped. Defaults to `null` - no url filter will be applied.
```javascript
// Links to other websites are filtered out by the urlFilter
var scrape = require('website-scraper');
scrape({
  urls: ['http://example.com/'],
  urlFilter: function(url){
    return url.indexOf('http://example.com') === 0;
  },
  directory: '/path/to/save'
}).then(console.log).catch(console.log);
```

#### filenameGenerator
String, name of one of the bundled filenameGenerators, or a custom filenameGenerator function. Filename generator determines where the scraped files are saved.

###### byType (default)
When the `byType` filenameGenerator is used the downloaded files are saved by type (as defined by the `subdirectories` setting) or directly in the `directory` folder, if no subdirectory is specified for the specific type.

###### bySiteStructure
When the `bySiteStructure` filenameGenerator is used the downloaded files are saved in `directory` using same structure as on the website:
- `/` => `DIRECTORY/index.html`
- `/about` => `DIRECTORY/about/index.html`
- `/resources/javascript/libraries/jquery.min.js` => `DIRECTORY/resources/javascript/libraries/jquery.min.js`

```javascript
// Downloads all the crawlable files. The files are saved in the same structure as the structure of the website
// Links to other websites are filtered out by the urlFilter
var scrape = require('website-scraper');
scrape({
  urls: ['http://example.com/'],
  urlFilter: function(url){ return url.indexOf('http://example.com') === 0; },
  recursive: true,
  maxDepth: 100,
  filenameGenerator: 'bySiteStructure',
  directory: '/path/to/save'
}).then(console.log).catch(console.log);
```

#### httpResponseHandler
Function which is called on each response, allows to customize resource or reject its downloading.
It takes 1 argument - response object of [request](https://github.com/request/request) module and should return resolved `Promise` if resource should be downloaded or rejected with Error `Promise` if it should be skipped.
Promise should be resolved with:
* `string` which contains response body
* or object with properies `body` (response body, string) and `metadata` - everything you want to save for this resource (like headers, original text, timestamps, etc.), scraper will not use this field at all, it is only for result.
```javascript
// Rejecting resources with 404 status and adding metadata to other resources
scrape({
  urls: ['http://example.com/'],
  directory: '/path/to/save',
  httpResponseHandler: (response) => {
  	if (response.statusCode === 404) {
		return Promise.reject(new Error('status is 404'));
	} else {
		// if you don't need metadata - you can just return Promise.resolve(response.body)
		return Promise.resolve({
			body: response.body,
			metadata: {
				headers: response.headers,
				someOtherData: [ 1, 2, 3 ]
			}
		});
	}
  }
}).then(console.log).catch(console.log);
```
Scrape function resolves with array of [Resource](https://github.com/s0ph1e/node-website-scraper/blob/master/lib/resource.js) objects which contain `metadata` property from `httpResponseHandler`. 

## callback 
Callback function, optional, includes following parameters:
  - `error`: if error - `Error` object, if success - `null`
  - `result`: if error - `null`, if success - array of [Resource](https://github.com/s0ph1e/node-website-scraper/blob/master/lib/resource.js) objects containing:
    - `url`: url of loaded page
    - `filename`: filename where page was saved (relative to `directory`)
    - `children`: array of children Resources

## Log and debug
This module uses [debug](https://github.com/visionmedia/debug) to log events. To enable logs you should use environment variable `DEBUG`.
Next command will log everything from website-scraper
```bash
export DEBUG=website-scraper*; node app.js
```

Module has different loggers for levels: `website-scraper:error`, `website-scraper:warn`, `website-scraper:info`, `website-scraper:debug`, `website-scraper:log`. Please read [debug](https://github.com/visionmedia/debug) documentation to find how to include/exclude specific loggers.
