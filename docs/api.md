# ImJoy API

Every plugin runs in its own sandbox-like container environment (web worker or
iframe for JavaScript, process for Python). This avoids interference with other
plugins and makes the ImJoy App more secure.

Interactions between plugins or between a plugin with the main ImJoy app are carried
out through a set of API functions (`ImJoy API`). All plugins have access
to a special object called `api`. In Javascript, `api` is a global object which you can use directly.
In Python, you can import it by calling `from imjoy import api`. With this object plugins can, for example,
show a dialog, send results to the main app, or call another plugin with
parameters and data.

To make the interaction more efficient and concurrently, we chose a
programming pattern called [asynchronous programming](http://cs.brown.edu/courses/cs168/s12/handouts/async.pdf)
for these API functions.

## Asynchronous programming

All ImJoy API functions are asynchronous. This means when an `ImJoy API` function
is called, ImJoy will not block its execution, but instead it will immediately return an object called [Promise(JS)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) or [Future (Python)](https://docs.python.org/3/library/asyncio-future.html). You can decide to wait for the actual result or set a callback function to retrieve the result once the process terminated. For example, if you popup a dialog to ask for user input, in many programming languages (synchronous programming), the code execution will be blocked until the user closes the dialog. However, an asynchronous program will return the `promise` object even if the user didn't close the dialog and continue processing.

Since every API call is asynchronous and non-blocking, a given plugin can call multiple other plugins to perform tasks simultaneously without using thread-like techniques.

ImJoy supports two asynchronous programming styles to access these asynchronous functions
for both Python and JavaScript: `async/await` and `callback` style. A few important
considerations

* `async/await` is recommended for JavaScript and Python 3.
* `callback` style can be used for JavaScript, Python 2 and Python 3.
*  **Note** that you **cannot** use both style at the same time.
* While you can use `try catch` (JavaScript) or `try except` (Python) syntax to capture error with `async/await` style, you cannot use them to capture error if you use `callback` style.

In the following list of API functions, we provided examples in `async` style. For Python 2, you can easily convert to callback style accordingly.


### async/await style
This style is natively supported and recommended for Javascript and Python 3+ plugins.

Declare your function with the `async` keyword. Add `await` before the asynchronous
function to wait for the result. This essentially allows synchronous style
programming without the need to set callbacks.

Below is a simple example for an api function named `XXXXX`. Please note that that you can **only**
use `await` when you add `async` before the function definition. For Python3,
don't forget to `import asyncio`.

<!-- tabs:start -->

#### ** JavaScript **
```javascript
 class ImJoyPlugin(){
  async setup(){
  }
  async run(ctx){
    try{
      result = await api.XXXXX()
      console.log(result)
    }
    catch(e){
      console.error(e)
    }
  }
}
```

#### ** Python **
```python
import asyncio
from imjoy import api

class ImJoyPlugin():
    async def setup(self):
        pass

    async def run(self, ctx):
        try:
            result = await api.XXXXX()
            print(result)
        except Exception as e:
            print(e)
 ```
<!-- tabs:end -->



### callback style
However, for Python 2 or Web Python, `asyncio` is not supported, therefore
you need to use `callback` style.

Call the asynchronous function and set its callback with `.then(callback_func)`.
For JavaScript plugins, a native JavaScript `Promise` will be returned ([More about Promise.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)). For Python plugins,
it will return a simplified Python implement of promise.

Below examples for an api function named `XXXXX`:

<!-- tabs:start -->
#### ** JavaScript **

```javascript
class ImJoyPlugin(){
  setup(){
  }

  run(ctx){
      api.XXXXX().then(this.callback)

      // optionally, you can catch error
      const error_callback(error){
        console.error(error)
      }
      api.XXXXX().then(this.callback).catch(error_callback)
  }

  callback(result){
     console.log(result)
  }
}
```

#### ** Python **
```python
from imjoy import api
class ImJoyPlugin():
    def setup(self):
        pass

    def run(self, ctx):
        api.XXXXX().then(self.callback)

        # this function will be called when we get results from api.XXXXX()
        def result_callback(result):
            print(result)

        # optionally, you can catch an error for the call
        def error_callback(error):
            print(error)

        api.XXXXX().then(result_callback).catch(error_callback)
```
<!-- tabs:end -->


## Input arguments
When calling the API functions, most functions take an object (JavaScript)
or dictionaries (Python) as its first argument. for `native-python` plugins.

The following function call will work both in JavaScript and Python:

```javascript
// JavaScript or Python
await api.XXXXX({"option1": 3, "option2": 'hi'})
```

This call will work only for Python:
```python
# Python
await api.XXXXX(option1=3, option2='hi')
```

## API functions
For each api function we provide a brief code snippet illustrating how this
function can be used. Below, you will find links **Try yourself >>**. These will
open a full example in ImJoy, where see the function in action. The examples are
in JavaScript, but the api functions are called in similar fashion in Python.

### api.alert

```javascript
await api.alert(message)
```

Shows an alert dialog with a message to the user.

**Arguments**
<!--****[TODO] add instructions about customizable parameters **-->

For plain text:

* **message**: String. Contains the message to be displayed. HTML tags can be used in the message, but only limited to a restricted set of tags and css, more details can be found [here](api?id=sanitized-html-and-css).

For HTML:

* **message**: Object. It contains the following fields:
  - **content**: Contains the question to be displayed. HTML tags can be used in the message, but only limited to a restricted set of tags and css, more details can be found [here](api?id=sanitized-html-and-css).
  - **title**: The title of the dialog


**Example**
```javascript
await api.alert('hello world')
```
[Try yourself >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:alert&w=examples)

### api.prompt

```javascript
const answer = await api.prompt(question, default_answer)
```

Shows a prompt to ask the user for input.

**Arguments**
<!--****[TODO] add instructions about customizable parameters **-->

For plain text:

* **question**: String. Contains the question to be displayed.  HTML tags can be used in the message, but only limited to a restricted set of tags and css, more details can be found [here](api?id=sanitized-html-and-css).

For HTML:

* **question**: Object. It contains the following fields:
  - **content**: Contains the question to be displayed. HTML tags can be used in the message, but only limited to a restricted set of tags and css, more details can be found [here](api?id=sanitized-html-and-css).
  - **placeholder**: The default answer of the question
  - **title**: The title of the dialog

* **default_answer** (optional): String. Contains the default answer to the question.

**Returns**
* **answer**: Boolean. The answer from the user.

**Example**
```javascript
const answer = await api.prompt('What is your name?')
```
<!--****[TODO] add example **-->

### api.confirm

```javascript
const confirmation = await api.confirm(question)
```

Shows a confirmation message to the user.

**Arguments**
<!--****[TODO] add instructions about customizable parameters **-->

For plain text:

* **question**: String. Contains the question to be displayed.  HTML tags can be used in the message, but only limited to a restricted set of tags and css, more details can be found [here](api?id=sanitized-html-and-css).

For HTML:

* **question**: Object. It contains the following fields:
  - **content**: Contains the question to be displayed. HTML tags can be used in the message, but only limited to a restricted set of tags and css, more details can be found [here](api?id=sanitized-html-and-css).
  - **title**: The title of the dialog

**Returns**
* **confirmation**: Boolean. True or false.


**Example**
```javascript
const confirmation = await api.confirm('Do you want to delete these files?')
if(confirmation){
  delete_file()
}
else{
  console.log('User cancelled file deletion.')
}
```
<!--****[TODO] add example **-->

### api.call
```javascript
result = await api.call(plugin_name, plugin_op, arg1, arg2 ...)
```

Calls a function from another plugin.

You can also transfer data. Number of argument number must match the required number
of arguments of the called function.

If you want to call frequently functions of another plugin, we recommend using `api.getPlugin`.

**Arguments**

* **plugin_name**: String. Name of the called plugin.
* **plugin_op**: String. Name of plugin function (op).
* **args** (optional): Any of the supported primitive data types that can be transferred.

**Returns**
* **result**: The result returned from the plugin function if any.

**Example**

To call a function `funcX` defined in the plugin  `PluginX` with the argument `1`, use

```javascript
await api.call("PluginX", "funcX", 1)
```
[Try yourself >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:call&w=examples)


### api.createWindow
```javascript
win = await api.createWindow(config)
```
Creates a new window in the ImJoy workspace.

Once an window is created, it will return an object with the APIs of the corresponding window plugin,
which can be used to update the window, e.g. you can update the data field with `win.run({"data": ...})`).

**Note 1**: calling functions of a closed window. If the window is closed and you try to call its function, you will get an error, one solution to this is to use `try ... catch `(JavaScript) or `try: ... except: ...`(Python) statement to capture the error.

**Note 2:** difference between `api.createWindow` and `api.getPlugin`. Both
functions can be used to obtain an object containing the plugin api for `window`
plugins. However, only the object obtained by `api.createWindow` can be used to
update an existing window. In contrast, the object returned by `api.getPlugin`
will create a new window each time used.

This difference in behaviour can be explained by the way ImJoy handles the different
plugin types. When ImJoy loads a plugin which is not `window` plugin, it will start
the appropriate standalone python process, webworker or iframe. This ensures that
only one instance of the plugin is running. In contrast, `window` plugins will
only be registered and a proxy plugin will be created. No actual instance will
be started unless the user clicks the plugin menu or the plugin is called by
another plugin. Each window in the workspace is then a new instance of the `window` plugin.

When `api.getPlugin` is called, it will return the api of the proxy plugin, e.g.
`proxy = await api.getPlugin({'name': 'Image Window'})`). Every time the `run` function
is executed, a new window will be created. For example, if you run `proxy.run({data: ...})`
for 10 times, 10 windows will be created. 

To retrieve the window created with `api.createWindow` (or `api.showDialog`), you can use `api.getWindow`.

When `api.createWindow` is used, it will return an instance of the window plugin,
e.g. `win = await api.createWindow({'name': 'new window', 'type': 'Image Window', 'data': {...}})`).
If you run `win.run({'data': ...})` for 10 times, the same window instance will be updated.
Run `win.close()` will close the window.


**Arguments**

* **config**. String or Object. Options for creating window. When it's a string, it will be converted into an object. The conversion is done according to the following rule: 1) if the string contains multiple line, is a URL or plugin URI, then it will be treated as window plugin source (see the `src` key below); 2) otherwise it will be treated as the window plugin type (see the `type` key below).

It contains the following fields:

  - **name**: String. Specifies the name of new window.
  - **type**: String. Specifies the window type. This can be either the name of
   window plugin, an internal ImJoy window type or `external`. The following internal window types are
   supported:
    - `imjoy/generic`. Will show all objects in `data`.
    - `imjoy/image`. Display an image. Requires `data.src` pointing to an image location.
    - `imjoy/image-compare`. Displays two images that can be compared with a slider. Images are passed as `data.first` and `data.second`. See example below.
    - `imjoy/panel`. Render the `ui` string in a `<config>` block. <!--****[TODO] what can you do with this?**-->
    - `imjoy/markdown`. Render some markdown text provided in `data.source`.
    - `imjoy/plugin-editor`. Opens the source code editors. `data.id` is a unique string (preferable random) specifying the window id, `data.code` contains the source code

    The value of `type` if you create the window from an external url, for example, a web app hosted on Github pages. In that case you need to specify the url with the `src` key. Details about how to support ImJoy from an external web app can be found [here](https://github.com/imjoy-team/imjoy-core/).
  
    If the external web page has the ImJoy plugin api loaded, you can interact with the external website like a normal ImJoy plugin. However, if the external web page does not support ImJoy, you need to set `passive=true` to tell ImJoy that there will be no plugin api from this window.
  
  - **src**: String, specify source code to the window plugin, the url to the window plugin source code, or a url to a web app that optionally support `imjoy-rpc`. The url will treated as source code if the url ends with `.imjoy.html`, is a `gist` url or a github source code page url. Passing source code to create a window allows, for example store the source code of a window plugin inside a Python plugin, and instantiate it when needed.
  - **tag**: String. Used with `src` to specify the tag of the plugin if the plugin support several `tags`.
  - **namespace**: String. Used with `src` to specify the namespace of the plugin.
  - **passive**: Boolean, only used when `src` is specified. Mark whether the plugin is a passive web page (no ImJoy api exposed). Default value is `false`. 
  - **w**: Integer. Window width in grid columns (1 column = 30 pixels).
  - **h**: Integer. Window height in grid rows (1 row = 30 pixels).
  - **config**: Object (JavaScript) or dictionary (Python).
  - **data**: Object (JavaScript) or dictionary (Python). Contains data to be transferred to the window.

**Returns**

* **win**: Object. The API object of the created window. Can be stored in the plugin class
  (`self` for Python, or `this` for JavaScript) for later use, e.g. to update the window.
  Note: If `type="external"` and `passive=true`, there will be no api exposed from the window, and `win` will be empty.

**Examples**

Create a simple window in JavaScript:
```javascript
win = await api.createWindow({name: 'new window', type: 'Image Window', w:7, h:7, data: {image: ...}, config: {}})
```
[Try yourself >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:createWindow&w=examples)

In python you can either use the `async/await` style for Python 3

```python
win = await api.createWindow(name='new window', type='Image Window', w=7, h=7, data={image: ...}, config={})
```

or the `callback` style for Python 2
```python
def window_callback(win):
    self.win = win
    print('window created')

api.createWindow({name: 'new window', type: 'Image Window', w:7, h:7, data: {image: ...}, config: {}}).then(window_callback)
```

Use the returned object to update the window, or use `win.on('close', callback)` to set a callback
function which will be called when the window is closed. Similarly, `win.on('resize', callback)` can be used to set callbacks which will be called when the window size is changed.

To close the created window, call `win.close()`.

To scroll to the window in the ImJoy workspace, call `win.focus()`.

Inside the window plugin, `this.close`, `this.on`, `this.emit`, `this.focus`, `this.resize` can be used.

<!-- tabs:start -->

#### ** JavaScript **
```javascript
// win is the object returned from api.createWindow
await win.run({'data': {'image': ...}})

// set `on-close` callback
win.on('close', ()=>{
  console.log('closing window.')
})
```

Create a window with two images and a comparison slider.
```python
  api.createWindow({
    name: 'test compare',
    type: 'imjoy/image-compare',
    data: {
      first: '//placehold.it/350x150/0288D1/FFFFFF',
      second: '//placehold.it/350x150/E8117F/FFFFFF'
    }
  })
```

[Try yourself >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:createWindowImgComp&w=examples)

#### ** Python **
```python
# win is the object returned from api.createWindow
# pass as a dictionary
await win.run({'data': {'image': ...}})

# or named arguments
await win.run(data={'image': ...})

# set `on-close` callback
def close_callback():
    print('closing window.')

win.on('close', close_callback)
```
<!-- tabs:end -->


### api.error

```javascript
api.error(message)
```

Log a error message for the current plugin, which is stored in its log history.

The presence of an error message is indicated with a red icon next to
the plugin name. Pressing on this icon will open a window showing the log history.

Similar to `console.error` or `print`, `api.error` can accept multiple arguments, which will be concatenated with a space.

**Arguments**

* **message**: String. Error message to be logged.

**Examples**

```javascript
api.error('Error occurred during processing.')
```


### api.echo

```javascript
api.echo(obj)
```

An echo function that returns the same object as passed in, used for testing purposes.

This is useful for testing the encoding / decoding object.


**Arguments**

* **obj**: Any.

**Examples**

```javascript
ret = await api.echo('hi')
console.log(ret) // should get 'hi'
```

### api.registerCodec

```javascript
api.registerCodec(config)
```

Register a custom codec for sending and receiving remote object.

!> Note: not supported in `web-python` plugins yet.

**Arguments**

* **config**: Object (JavaScript) or dictionary (Python). Options for the codec.
It contains the following fields:
  - **name**: String. Name of the codec
  - **type**: Class. A class object that used to match the object for encoding. In Javascript, `instanceof` will be used to match the type. In Python `isinstance()` will be used, that also means in Python, `type` can be an tuple of classes.
  - **encoder**: Function. The `encoder` function take an object as input and you need to return the represented object/dictionary. Note that, you can only use primitive types plus array/list and object/dict in the represented object. By default, if your returned object does not contain a key `_rtype`, the codec `name` will be used as `_rtype`. You can also assign a different `_rtype` name, that allows the conversion between different types.
  - **decoder**: Function. The `decoder` function converts the encoded object into the actual object. It will only be called when the `_rtype` of an object matches the `name` of the codec.

**Examples**

```javascript
class Cat{
  constructor(name, color, age, clean){
    this.name = name
    this.color = color
    this.age = age
    this.clean = clean
  }
}

api.registerCodec({
    'name': 'cat', 
    'type': Cat, 
    'encoder': (obj)=>{
        // convert the Cat instance as a dictionary with all the properties
        return {name: obj.name, color: obj.color, age: obj.age, clean: obj.clean}
    },
    'decoder': (encoded_obj)=>{
        // recover the Cat instance
        return new Cat(encoded_obj.name, encoded_obj.color, encoded_obj.age, encoded_obj.clean)
    }
})

class Plugin {
    async setup(){
    }
    async run(){
        const dirtyCat = new Cat('boboshu', 'mixed', 0.67, false)
        // assuming we have a shower plugin
        const showerPlugin = await api.getPlugin({'name': 'catShower'})
        // now pass a cat into the shower plugin, and we should get a clean cat, the name should be the same
        // note that the other plugin is running in another sandboxed iframe or in Python
        // because we have the cat codec registered, we can send the Cat object to the other plugin
        // Also notice that the other plugin should also define custom encoding decoding following the same representation
        const cleanCat = await showerPlugin.wash(dirtyCat)
        if(cleanCat.clean) api.alert(cleanCat.name + ' is clean.')
    }
};
api.export(new Plugin())
```


### api.echo

```javascript
api.echo(obj)
```

An echo function that returns the same object as passed in, used for testing purposes.

This is useful for testing the encoding / decoding object.


**Arguments**

* **obj**: Any.

**Examples**

```javascript
ret = await api.echo('hi')
console.log(ret) // should get 'hi'
```

### api.disposeObject

```javascript
api.disposeObject(obj)
```

Remove the remote object from its object store, such that it can be recycled by the garbage collector.

It is important that this function is called when you don't need a remote object anymore, 
otherwise, it will cause memory leak since the object will remain in its object store.

**Arguments**

* **obj**: Object. To remote object to be disposed.


### api.export

Exports funcstions defined by the plugin as `Plugin API`. Every imjoy plugin should export its plugin api functions, except when `passive` key under `<config>` is set to `true`.

`Plugin API` can be exported as a plugin class or an object/dictionary contains all the api functions:

<!-- tabs:start -->
#### ** JavaScript (class) **
```javascript
class ImJoyPlugin(){
  async setup(){
  }
  async run(ctx){
  }
}

api.export(new ImJoyPlugin())
```
#### ** JavaScript (functions) **
```javascript
function setup(){

}

function run(){

}

api.export({setup: setup, run: run})
```

#### ** Python (class) **
```python
class ImJoyPlugin():
    def setup(self):
        pass

    def run(self, ctx):
        pass

api.export(ImJoyPlugin())
```

#### ** Python (functions) **
```python
def setup():
    pass

def run(ctx):
    pass

api.export({'setup': setup, 'run': run})
```
<!-- tabs:end -->

This call is mandatory for every ImJoy plugin (typically as the last line of the plugin script).
Every member of the `ImJoyPlugin` instance will be exported as `Plugin API`, which means those exported functions can be called or used by the ImJoy app or another plugin.
This then allows other plugins to use `api.run` or `api.call` to call functions of the plugin.

Only functions and variables with primitive types can be exported (number, string, boolean).
And if a variable or function has a name start with `_`, it means that's an internal variable or function, will not be exported.

**Note** that in JavaScript, the `new` keyword is necessary to create an
instance of a class, while in Python there is no `new` keyword.


### api.init
```javascript
api.init(config)
```

Initialize with a minimal plugin interface and config. This can be used as a shortcut for `api.export` when you don't want to export any plugin api.

In Python for example:
```python
api.init(config)
```

Will be equivalent to:
```
def setup():
    pass

api.export({"setup": setup}, config)
```

**Arguments**
* **config**: Object, optional. Configuration for the plugin, with all the config fields including `name`, `type` (a full list can be found [here](https://imjoy.io/docs/#/development?id=ltconfiggt-block)).


### api.exportFile
```javascript
api.exportFile(file, name)
```

Trigger a download for a file from the browser.

**Arguments**
* **file**: File, Blob or String. The file object to be downloaded.
If a string is passed, it will be wrapped as a text file.

**Returns**
* **name**: String. The file name.

**Examples**

```javascript
var blob = new Blob(["Hello, world!"], {type: "text/plain;charset=utf-8"});
api.exportFile(blob, 'hello.txt')
```

### api.getAttachment
```javascript
content = await api.getAttachment(att_name)
```
Retrieve data stored in the `<attachment>` block of the plugin file.

You can store any text data such as base64 encoded images, code and json in the `<attachment>` block.

**Arguments**
* **att_name**: String. Identifier of the attachment.

**Returns**
* **content**: The text content stored in the `<attachment>` block.


**Examples**

```html
<attachment name="att_name"></attachment>
```
```javascript
content = await api.getAttachment(att_name)
```

### api.getConfig
```javascript
config_value = await api.getConfig(config_name)
```
Retrieves configurations for plugin.

**Note 1:** numbers are converted to strings when saved with `api.setConfig`. They have to be converted back to numbers before using them (in JavaScript use `parseInt()` or `parseFloat()`, in Python `int()` or `float()`).

**Note 2:** you can also access the fields defined in the `<config>` block by adding `_` to the field name, for example, if you want to read the plugin name `defined` in the `<config>` block, you can use `plugin_name = await api.getConfig('_name')`.

**Arguments**
* **param_name**: String. Name of parameter.

**Returns**

* **param**: String. Returned parameter value. Please note that numbers will also be returned as string.

**Examples**

```javascript
sigma = await api.getConfig('sigma')
```
[Try yourself in the setConfig example >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:setConfig&w=examples)


### api.installPlugin
```javascript
plugin = await api.installPlugin(config)
```

Install a plugin by passing a plugin URI or source code, the plugin source code will be saved to the browser database ( in the current workspace).

**Arguments**
* **config**: Object. configuration object. Currently, you can pass the following config:
  - `src`: String. The source code, URI of the plugin to be installed. 
  - `tag`: String, optional. The selected plugin tag if the plugin has multiple tags.
  - `namespace`: String, optional. The namespace of the plugin.

**Examples**

```javascript
await api.installPlugin({uri: "https://raw.githubusercontent.com/imjoy-team/imjoy-core/master/src/plugins/webWorkerTemplate.imjoy.html"})
```

### api.uninstallPlugin
```javascript
plugin = await api.uninstallPlugin(config)
```

Uninstall an installed plugin.

**Arguments**
* **config**: Object. configuration object. Currently, you can pass the following config:
  - `name`: String. The name of the plugin to be uninstalled.
  - `namespace`: String, **TODO: NOT SUPPORTED YET**. The namespace of the plugin.

**Examples**

```javascript
await api.uninstallPlugin({name: "MyAwesomePlugin"})
```


### api.getPlugin
```javascript
plugin = await api.getPlugin(config)
```

Gets the API object of a loaded plugin by its id or name. The plugin must be already loaded into the workspace.

**Note 1:** If the plugin is terminated and you try to call its function, you will get an error. One solution to this is to use `try ... catch `(JavaScript) or `try: ... except: ...`(Python) statement to capture the error. <!--(TODO: this does not work for `native-python` plugin yet.)-->

**Note 2** on `api.getPlugin` and `api.call`. If you want to constantly access
different functions from another plugin, it is preferable to get all the API
functions of this plugin with `api.getPlugin`. Then you can access them
through the returned object. If you only access the API function in another
plugin occasionally, you can also use `api.call`

**Arguments**

* **config**: String or Object. If it's a string, then it should be the name of the plugin, otherwise, you can pass an object contains the key `id` or `name`.

Currently, you can pass the following config:
  - **name**: String. Name of the plugin.
  - **id**: String. Id of the plugin.

**Returns**
* **plugin**: Object. An object which can be used to access the plugin API functions.

**Example**

Get the API of the plugin `PluginX`, and access its functions:

```javascript
pluginX = await api.getPlugin("PluginX")
result = await pluginX.run()

// Assuming that PluginX defined an API function `funcX`, you can access it with:
await pluginX.funcX()
```
[Try yourself >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:getPlugin&w=examples)


### api.loadPlugin
```javascript
plugin = await api.getPlugin(config)
```

Load an plugin from source code or URL, then return the plugin API object.

**Arguments**

* **config**: String or Object. Configuration for getting the plugin. If it's a string, then depending on the content of the string it will be converted into a config object. The conversion is done according to the following rule: 1) if the string contains multiple line, is a URL or plugin URI, then it will be treated as plugin source (see the `src` key below); 2) otherwise it will be treated as a plugin name and return the same result as `api.getPlugin`.

Currently, you can pass the following config:
  - **src**: String. URL or source code of the plugin, in this case it will be instantiate on-the-fly. Otherwise, pass a valid plugin URI or its source code. By passing the source code, it allows the flexibility of embedding one or more plugin source code inside another plugin. For example, a Python plugin can dynamically populate 
a window plugin in HTML.
  - **tag**: String, optional. Specify the tag of the plugin if the plugin support several `tags`, only used when `src` is the source code of the plugin.
  - **namespace**: String, optional. Specify the namespace of the plugin, only used when `src` is the source code of the plugin.
  - **engine_mode**: String, optional. **Only for plugin that runs via an Engine** Select the default engine mode, it can be `auto` or an engine URL (e.g.: `https://mybinder.org`). 

**Returns**
* **plugin**: Object. An object which can be used to access the plugin API functions.

**Example**

```javascript
const pokemonChooser = await api.loadPlugin({src: "https://gist.github.com/oeway/3c2e1ee72c79a6aafd9d6e3b473f0bbf"})
const result = await pokemonChooser.choosePokemon()
```
### api.getServices
```javascript
services = await api.getServices(config)
```

Gets a list of plugin services (registered with `api.registerService()`) by specifying their `name`, `type`, `id` etc.


**Arguments**

* **config**: Object. It's a query object that optinally consists of the several fields (at least one):
  - `id`: String. ID of the service (if matched, will return a list with one element).
  - `name`: String. Name of the services.
  - `type`: String. Type of the services.
  - any other keys that defined in the service api.

**Returns**
* **services**: Object. A list of service api or object.

**Example**

Get all the plugin services registered with `type="model"`:

```javascript
const models = await api.getServices({type: "model"})

console.log(models)
```

### api.getWindow
```javascript
w = await api.getWindow(config)
```

Get an existing window by its id, window_id, name or type.


**Arguments**

* **config**: String or Object. It can be a window name string, or an object that optinally consists of the several fields (at least one):
  - `name`: String. Name of window.
  - `type`: String. Type of the window.
  - `window_id`: String. The id of the window.
  - `plugin_id`: String. The id of the plugin instance attached to the window.

**Returns**
* **w**: Object. An windowobject which can be used to access the window API functions.


**Example**

Get an existing [Kaibu](https://kaibu.org) window and access it.

```javascript
await createWindow({name: "My Kaibu Window", src: "https://kaibu.org"})

w = await api.getWindow("My Kaibu Window")
await w.open("https://imjoy.io/static/img/imjoy-icon.png")
```


### api.getEngine
```javascript
engine = await api.getEngine(engine_url)
```

Gets the API object of an plugin engine.

**Arguments**

* **engine_url**: String. URL of the plugin engine.

**Returns**
* **engine**: Object. An engine object which can be used to access the engine API functions.

**Example**

Get the API of the engine (url = `https://127.0.0.1:2957`), and access its functions:

```javascript
engine = await api.getEngine("https://127.0.0.1:2957")
await engine.disconnect()
```

### api.getEngineFactory
```javascript
engine_factory = await api.getEngineFactory(engine_factory_name)
```

Gets the API object of an plugin engine factory.

**Arguments**

* **engine_factory_name**: String. Name of the plugin engine factory.

**Returns**
* **engine_factory**: Object. An plugin engine factory object which can be used to access the engine API functions.

**Example**

Get the API of the plugin engine factory (name = `ImJoy-Engine`), and access its functions:

```javascript
engine_factory = await api.getEngineFactory("ImJoy-Engine")
await engine_factory.addEngine(config)
```


### api.getFileManager
```javascript
file_manager = await api.getFileManager(file_manager_url)
```

Gets the API object of an file manager.

Note: since `api_version > 0.1.6`, `api.getFileUrl` and `api.requestUploadUrl` are deprecated, the replacement solution is to use `api.getFileManager` to get the file manager first, and access `getFileUrl` and `requestUploadUrl` from the returned file manager object.

**Arguments**

* **file_manager_url**: String. URL of the file manager.

**Returns**
* **file_manager**: Object. An file manager object which can be used to access the file manager API functions.

**Example**

Get the API of the file manager (url = `https://127.0.0.1:2957`), and access its functions:

```javascript
file_manager = await api.getFileManager("https://127.0.0.1:2957")
await file_manager.listFiles()
```

Get file URL for downloading (replacement of `api.getFileUrl`)
```javascript
file_manager = await api.getFileManager("https://127.0.0.1:2957")
await file_manager.getFileUrl({'path': './data/output.png'})
```


If you want to get the file URL of your current file
<!-- ImJoyPlugin: { "type": "native-python", "name": "my-python-plugin"} -->
```
from imjoy import api


class ImJoyPlugin():
    def setup(self):
        api.log('initialized')

    async def run(self, ctx):
        file_manager = await api.getFileManager(api.config.file_manager)
        url = await file_manager.getFileUrl({"path": './screenshot-imjoy-notebook.png'})
        await api.alert(url)

api.export(ImJoyPlugin())
```

Request file URL for uploading (replacement of `api.requestUploadUrl`)
```javascript
file_manager = await api.getFileManager("https://127.0.0.1:2957")
await file_manager.requestUploadUrl({'path': './data/input.png'})
```

### api.log

```javascript
api.log(message)
```

Log a status message for the current plugin, which is stored in its log history.

The presence of a status message is indicated with a grey icon next to
the plugin name. Pressing on this icon will open a window showing the history of
logged messages.

Status message can either be a string or an image. The latter can be used to create
an automated log, for example, to log the training of a neural network.

Similar to `console.log` or `print`, `api.log` can accept multiple arguments, which will be concatenated with a space.

**Arguments**

* **message**: String. Message to be logged.value

**Examples**

Create a simple text message:
```javascript
api.log('Processing data ...')
```

Log a an image file.
```javascript
api.log({type: 'image', value: 'https://imjoy.io/static/img/imjoy-icon.png' })
```


### api.progress
```javascript
api.progress(progress)
```
Updates the progress bar of the current plugin.

This progress bar will be displayed in the plugin menu itself. Please use
`api.showProgress` to generate a progress bar for the ImJoy status bar.

**Arguments**

* **progress**: Float or Integer. Progress in percentage. Allowed range from 0 to 100 for Integer, or 0 to 1 for float.

**Examples**
```javascript
api.progress(85)
```
[Try yourself >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:progress&w=examples)


### api.registerService
```javascript
const service_id = await api.registerService(config)
```

Register a plugin service.

**Arguments**

* **config**: Object (JavaScript) or dictionary (Python). It must contain a least a `type` key and a `name` key. Other keys depend on the corresponding type definition (See below).

**Returns**
* **service_id**: String. A service id which can be used to get or unregister the service.


#### Built-in plugin services

operator service (type=`operator`) is a type of built-in service that used to extend the plugin menu to perform a specific task.

For the service `config`, several fields are allowed:
  - `name`: String. Name of op.
  - `ui`: Object (JavaScript) or dictionary (Python). Rendered interface. Defined
       with the same rule as the `ui` field in `<config>`.
  - `run`: Function, optional. Specifies the `Plugin API` function that will run when
      the op is executed. Note that it must be a member of the plugin class or a function being exported with `api.export`. If not specified, the `run` function of the plugin will be executed.
  - `update`: String, optional. Specifies another `Plugin API` function that will run
      whenever any option in the `ui` is changed.
  - `inputs`: Object, optional. A [JSON Schema](https://json-schema.org/) which defines the inputs of this op.
  - `outputs`: Object, optional. A [JSON Schema](https://json-schema.org/) which defines the outputs of this op.

(Please also consult [this section](api?id=input-arguments) for how arguments can be set.)

An op can have its own GUI defined by the `ui` string. By default, all ops of a plugin will call the `run` function of the plugin. You can use `ctx.config.type` in the `run` function to differentiate which op was called.

If you want to change your interface dynamically, you can run `api.registerService`
multiple times to overwrite the previous version. `api.registerService` can also be used to overwrite the default ui string of the plugin defined in `<config>`, just set the plugin name as the op name (or without setting a name).

[TODO: finalize `engine`, `engine-factory` and `file-manager` service api and add the docs here ]

**Examples**

Example for registering a new plugin operator:
```javascript
// JavaScript
await api.registerService({
     "type": "operator",
     "name": "LUT",
     "ui": [{
        "apply LUT": {
            id: 'lut',
            type: 'choose',
            options: ['hot', 'rainbow'],
            placeholder: 'hot'
          }
      }],
      "run": this.apply_lut,
      "update": this.update_lut
});

```
[**Try yourself >>**](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:register&w=examples) Compare how the ops for favourite number and animal are implemented.

#### Contributed plugin services

You can use plugin services the following service types:
 * type=`transformation`: [scikit-learn compatible dataset transformations](https://scikit-learn.org/stable/data_transforms.html)
   - `transform(data)`: applies this transformation model to unseen data
   - `fit(data)`: learns model parameters from data
   - `fit_transform(data)`: modelling and transforming the training data simultaneously
 * type=`model`: keras compatible model service
   - `predict(data)`: make predictions on data (see [here](https://keras.io/api/models/model_training_apis/#predict-method))
   - `fit(data)`: train model on data (see [here](https://keras.io/api/models/model_training_apis/#fit-method))
 * [...contribute your service type here...]

#### Define a new plugin service type
If both the built-in and contributed service types do not meet your requirements, you can define a new plugin service type. The easest way is to start by defining a type name starts with `#`, for example: `api.registerService({"type": "#my-awesome-service", ...})`. You can use this to develop and test your service. Once your service type definition is stable, you can submit your type to imjoy-core repo such that you can remove the leading `#` from the type name.

Here are the steps for submitting your type definition:
 1. fork the [imjoy-core repo](https://github.com/imjoy-team/imjoy-core)
 1. editing [this page](https://github.com/imjoy-team/imjoy-core/blob/master/docs/api.md) add the new type to the above list with detailed description
 2. define the schema in the [serviceSpec.js](https://github.com/imjoy-team/imjoy-core/blob/master/src/serviceSpec.js) file.
 3. submit a PR to the [imjoy-core repo](https://github.com/imjoy-team/imjoy-core).

### api.run
```javascript
await api.run(plugin_name)
```
Run another plugin by specifying its name.

You can also pass [`ctx`](development?id=plugin-during-runtime)
to this plugin to transfer data.

**Arguments**

* **plugin_name**: String. Plugin name.

**Examples**


Example to call one plugin:

```python
await api.run("Python Demo Plugin")
```
[**Try yourself >>**](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:run&w=examples)

Example for concurrent execution of two plugins, where the two plugins are
executed simultaneously, but ImJoy waits for the result one after the other.

```python
# Python
p1 = api.run("name of plugin 1")
p2 = api.run("name of plugin 2")

result1 = await p1
result2 = await p2
```

This can also be achieved with `asyncio.gather` in Python:

```python
p1 = api.run("name of plugin 1")
p2 = api.run("name of plugin 2")
result1, result2 = await asyncio.gather(p1, p2)
```

Similarly for JavaScript, you can use

```javascript
const p1 = api.run("name of plugin 1")
const p2 = api.run("name of plugin 2")
const [result1, result2] = [await p1, await p2]
```

Example for sequential execution of two plugins
```python
result1 = await api.run("name of plugin 1")
result2 = await api.run("name of plugin 2")
```



### api.setConfig
```javascript
api.setConfig(config_name, config_value)
```
Store plugin data in its configuration.

These values can be retrieved when ImJoy is restarted. This function is ideally suited to store and reload settings. However, the  function  is designed for storing small amounts of data, not large objects. The current implementation uses `localStorage` for storage.
Most browsers only allow 5MB data storage shared by all the plugins and
the ImJoy app itself.

To remove a parameter, set its value to `null` (JavaScript) or `None` (Python).

**Arguments**
* **config_name**: String. Name of parameter. Don't use names starting with an `_` followed by
any of the field name of the `<config>` block.
* **config_value**: Number or String. Neither objects/arrays (JS) nor dict/list (Python). Please note that numbers are stored as strings.

**Examples**
```javascript
api.setConfig('sigma', 928)
```
[Try yourself >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:setConfig&w=examples)


### api.showDialog
```javascript
answer = await api.showDialog(config)
```

Show a window or customized GUI as a dialog.

Similar to `api.createWindow`, you can pass an object `{"type": "WINDOW_PLUGIN_NAME", "name": "new dialog", "config": {...}, "data": {...}}`. This will show the window plugin instance as a dialog. The dialog can be closed programmatically with `win.close()` or by the user with the close button.

For a simple dialog with joy ui, you can pass `{"type": "joy", "name": "new dialog", "config": {...}, "data": {...}}`. The answer is stored in the returned object, and can be retrieved with the specified `id`. To consider the case when the user presses `cancel`, you can use the `try catch` (JavaScript) or `try except` (Python) syntax.

**Arguments**
* **config**. Object (JavaScript) or dictionary (Python). Specifies the dialog.
    Contains following fields:
    - `name`: String. Title of dialog.
    - `type`: String. Type of the dialog (use the window plugin name or `joy`).

If `type="joy"`, you need to provide `ui` which is defined with the same rule as the `ui` field in `<config>`. Otherwise, you need to provide `config` and `data` as `api.createWindow`.

(Please also consult [this section](api?id=input-arguments) for how arguments can be set.)

**Returns**
* **answer**. Object (JavaScript) or dictionary (Python). Contains provided answer as field `answer[id]`.

**Example**
```javascript
result = await api.showDialog({
   "name": "This is a dialog",
   "ui": "Hey, please select a value for sigma: {id:'sigma', type:'choose', options:['1', '3'], placeholder: '1'}.",
})
```
[Try yourself >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:showDialog&w=examples)


### api.showFileDialog
```javascript
ret = await api.showFileDialog(config)
```

Shows a file dialog to select files or directories.

The function will return a promise from which you can get the file path string.

Depending on the plugin engine implementation, ImJoy will try to select the a file manager specified by the plugin engine via the `api.config.file_manager`.

The file handling is different for the ImJoy app and the plugin engine. We recommend
reading the dedicated section in the [user manual](development?id=loading-saving-data) to understand the difference.

Please note that the file-path for a JavaScript plugin is returned as an url, while
for a Python plugin it will be the absolute file-path. The url format is required for
a JavaScript plugin to be able to open a file. You can change this behaviour with the
`uri_type` option (see below). For instance, you can obtain the absolute path also
for a JavaScript plugin. However, you can not use this path to open the file in
JavaScript, but you can pass it to another Python plugin for processing.

**Arguments**

* **config**. Object (JavaScript) or dictionary (Python). Options for showing the file dialog.
It contains the following fields:

  - **type**: String. Supported modes of file dialog:
    - `file` (default): select one or multiple files;
    - `directory`:  select one or multiple directories. For Python plugins, if you don't specify the type, both file or directory can be selected.
  - **title**: String. Title of the dialog.
  - **root**: String. Initial path for the dialog to show. Note: for Python plugins on Windows,
   you may want to define the path string as raw string using `r"xxxxxx"` syntax,
   we have encountered unrecognised path issue with normal strings.
  - **mode**: String. Modes for file selection. By default, the user can select a single or multiple file (with the `shift` key pressed)
    - `single`: only a single file or directory can be selected.
    - `multiple`: multiple files or directories are selected and are returned in an array or list.
    - `single|multiple` (default): both single and multiple selection are allowed.
  - **file_manager**: String. Specify the file manager via url, in a `native-python` plugin for example, you can get the file manager URL via `api.config.file_manager`.

(Please also consult [this section](api?id=input-arguments) for how arguments can be set.)

**Returns**
* **selected**: An array of object (JavaScript) or dictionary (Python). It can contain 0 to many selected file/directories. If the returned array is empty, it means the user did not select any file/directory. The file items in the array typically contains (depending on different file manager implementation):
  - **path**: String. File path.
  - **url**: String. URL to the file
  - other fields.

**Examples**

Example will show either the specified file-name or an message that the
user canceled or that the plugin engine was not running.

```javascript

const selected = await api.showFileDialog()
if(selected.length>0){
  await api.alert("Selected file " + selected[0].url)
}
else{
  await api.alert("User cancelled file selection.")
}


```
<!--**[TODO]: update this example to new api**-->
<!-- [Try yourself >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:showFileDialog&w=examples) -->


### api.showMessage
```javascript
api.showMessage(message,duration)
```
Updates the status text on the status bar of ImJoy and displays the message with a snackbar.

If duration is not specified, snackbar will be shown for 10s.

**Arguments**

* **message**: String. Message to be displayed.
* **duration** (optional): Integer. Duration in seconds for message to be shown.

**Examples**

```javascript
api.showMessage('Processing...', 5)
```


### api.showProgress
```javascript
api.showProgress(progress)
```
Updates the progress bar on the Imjoy GUI.

**Arguments**

* **progress**: Float or Integer. Progress in percentage. Allowed range from 0 to 100 for integer, or 0 to 1 for float.

**Examples**
```javascript
api.showProgress(85)
```
[Try yourself >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:showProgress&w=examples)

### api.showSnackbar
```javascript
api.showSnackbar(message, duration)
```
Shows a popup message with a snackbar, and disappear in a specific amount of time.

**Arguments**

* **message**: String. Message to be displayed.
* **duration**: Integer. Duration in seconds message will be shown.

**Examples**

```javascript
api.showSnackbar('processing...', 5)
```

[Try yourself >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:showSnackbar&w=examples)

### api.showStatus
```javascript
api.showStatus(status)
```
Updates the status text on the Imjoy GUI.

**Arguments**

* **status**: String. Message to be displayed.

**Examples**

```javascript
await api.showStatus('processing...')
```

[Try yourself >>](https://imjoy.io/#/app?plugin=imjoy-team/imjoy-demo-plugins:showStatus&w=examples)


### api.TAG constant
The current tag chosen by the user during installation.


### api.unregisterService
```javascript
await api.unregisterService(config)
```

Unregister a plugin service.

**Arguments**

* **config**: Object. It must contain the `id` of the plugin service.

**Examples**

```javascript

const sid = await api.registerService({type: 'my-service', my_data: 123})

await api.unregisterService({id: sid})
```




### api.utils.*
```javascript
await api.utils.UTILITY_NAME()
```

Call utility function.

Currently supported functions for **all plugins** are:
 * `api.utils.$forceUpdate()`: refreshes the GUI manually.
 * `api.utils.openUrl(url)`: opens an `url` in a new browser tab.
 * `api.utils.sleep(duration)`: sleeps for the indicated `duration` in seconds. Note for Python plugins, use `time.sleep` instead.)
 * `api.utils.showOpenFilePicker`: only works in Chrome 86+, popup a dialog for opening a file with the [File System Access](https://web.dev/file-system-access/) API
 * `api.utils.showSaveFilePicker`: only works in Chrome 86+, popup a dialog for saving a file with the [File System Access](https://web.dev/file-system-access/) API
 * `api.utils.showDirectoryPicker`: only works in Chrome 86+, popup a dialog for choose a directory with the [File System Access](https://web.dev/file-system-access/) API

### api.config
The configuration information consists of:
 * `workspace`: the current workspace.
 * `engine`: URL of the current plugin engine. **Only available to native-python plugins**
 * `file_manager`: URL of the file manager registered by the current plugin engine. **Only available to native-python plugins**

### api.config
The configuration information consists of:
 * `workspace`: the current workspace.
 * `engine`: URL of the current plugin engine. **Only available to native-python plugins**
 * `file_manager`: URL of the file manager registered by the current plugin engine. **Only available to native-python plugins**

### api.WORKSPACE constant
**Deprecated!** Use `api.config.workspace` instead

Name of the current workspace.


### api.ENGINE_URL constant
**Deprecated!** Use `api.config.engine` instead

**Only available to native-python plugins**

URL of the current plugin engine.

### api.FILE_MANAGER_URL constant
**Deprecated!** Use `api.config.file_manager` instead

**Only available to native-python plugins**

URL of the file manager registered by the current plugin engine.

## Internal plugins

Besides the default ImJoy api, we provide a set of internally supported plugins which can be used directly. These plugins will be loaded only if the plugin is requested by another plugin via `api.getPlugin(...)`.

Here is a list of these internal plugins along with their api functions:

### BrowserFS

To use the `BrowserFS` plugin, you need to first call:

`const bfs = await api.getPlugin('BrowserFS')` in Javascript, or `bfs = await api.getPlugin('BrowserFS')` in Python.

Then, you can use [Node JS filesystem API](https://nodejs.org/api/fs.html) to access the in-browser filesystem (e.g.: `bfs.readFile('/tmp/temp.txt', 'utf-8')`). For more details about the underlying implemetation, see [BrowserFS](https://github.com/jvilk/BrowserFS), the default file system in ImJoy supports the following nodes:

 * `/tmp`: `InMemory`, data is stored in browser memory, cleared when ImJoy is closed.

 * `/home`: `IndexedDB`, data is stored in the browser IndexedDB database, can be used as persistent storage.

**Examples**

<!-- tabs:start -->
#### ** JavaScript **
```javascript
async function test_browser_fs(){
  const bfs_plugin = await api.getPlugin('BrowserFS')
  const bfs = bfs_plugin.fs

  bfs.writeFile('/tmp/temp.txt', 'hello world', function(err, data){
      if (err) {
          console.log(err);
          return
      }
      console.log("Successfully Written to File.");
      bfs.readFile('/tmp/temp.txt', 'utf8', function (err, data) {
          if (err) {
              console.log(err);
              return
          }
          console.log('Read from file', data)
      });
  });
}
```
#### ** Python **
```python

async def test_browser_fs():
  bfs_plugin = await api.getPlugin('BrowserFS')
  bfs = bfs_plugin.fs

  def read(err, data=None):
      if err:
          print(err)
          return

      def cb(err, data=None):
          if err:
              print(err)
              return
          api.log(data)
      bfs.readFile('/tmp/temp.txt', 'utf8', cb)

  bfs.writeFile('/tmp/temp.txt', 'hello world', read)

```
<!-- tabs:end -->


Reading large files chunk-by-chunk in JavaScript:
```javascript
function generate_random_data(size){
    var chars = 'abcdefghijklmnopqrstuvwxyz'.split('');
    var len = chars.length;
    var random_data = [];

    while (size--) {
        random_data.push(chars[Math.random()*len | 0]);
    }

    return random_data.join('');
}

function fsRead(fd, buffer, offset, chunkSize, bytesRead) {
  return new Promise((resolve, reject) => {
    fs.read(fd, buffer, offset, chunkSize, bytesRead, (err, bytesRead,
      read_buffer) => {
      if (err) {
        console.log('err : ' + err);
        reject(err)
        return
      }
      const bytes = read_buffer.slice(0, bytesRead)
      resolve(bytes)
    });
  })
}

bfs.writeFile('/tmp/test.txt', generate_random_data(100000), function(err){
if (err){
    console.error(err);
}
bfs.open('/tmp/test.txt', 'r', function(err, fd) {
    bfs.fstat(fd, async function(err, stats) {
      if(err){
          console.error(err)
          return
      }
      var bufferSize = stats.size,
          chunkSize = 512,
          buffer = new Uint8Array(new ArrayBuffer(chunkSize)),
          bytesRead = 0;
      try{
        while (bytesRead < bufferSize) {
            if ((bytesRead + chunkSize) > bufferSize) {
                chunkSize = (bufferSize - bytesRead);
            }
            const bytes = await fsRead(fd, buffer, 0, chunkSize, bytesRead)
            console.log(bytes)
            bytesRead += chunkSize;
        }
        console.log("Finished reading.")
      }
      catch(e){
        console.error(e)
      }
      finally{
        bfs.close(fd);
      }

    });
  });
})
```

## Sanitized HTML and CSS

For security reasons, we used [DOMPurify](https://github.com/cure53/DOMPurify) to sanitize the HTML and CSS provided to shown in the ImJoy main interface. For example, all the markdown, `ui` string for `<config>` block and `api.register`, and the content shown in `api.alert`, `api.confirm` and `api.prompt`.

Also notice that the content shown inside a `window` plugin do not have these restrictions.



## API changelog

#### api_version: 0.1.8
 * deprecate `api.register` and `api.unregister`, use `api.registerService` and `api.unregisterService` instead.
 * add `api.getWindow` to obtain an existing window instance.
 * `api.showFileDialog`:
   - if the file manager provide `showFileDialog` function, then ImJoy will use it.
   - remove the key `uri_type` from input arguments, remove `engine` from its result.
   - it will always return an array of items.
 * support `base_frame` (in `<config>` block) option for `window` plugins to load from a custom html url.
 * support creating window or dialog from an external web page, for example:  `api.createWindow({type: "external", src: "https://kitware.github.io/itk-vtk-viewer/app", passive: true})`.
 * fix `api.alert` display when passing an object
 * support passing plugin url or source code to `api.getPlugin`, `api.createWindow`, `api.showDialog`, which enables dynamic plugin loading
 * support passing `tag` and `namespace` to `api.getPlugin` and `api.createWindow` when constructing plugin or window from source code
 * The plugin api object (returned from api.getPlugin, api.getWindow, api.createWindow, api.showDialog) will also include a config object which contains id, name, namespace, workspace, tag (and window_id for window plugin instance).
 * support `api.installPlugin` and `api.uninstallPlugin`.
 * support setting `passive` key in `<config>`.
 * remove `_rpcEncode` and `_rpcDecode` (use `api.registerCodec` instead)
 * more complete web-python implementation: using imjoy-rpc library from pip in pyodide; switch to web-worker mode
 * deprecate plugin type `web-python-window`
 * support `base_worker` for specifying a web-worker script for `web-python` plugins.
 * add `api.config` and deprecate constants including `api.WORKSPACE`, `api.TAG`, `api.ENGINE_URL`, `api.FILE_MANAGER_URL`.
 * make the api more consistent for `api.createWindow` and `api.getPlugin`. Allowing passing a string to `api.createWindow` and accepting a `config` object for `api.getPlugin`
 * added `api.loadPlugin` which is currently directly mapped to `api.getPlugin` but will provide better semantic meaning. Overall, we have `api.getPlugin` and `api.getWindow` for obtain an existing plugin or window, and we have `api.createWindow` and `api.loadPlugin` for creating a new instance of window or plugin.

 * Add supports for File System Access API (only for Chrome): `api.utils.showOpenFilePicker`, `api.utils.showSaveFilePicker`, `api.utils.showDirectoryPicker`.
 * Change the prefix for registerService (from `_` to `#`)
#### api_version: 0.1.7
 * `api.fs` has been deprecated, the browser file system is moved to a separate plugin `BrowserFS`, to use the file system, you can do `const bfs_plugin = await api.getPlugin('BrowserFS'); const bfs = bfs_plugin.fs;`, now `fs` will be equivalent to `api.fs`. Notice: the data saved with `api.fs` will not be accessible with the new API, to get access the old data, please change `api_version` in the plugin config to `0.1.6`.
 * added `_rpcEncode` and `_rpcDecode` to support custom encoding and decoding
 * remove `api.utils.assert` because the async version is misleading
 
#### api_version: 0.1.6
 * added new api functions `api.getFileManager`, `api.getEngine`, `api.getEngineFactory`
 * `api.getFileUrl` is deprecated, call `file_manager = await api.getFilemanager('http://...');` and then access them with `file_manager.getFileUrl`.
 * `api.requestUploadUrl` is deprecated, call `file_manager = await api.getFilemanager('http://...');` and then access them with `file_manager.requestUploadUrl`.
 * since `api_version > 0.1.6` you must specify the `lang` field for the `<config>` block, e.g.: `<config lang="json">`.
