# Ideas

[![Join the chat at https://gitter.im/Klortho/config-one](https://badges.gitter.im/Klortho/config-one.svg)](https://gitter.im/Klortho/config-one?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

The design pattern of cascading (overridable) heirarchical data shows up everywhere you look, but I've never encountered an implementation that gets it right. (That's not to say there isn't one!)

## Data model

Rough sketch:

* All config objects are immutable forever.
* Every source (file, env. vars, command-line, etc.) results in a config object. The config object does not store any information about what are defaults and what are overrides -- i.e. it doesn't know anything about how they are chained together.
* All access to the config data is done through a view, which is analogous to the resolver we just implemented. From the outside, a view looks just like a static tree of data, with everything resolved. But inside, a view is implemented as an ordered list of config objects, with the first one being the default, and subsequent ones being overrides.
* The "deferred" functions are are always maintained in the config object as-is -- they are never resolved and thrown away. Remember, the config object itself, that holds the deferred, is immutable.
* Deferreds are all lazily evaluated by the *view*, and memoized (cached). The view passes itself in as the context (just the same way that the resolver does now). So, that's why we can say that the config data is immutable, even though there might be different config objects layered on at different times: for any given *view*, the tree is static and immutable. If you layer on a new config, or even insert a new config anywhere you want into the list, you've now created a new view, which is a separate static, immutable tree.
* By lazy, I mean really lazy. Deferreds are not resolved until the data is accessed at runtime. This is different from the way they are now, after the config data is read in.

## Implementation

Some notes:

* ES2015's proxy objects would be ideal for this, but it's possible to implement quite well with ES5 getters. With getters, there are some limitations, because inside the lambda functions, views are not really the same thing as the orig. config objects, so inherited methods are not accessible. Maybe this could be improved.

* For Python, see:
    * [ProxyTypes](https://pypi.python.org/pypi/ProxyTypes/0.9)
    * [A Guide to Python's Magic Methods](http://www.rafekettler.com/magicmethods.html)

## Features / possibilities

Some of the things that this would facilitate:

* Create a new configuration at any time, using any other one to provide the "defaults".
* Not only that -- but any internal logic based on the deferred functions would be preserved! The final value depends on the view -- the deferreds themselves never get discarded.
* Continuing along those lines -- imagine a universal config mechanism -- user data like cookies, or HTTP request query-string parameters, could be used to instantiate a new config.
* The deferreds could easily be enhanced with hooks for validation and/or normalization -- I saw that you have a ticket open about that.
* Features like configly's pluggable parsers would be a snap -- they would just have to produce a config object.
* Hierarchical configs, or nested views, would also work seamlessly! In other words, submodules wouldn't have to try to do anything to integrate with the larger environment. They just provide a local view into any config sources they know about, and the app plugs this in wherever it wants.
* A submodule could even just export itself as a view -- that is, it could just mix-in the `get()` method, or define getters that do the same thing.

Should have plugins for lots of file formats, as well as:

* URL query-string params, or POST params
* environment variables
* browser cookies
* ini files
* xml

And, of course, there's no reason that more primitive mechanisms like template strings couldn't be implemented with deferred functions.


## Exemplars

* node-config - see [this implementation](https://github.com/Klortho/node-config/blob/resolver/defer.js); [this pull request](https://github.com/lorenwest/node-config/pull/318)

* xmltools (not released publicly yet) had a cool feature whereby you could instantiate new "toolboxes" from existing ones, while overriding config values, in a cascade, ad-infinitum:

    ```
    var toolbox = require('xmltools');  // encapsulates a set of defaults
    ...
    // Now, create a new toolbox, overriding a setting. From an API point of view,
    // this is indistinguishable from the first:
    var myToolbox = xmltools({haltOnError: false});
    // Ad-infinitum
    var hisToolbox = myToolbox({verbose: true});
    ```

  config-one would make this trivial, but even better: the logic encoded in the deferred functions is preserved.

* [settings-resolver](https://github.com/Klortho/settings-resolver) - implementation of a large part of this for Django settings. There's some good write-up of "motivation" that could be reused.

* [ChainTreeMap.py](https://github.com/Klortho/chain-tree-map/blob/master/ChainTreeMap.py) - an implementation of the "view", in Python. It is an enhancement of [this Chainmap](http://code.activestate.com/recipes/305268/), which is a view of a Python dictionary, but this extends it to work on hierarchical dictionaries.

* [grunt-template-functions](https://github.com/Klortho/grunt-template-functions) - a very early attempt at this, to reimplement Grunt's string templates as first-class JS functions.


## Documentation / illustrations

* Let's use [d3-flextree](http://klortho.github.io/d3-flextree/index.html) to make some illustrations of how it works.
* See also [dtd-diagram](http://klortho.github.io/dtd-diagram/)



