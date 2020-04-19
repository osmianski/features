# `f5` Using Cache

It is always faster to load objects from cache than to construct them and, actually, most objects are cached and then retrieved from cache. 

However, there is a couple of things to take care of in order to make object cache-friendly. 

This article describes how to store and retrieve objects from cache, how cache works and how to write cache-friendly classes.

{{ toc }}

## Using Cache

**Cache** is just a key-value store: you can put a value into cache by telling that it can be accessed by a specific key and later you can retrieve the same value by that key. Key is a string and value can be any type.

To use the cache, add a dependency property to the caller class returning `$osm_app->cache` (see the next section for a full code sample.

Values stored in cache are also called **cache items**. Most often, cache items are objects of `CacheItem` or derived class, and most often, they are stored and retrieved using `remember()` method:

    /**
     * @property Cache $cache @required
     * @property Bar $bar @required
     */
    class Foo extends Object_
    {
        protected function default($property) {
            global $osm_app; /* @var App $osm_app */
    
            switch ($property) {
                case 'cache': return $osm_app->cache;
                case 'bar': return $this->cache->remember('bar', function($data) {
                    return Bar::new($data);
                });
            }
            
            return parent::default($property);
        }
    }
    
`remember()` method checks if the object is in the cache and if so, loads it from cache, otherwise it execute the callback function to create the object and stores created object in cache.     
 
**Note**. First parameter of `remember()` method is globally unique object's cache key. Don't accidentally reuse the same cache key for another object or you may not get the object you expect.  

**Note**. Technically you can store any object, but `CacheItem` class is highly recommended as it provides means to automatically update the cached copy if the object itself or its child objects change.

Most common examples of cache items are registry classes and `Settings` class which merge and process configuration files from `config/` directories and store it in cache as a ready to use object.

Sometimes functionality of `CachehItem` class is not needed. An example of not using `CacheItem` is the layer system which collects and loads layer files from `layers` directories of all modules and themes and stores arrays defined in those files in cache in order to avoid collecting layer files on every request.

## Writing Cache-Friendly Classes 

In order to make object serializable (and hence cache-friendly): 

1. Decide which properties should be serialized or, in other words, which properties are `@part` of the object.
2. Correctly assign object's `parent` properties, so that any object in a tree can notify the root cache item that something in the object tree has been modified.

Mark property using `@part` attribute if it is:

* part of the object's data, like person's age or name
* calculated slow  

Don't mark property using `@part` attribute if it is:

* calculated fast
* dependency property
* contains temporary data

### Mark `@part` Properties

When cache item is put into the cache, its properties marked with `@part` attribute are serialized into a string. 

When cache item is retrieved, the stored string is unserialized back into an object. 

Serialization/unserialization is recursive. It means that if a `@part` property of a cache item contains an `Object_` instance or an array of `Object_` instances, then all such objects are serialized and according to the `@part` property attribute.

In the following example, `name` and `children` properties are serialized (of which `children` recursively) and `dependency` property is not serialized.  

    /**
     * @property string $name @required @part
     * @property Joe[] $children @part
     * @property Dependency $dependency @required
     */
    class Bar extends CacheItem {
    }       

`children` is array of objects which also contain serializable properties:

    /**
     * @property int $age @required @part
     */
    class Joe extends Object_ {
    }       

To make sure that you serialize the right properties, you can try to serialize the object into a string and then unserialize it back into an object. If the new object works as expected, good, if it misses some data of the original object (or on the contrary, saves too much data), review `@part` attributes:

    $bar = Bar::new(['name' => 'bar']);
    
    $bar2 = unserialize(serialize($bar));
    // $bar2->name === 'bar'  

### Call `modified()` Method

Objects that are loaded from cache and modified in memory should be put into the cache anew. To re-save the object, inform the cache that it requires saving by calling its `modified()` method:

    $bar->name = 'bar';
    $bar->modified();
    
In case `@part` property is calculated in the `default()` method, `modified()` method is called automatically, so next time the object is loaded from cache, it will already contain calculated value.

`modified()` method of a child object recursively calls `modified()` method of its parent right up the root of object tree - the cache item. It means that if a property of some child object is modified or calculated in the `default()` method, the parent cache item will be re-saved.

### Assign `parent` Property

It is important, though, that `parent` property of child object is property assigned. The cache detects incorrectly assigned object parents and throws an exception. 

