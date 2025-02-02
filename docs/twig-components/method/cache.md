# cache

`cache` is a Twig tag to tell Twig to cache a template fragment.


### New in version 3.2

The cache tag was added in Twig 3.2.



```twig
{% cache "cache key" %}
    Cached forever (depending on the cache implementation)
{% endcache %}
```

If you want to expire the cache after a certain amount of time, specify an expiration in seconds via the ttl() modifier:

```twig
{% cache "cache key" ttl(300) %}
   Cached for 300 seconds
{% endcache %}
```

The cache key can be any string that does not use the following reserved characters {}()/\@:; a good practice is to
embed some useful information in the key that allows the cache to automatically expire when it must be refreshed:

<ul>
<li>Give each cache a unique name and namespace it like your templates;</li> 
<li>Embed an integer that you increment whenever thetemplate code changes (to automatically invalidate all current caches);</li> 
<li>Embed a unique key that is updated whenever thevariables used in the template code changes.</li> 
</ul>

For instance, I would use 
```twig
{% cache "blog_post;v1;" ~ post.id ~ ";" ~post.updated_at %}
```
to cache a blog content template fragment where blog_post describes the template fragment, v1
represents the first version of the template code, post.id represent the id of the blog post, and post.updated_at
returns a timestamp that represents the time where the blog post was last modified.

Using such a strategy for naming cache keys allows to avoid using a ttl. It's like using a "validation" strategy instead
of an "expiration" strategy as we do for HTTP caches.

If your cache implementation supports tags, you can also tag your cache items:

```twig
{% cache "cache key" tags('blog') %} 
    Some code 
{% endcache %}
```

```twig
{% cache "cache key" tags(['cms', 'blog']) %} 
    Some code 
{% endcache %}
```

 The cache tag creates a new "scope" for
variables, meaning that the changes are local to the template fragment:

```twig
{% set count = 1 %}

{% cache "cache key" tags('blog') %} 
    {# Won't affect the value of count outside of the cache tag #} 
    {% set count = 2 %}
    Some code 
{% endcache %}

{# Displays 1 #} 
{{ count }} 
```


### Note

The cache tag is part of the CacheExtension which is not installed by default. Install it first:

```twig
$ composer require twig/cache-extra</pre>
```

On Symfony projects, you can automatically enable it by installing the
twig/extra-bundle:

```twig
$ composer require twig/extra-bundle</pre> 
```

Or add the extension explicitly on the Twig environment:

```twig
use Twig\Extra\Cache\CacheExtension;


$twig = new \Twig\Environment(...); 
$twig->addExtension(new CacheExtension());
```

If you are not using Symfony, you must also register the extension runtime:

```twig
use Symfony\Component\Cache\Adapter\FilesystemAdapter; 
use Symfony\Component\Cache\Adapter\TagAwareAdapter; 
use Twig\Extra\Cache\CacheRuntime; 
use Twig\RuntimeLoader\RuntimeLoaderInterface
```

```twig
$twig->addRuntimeLoader(new class implements RuntimeLoaderInterface{ 
    public function load($class) {
        if (CacheRuntime::class === $class) {
            return new CacheRuntime(new TagAwareAdapter(new FilesystemAdapter())); 
        } 
    } 
});
```


Source: [Twig](https://twig.symfony.com/cache)