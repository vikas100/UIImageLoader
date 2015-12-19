# UIImageDiskCache

UIImageDiskCache is an alternative to a subset of caching functionality from SDWebImage.

It can use server cache control policies to re-download images when expired.

Or you can completely ignore the cache control policies from a server and manually clean-up images.

It's very small at roughly 500+ lines of code and only a header/implementation file.

Server cache control logic is implemented manually instead of a NSURLCache. There's a noticeable difference in
performance without NSURLCache.

Everything is asynchronous and uses modern objective-c with libdispatch and NSURLSession.

## No Flickering or Noticeable Delayed Loads

Images that are cached and available on disk load into UIImageView or UIButton almost immediatly.

This is most noticeable on table view cells. The slight delay that you may sometimes see before a cached image is loaded and displayed is entirely gone.

## Server Cache Policies

It works with servers that support ETag and Cache-Control headers.

If the server responds with only ETag you can optionally cache the image for a default amount of time. Or don't cache it at all and send requests each time.

If a response is 403 it uses the cached image available on disk.

## UIImageDiskCache Object

There's a default configured cache which you're free to configure how you like.

````
UIImageDiskCache * cache = [UIImageDiskCache defaultDiskCache];
//set cache properties here.
````

Or you can setup your own and configure it:

````
UIImageDiskCache * cache = [[UIImageDiskCache alloc] init];
//set cache properties here.
````

By default the cache will use server cache control policies. You can disable server cache control:

````
myCache.useServerCachePolicy = FALSE;
````

For responses that return an ETag header but no Cache-Control header you can set a default amount of time to cache those images for:

````
myCache.etagOnlyCacheControl = 604800; //1 week;
myCache.etagOnlyCacheControl = 0;      //don't cache. Always send requests even if responses are 403.
````

## UIImageView & UIButton

If you want to use the default cache, use one of these methods:

````
UIImageView:
- (NSURLSessionDataTask *) setImageWithURL:(NSURL *) url completion:(UIImageDiskCacheCompletion) completion;
- (NSURLSessionDataTask *) setImageWithRequest:(NSURLRequest *) request completion:(UIImageDiskCacheCompletion) completion;

UIButton:
- (NSURLSessionDataTask *) setImageForControlState:(UIControlState) controlState withURL:(NSURL *) url completion:(UIImageDiskCacheCompletion) completion;
- (NSURLSessionDataTask *) setImageForControlState:(UIControlState) controlState withRequest:(NSURLRequest *) request completion:(UIImageDiskCacheCompletion) completion;
````

If you use a custom configured cache use these methods:

````
UIImageView:
- (NSURLSessionDataTask *) setImageWithURL:(NSURL *) url customCache:(UIImageDiskCache *) customCache completion:(UIImageDiskCacheCompletion) completion;
- (NSURLSessionDataTask *) setImageWithRequest:(NSURLRequest *) request customCache:(UIImageDiskCache *) customCache - completion:(UIImageDiskCacheCompletion) completion;

UIButton:
- (NSURLSessionDataTask *) setImageForControlState:(UIControlState) controlState withURL:(NSURL *) url customCache:(UIImageDiskCache *) customCache completion:(UIImageDiskCacheCompletion) completion;
- (NSURLSessionDataTask *) setImageForControlState:(UIControlState) controlState withRequest:(NSURLRequest *) request customCache:(UIImageDiskCache *) customCache completion:(UIImageDiskCacheCompletion) completion;
````

## Memory Cache

_UIImage System Cache Overview_

````
UIImage.imageNamed: returns cached images. Images are purged only when memory conditions start to get volatile.
````

````
UIImageView.imageWithContentsOfFile: always returns a new copy of the image off disk. No memory cache.
````

In UIImageDiskCache, all images are loaded off disk with UIImage.imageWithContentsOfFileURL:. In most cases this is preferred behavior. For most iOS apps this is probably better. As you load / unload views the images you use get deallocated immediately. If you're only displaying unique images, or even using an image only a couple times, then this is prefferred.

If however you have an image that is going to be used a lot, you can cache the image in memory with:

````
[myCache.memoryCache cacheImage:myImage forURL:url];
[myCache.memoryCache cacheImage:myImage forRequest:request];
````

After an image is cached in memory it will be used instead of a new image from disk cache.

You can set the max cache size in bytes with:

````
myCache.memoryCache.maxBytes = 26214400; //25MB
````

You can purge all from memory with:

````
[myCache.memoryCache purge];
````

## Other Useful Features

### SSL

If you need to support self signed certificates you can use (false by default):

````
myCache.trustAnySSLCertificate = TRUE;
````

### Auth Basic Password Protected Directories/Images

You can set default user/pass that gets sent in every request with:

````
[myCache setAuthUsername:@"username" password:@"password"];
````
