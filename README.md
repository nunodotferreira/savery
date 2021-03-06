# savery

### Table of contents
* [Installation](#installation)
* [Usage](#usage)
* [Methods](#methods)
* [Advanced usage](#advanged-usage)
* [Supported browsers](#supported-browsers)
* [Additional information](#additional-information)
* [Development](#development)

### Installation

```
$ npm i savery --save
```

### Usage

```javascript
// ES2015+
import savery from 'savery';

// CommonJS
const savery = require('savery');

// script
const savery = window.savery;

// save the file immediately with a single line
savery.save('.foo { display: block; }', 'foo.css');

// or create use the partial applcation function option with name and options
const saveCssFile = savery('foo.css', {
    onBeforeSave() {
      console.log('Starting!');
    }  
});

// and then pass it the data
const file = saveCssFile('.foo { display: block; }');

file.save(); // to execute the creation of the file from the data
file.abort(); // if you want to abort the request to create the file

// all save executions are chainable
savery.save('.foo { display: block; }', 'foo.css')
    .then((saveryInstance) => {
        console.log('I am complete! Here is the instance to prove it: ', saveryInstance);
    })
    .catch((saveryInstance) => {
        console.log('Oops, something went wrong. :(');
        
        throw saveryInstance.error;
    });
```

As you can see, you can either call `savery.save` to immediately fire the save process, or you can create a partial function which accepts `filename` and `saveryOptions` but returns a function that accepts the `data`. The reason for the partial function is the ability for reuse. For example, if you know the file type and want a consistent filename from a remote download:

```javascript
import axios from 'axios';
import savery from 'savery';

const PDF_NAME = 'my-consistently-named.pdf';

// save the consistent implementation
const pdfFileSavery = savery(PDF_NAME);

// create a function that accepts the data and calls the .save()
const savePdfFile = (data) => {
  return pdfFileSavery(data).save();
};

// in your API call, you can pass it as a simple method
const getPdf = () => {
  axios({
    method: 'get',
    responseType: 'arraybuffer',
    uri: '/location/of/pdf'
  })
    .then(savePdfFile);
};
```

Or if you needed to support a dynamic filename, it could still be a simple one-liner:

```javascript
import axios from 'axios';
import savery from 'savery';

// create a function that accepts the data and calls the .save()
const savePdfFile = (name, options = {}) => {
  const saveryInstance = savery(name, options);
  
  return (data) => {
    return saveryInstance(data).save();
  };
};

// in your API call, you can pass it as a simple method
const getPdf = (dynamicFilename) => {
  axios({
    method: 'get',
    responseType: 'arraybuffer',
    uri: '/location/of/pdf'
  })
    .then(savePdfFile(dynamicFileName));
};
```

With the partial applcation function usage, you have control over the execution while still maximizing code reuse and readability.

### Methods

**savery(filename: string = 'download.txt', saveryOptions: Object = {}): saveryInstance**

The `savery` function accepts the `filename` and `options` parameter, both with defaults. It returns a `saveryInstance` that has the attributes passed applied.

**saveryOptions**
```javascript
{
  onAbort: ?Function,
  onAfterSave: ?Function,
  onBeforeSave: ?Function,
  onError: ?Function,
  onStartSave: ?Function,
  onEndSave: ?Function,
  shouldAutoBom: ?boolean = true,
  type: ?string
}
```

All `saveryOptions` properties are optional, and `type` specifically will default to being intuited from the filename extension and that extension's standard MIME type. The `shouldAutoBom` feature automatically provides Unicode text encoding hints (see [byte-order mark](https://en.wikipedia.org/wiki/Byte_order_mark) for more details).

**saveryInstance.save(): Promise**

This will execute the saving of the file, and return the `Promise` of the file's processing. This allows you to chain the results:

```javascript
const saveryInstance = savery('foo.txt');

saveryInstance.save()
  .then((instance) => {
    console.log(instance); // instance after completion
  })
  .catch((instance) => {
    console.log(instance); // instance after the error completion
    console.error(instance.error); // completion error
  });
```

The lifecycle methods shown above in options are fired as part of the save process in the order expected:
* `onBeforeSave` (before all other execution)
* `onStartSave` (before blob is created)
* `onEndSave` (after blob is created and save is attempted)
* `onAfterSave` (after all other execution)

**saveryInstance.abort(): void**

This will abort the instance execution immediately. This fires two methods in the order shown:

* `onAbort` (immediately upon aborting)
* `onError` (once aborted, internally it fires the `onError` so that the `.catch()` in the chain is also called)

**savery.save(data: any, filename: string = 'download.txt', saveryOptions: Object = {}): Promise**

This will immediate trigger the save process with the `data`, `filename`, and `saveryOptions` passed, and will return the same `Promise` that calling `save()` on a `saveryInstance` will.

### Advanced usage

`savery` uses a list of commonly-used MIME types which is intended to be comprehensive enough for most common use cases, however if you want to include the complete list of MIME types in your package you can do either of the following:

* Build systems (`webpack`, `browserify`, etc.)
  * Add the `SAVERY=full` environment variable to the command that runs your build script
* `<script>` tag
  * Use the `savery-full` script instead of the `savery` script
  
This will tell `savery` to use the [mime-types](https://github.com/jshttp/mime-types) package instead of the local list. 

### Supported browsers

* Full support
  * Firefox 20+
  * Chrome
  * Edge
  * Internet Explorer 10+
  * Opera 15+
* Partial support (filenames not supported)
  * Firefox <20
  * Safari
  * Opera <15
* Requires [Blob polyfill](https://www.npmjs.com/package/blob-polyfill)
  * Firefox <20
  * Opera <15
  * Safari <6

Please note that it is highly recommended to include a `Promise` polyfill, as it will be needed for `savery` to work in the following environments:
* IE 10/11
* IE Mobile 10/11
* Safari <7.1
* iOS Safari <8
* Android browser <4.4.4

### Additional information

The `data` property in `savery` will accept anything for use in the `Blob` construction, however `canvas` elements specifically need to have the data converted by the `canvas.toBlob()` method prior to calling via `savery`.

### Development

Standard stuff, clone the repo and `npm install` dependencies. The npm scripts available:
* `build` => run webpack to build savery-full.js with NODE_ENV=development
* `build:lite` => run webpack to build savery.js with NODE_ENV=development
* `build:minifed` => run webpack to build savery-full.min.js with NODE_ENV=production
* `build:minifed:lite` => run webpack to build savery.min.js with NODE_ENV=production
* `dev` => run webpack dev server to run example app based on full MIME type map provided by `mime-types` package
* `dev:lite` => run webpack dev server to run example app based on local lite MIME type map
* `flow` => run flowtype analysis on `src` folder
* `lint` => run ESLint against all files in the `src` folder
* `prepublish` => run `lint`, `test`, `transpile`, `build`, and `build-minified`
* `test` => run AVA test functions with `NODE_ENV=test`
* `test:watch` => same as `test`, but runs persistent watcher
* `transpile` => run babel against all files in `src` to create files in `lib`
