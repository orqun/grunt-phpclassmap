# grunt-phpclassmap

> Generate PHP classmaps

## Getting Started
This plugin requires Grunt `~0.4.4`

If you haven't used [Grunt](http://gruntjs.com/) before, be sure to check out the [Getting Started](http://gruntjs.com/getting-started) guide, as it explains how to create a [Gruntfile](http://gruntjs.com/sample-gruntfile) as well as install and use Grunt plugins. Once you're familiar with that process, you may install this plugin with this command:

```shell
npm install grunt-phpclassmap --save-dev
```

Once the plugin has been installed, it may be enabled inside your Gruntfile with this line of JavaScript:

```js
grunt.loadNpmTasks('grunt-phpclassmap');
```

## The "phpclassmap" task

### Overview
In your project's Gruntfile, add a section named `phpclassmap` to the data object passed into `grunt.initConfig()`.

```js
grunt.initConfig({
  phpclassmap: {
    options: {
      // Task-specific options go here.
    },
    your_target: {
      // Target-specific file lists and/or options go here.
    },
  },
});
```

### Options

#### options.basedir
Type: `String`
Default value: .

Base directory from which to create the relative paths to class files, defaults to `process.cwd`.

#### options.phpbin
Type: `String`
Default value: 'php'

Path to the php executable

#### options.quote_path
Type: `Boolean`
Default value: true

Wether or not the classes in the classmap should be quoted. There might be some situations where you don't want to do that, for instance in the [filter example](#filter-example).

#### options.dest
Type: `String`
Default value: none

Where to write the classmap file to.

#### options.map
Type: `Function`
Default value: none

Function which maps the entry inside the destination classmap file. The function recieves a item object which in turn has the following properties:

* absolute_path (absolute path to the object)
* relative_path (relative path to the object, which is the absolute_path without the basedir option)
* name (name of the object)
* type (class, interface, trait)

**IMPORTANT** The function should return the resulting item

#### options.filter
Type: `Function`
Default value: none

Function which can be used to filter out unwanted items for the classmap, see the [filter example](#filter-example)

#### options.sort
Type: `Function`
Default value: none

Function which allows you to sort the found classes by providing a function which will be used by the Array.prototype.sort function, see the [sort example](#sort-example)

### Usage Examples

#### Basic usage

The following is a basic example usage of the grunt-phpclassmap. This will scan all files under the directory `src` and will try to find all php classes.

```js
grunt.initConfig({
    phpclassmap: {
        options: {
            dest: './classmap.php'
        },
        files  : {
            src   : [ 'src/**/*.php' ],
            expand: true
        }
    },
});
```

The resulting `classmap.php` will look something like:

```php
<?php
/**
 * Generated by grunt-phpclassmap on {{date}}
 */
return array(
	"Foo" => "/src/foo.class.php",
	"Bar" => "/src/class-bar.php"
);

/* EOF */
```

#### Map items to use a defined constant

In this example, we use override the format function in order to use a defined constant inside the resulting classmap

```js
grunt.initConfig({
    phpclassmap: {
        options: {
			quote_path: false,
            dest: './classmap.php',
            map: function(item) {
                item.relative_path = 'DEFINED_BASE_PATH_CONSTANT . "' + item.relative_path + '"';
                return item;
            }
        },
        files  : {
            src   : [ 'classes/**/*.php' ],
            expand: true
        }
    },
});
```

#### Custom handlebars template

grunt-phpclassmap uses handlebars to render the classmap file, you can specify a custom handlebars template, like so:

```js
grunt.initConfig({
    phpclassmap: {
        options: {
            dest: './classmap.php',
            template: 'my-classmap-template.tpl'
        },
        files  : {
            src   : [ 'classes/**/*.php' ],
            expand: true
        }
    },
});
```

currenty the compiled template recieves data in the form:

```js
{
	quote_path: Boolean,
    items: Array,
    date: String
}
```

#### Custom the render functionality

besides changing the template it's also possible to define you own render function, like so:

```js
grunt.initConfig({
    phpclassmap: {
        options: {
            dest: './classmap.php',
            render: function(objects, cb) {
                var content = '<?php\n' +
                    'return array(' +
                    '%items%' +
                    ');';
                // Build up the item array
                var items = [];
                for(var i=0; i<objects.length;i++) {
                    items.push('"' + objects[i].name + '" => "' + objects[i].absolute_path + '"');
                }
                cb(content.replace('%items%', items.join(',')));
            }
        },
        files  : {
            src   : [ 'classes/**/*.php' ],
            expand: true
        }
    },
});
```

**IMPORTANT**: When you override the render method be sure to call to callback which is provided to the function, as this is the function which writes to results to the classmap.

#### Custom map and render functionality

In this example you can see how you can mix the map and render function to do some custom stuff.
Note that the map allows you to add additional data, which is written in de render function. This could have also been
done by providing a custom template.

```js
grunt.initConfig({
    phpclassmap: {
        options: {
            map: function (item) {
                item.rewritten_path = your_rewrite_function(item);
                return item;
            },
            render: function(objects, cb) {
                    var content = '<?php\n' +
                                'return array(' +
                                '%items%' +
                                ');';
                    // Build up the item array
                    var items = [];
                    for(var i=0; i<objects.length;i++) {
                        items.push('"' + objects[i].name + '" => "' + objects[i].rewritten_path) + '"';
                    }
            cb(content.replace('%items%', items.join(',')));
        }
```

#### Filter example

The filter option can be used to remove unwanted items, the function should return a `Boolean`. It should return true to include the item in the classmap; otherwise, false.

```js
grunt.initConfig({
    phpclassmap: {
        options: {
			dest: 'classmap.php',
            filter: function (item) {
                return item.name != 'My_Unwanted_Class';
            }
        }
	}
});
```

#### Sort example

The sort option can be used to sort the objects before they are written to the classmap file.

```js
grunt.initConfig({
    phpclassmap: {
		options: {
			dest: './classmap.php',
			sort: function(a, b) {
				var aa = a.name.toUpperCase();
				var bb = b.name.toUpperCase();
				return (aa < bb) ? -1 : (aa > bb) ? 1 : 0;
			}
		},
		files  : {
            src   : [ 'src/**/*.php' ],
            expand: true
        }
	}
}
```

#### Combined with grunt-contrib-watch

Example of a nice way to automaticly generate classmaps during development.

```js
grunt.initConfig({
    phpclassmap: {
        options: {
			dest: 'classmap.php',
            filter: function (item) {
                return item.name != 'My_Unwanted_Class';
            }
        },
		files  : {
            src   : [ 'classes/**/*.php' ],
            expand: true
        }
	},
	watch: {
		phpclassmap: {
			files: ['classes/**/*.php' ],
			tasks: [ 'phpclassmap' ]
		}
	}
});
```


## Release History


### Version 0.0.1

Initial release

### Version 0.0.2

Fix: added handlebars as a required dependency

### Version 0.0.3

Fix: added dateformat as a required dependency

### Version 0.0.4

Fix: remove some debug statements
Fix: typo in the default template comments

### Version 0.0.5

Fix: correct stupid mistake of writing the classmap inside a loop
Update: add the quote_path option
Update: the default template to use the quote_path

### Version 0.0.6

Update: add the map option for changing the item
Change: change the existing filter option functionality to filter found entries

### Version 0.0.7

Update: add the sort option
Update: provide better feedback of the found classmap results

### Version 0.0.8

Update: support for bigger projects

### Version 0.0.9

Fix: PHP notice during class map generation

### Version 0.0.10

Fix: Temporary path fix

### Version: 0.0.11

Fix: add underscore and underscore.string as dependencies