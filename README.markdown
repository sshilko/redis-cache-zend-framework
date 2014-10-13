Redis cache backend for Zend Framework by Carl Oscar Aaro
=============

Modified to support Redis MGET command (fetching multiple keys in one request)
=============

Requires the phpredis extension for PHP to enable PHP to communicate with the [Redis](http://redis.io/) key-value store.
Start with installing the phpredis PHP extension available at https://github.com/nicolasff/phpredis

Set up the Redis cache backend by invoking the following code somewhere in your project.

ZF1 application ini setup

```
;Redis Extension-Based Cache -->
; @see https://github.com/nicolasff/phpredis
; @see https://github.com/kalaspuff/redis-cache-zend-framework
; @see https://github.com/sshilko/redis-cache-zend-framework
; Put files into your Zend library folder i.e. root/library/Extended....

resources.cachemanager.redis2.frontend.name = "Core"
resources.cachemanager.redis2.frontend.options.lifetime = 86400
resources.cachemanager.redis2.frontend.options.automatic_serialization = true
resources.cachemanager.redis2.frontend.options.write_control = false
resources.cachemanager.redis2.frontend.options.automatic_cleaning_factor = 0
resources.cachemanager.redis2.frontend.options.ignore_user_abort = true
resources.cachemanager.redis2.backend.name  = 'Extended_Cache_Backend_Redis'
resources.cachemanager.redis2.backend.options.servers.0.host = 'redis.somehost.com'
resources.cachemanager.redis2.backend.options.servers.0.port = 6379
resources.cachemanager.redis2.backend.options.servers.0.persistent = true

; USE BACKEND PREFIX INSTEAD OF CORE PREFIX TO ALLOW DIRECT BACKEND USAGE
resources.cachemanager.redis2.backend.options.key_prefix = "ENVIRONMENT_PROBABLY"
;resources.cachemanager.redis2.frontend.options.cache_id_prefix = 'ENVIRONMENT_PROBABLY'

resources.cachemanager.redis2.backend.customBackendNaming = true
resources.cachemanager.redis2.frontendBackendAutoload = true
;Redis Extension-Based Cache <--

```

Somewhere in your bootstrap

```
    protected function _initCaches() {
        $this->bootstrap('cachemanager');
        Zend_Registry::set('cachemanager', $this->getPluginResource('cachemanager')->getCacheManager());
    }
```

Somewhere where u need cache

<pre>
Zend_Registry::get('cachemanager')->getCache('redis2');
$cacheBackend = $cache->getBackend();

//use backend to fetch multiple values
$cachedRows = $cacheBackend->load(array('1', '2', '3'));

//...

//use frontend (core) to save single values as usual
$cache->save('1', 'newval');
</pre>

Original initialization

<pre>

$redisCache = Zend_Cache::factory(
    new Zend_Cache_Core(array(
        'lifetime' => 3600,
        'automatic_serialization' => true,
    )),
    new Extended_Cache_Backend_Redis(array(
        'servers' => array(
            array(
                'host' => '127.0.0.1',
                'port' => 6379,
                'dbindex' => 1,
            ),
        ),
    ))
);
</pre>

Writing and reading from the cache works the same way as all other Zend Framework cache backends.

<pre>
$cacheKey = 'my_key';
$data = 'e48e13207341b6bffb7fb1622282247b';

/* Save data to cache */
$redisCache->save($data, $cacheKey, array('tag1', 'tag2'));

/* Load data from cache */
$data = $redisCache->load($cacheKey);

/* Clear all keys with tag 'tag1' */
$redisCache->clean(Zend_Cache::CLEANING_MODE_MATCHING_ANY_TAG, array('tag1'));

/* Clear all cache (flush cache) */
$redisCache->clean(Zend_Cache::CLEANING_MODE_ALL);
</pre>
