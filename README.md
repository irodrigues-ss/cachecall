# Cachecall

## Description
A cache library for sync and async functions with ttl and expiration time.

## Features
- [Algorithm (FIFO)](#algorithm)
- [TTL](#ttl-time-to-live)
- [Expire Time](#exp)
- [Cache groups](#cache-groups)
- [Examples](#examples)
    - [Cache regular (sync) functions](#cache-regular-sync-functions)
    - [Cache async functions](#cache-async-functions)
    - [Cache with max size](#cache-with-max-size)
    - [Cache with ttl (time to live)](#cache-with-ttl-time-to-live)
    - [Cache with expire time](#cache-with-expire-time)
    - [Cache group](#cache-group)
    - [Clean cache group](#clean-cache-group)
    - [Keep cache ignoring some keys](#keep-cache-ignoring-some-keys)
- [Documentation](#documentation)
    - [Cache decorator](#cache-decorator)
    - [Expire Time class](#expire-time-class)

### Algorithm
The FIFO Algorithm is used to control the input and output of data for cache groups.

### Cache groups
By default each function has its own cache group to keep the cached data separated. Is possible create a shared cache group between different functions or clear cache group using ```clear_cache("group_name")```.

### TTL (Time To Live)
By default the ```cache``` decorator has the ttl parameter to define *time to live* in seconds to expire the cached data.

### Expire Time
With **cachecall.ExpireTime** is possible define a specific time to cached data expire every day. [Example here](#cache-with-expire-time).

### Examples

##### Cache regular (sync) functions

```python
from cachecall import cache

@cache()
def func(x=None):
    print("call")
    return 10
```

##### Cache async functions
```python
from cachecall import cache

@cache()
async def afunc(x=None):
    print("call")
    return 15

await afunc()
```

##### Cache with Max Size
```python
from cachecall import cache

@cache(max_size=2)
def func(x):
    print("call")
    return x

func("a") # Add value "a" in cache
func("b") # Add value "b" in cache
func("c") # Remove value "a" from cache and Add value "c"
func("d") # Remove value "b" from cache and Add value "d"
```

##### Cache with TTL (Time To Live)

```python
from cachecall import cache

@cache(ttl=10)
def func(x=1):
    print("call")
    return "abc"
```

- Each 10 seconds the cache for the function ```func``` will be expired.

##### Cache with Expire Time

```python
from cachecall import cache, ExpireTime

@cache(expire_time=ExpireTime(10, 15, 20))
def func(a=None):
    print("call")
    return "xyz"
```

- Every day at 10:15:20 the value cached will expire.
- If the ```ExpireTime(10, 15, 20)``` will defined before 10:15:20 in current day, the the data cached will expired in same day, but if ExpireTime will defined after the 10:15:20, the cached value will be expired in next day.


##### TTL with Expire Time

```python
from cachecall import cache, ExpireTime

@cache(ttl=18000, expire_time=ExpireTime(6, 0, 0))
def func(a=None):
    print("call")
    return "xyz"
```
- ```ttl``` and ```expire_time``` can be used together. In this example the cached data will expire each 5 hours and every day at 6:00:00.

##### Cache group
```python
from cachecall import cache

@cache(group_name='some_group_name')
def do_a(x=None):
    print("call a")
    return "xyz"

@cache(group_name='some_group_name')
def do_b(x=None):
    print("call abc")
    return "abc"
```

- All the cached data will be stored inside a cache group called "some_group_name".
- Even though you use many different functions with the same parameters inside the same cache group, all cached data are associated with your specific function.

##### Clean cache group

```python
from cachecall import cache, clear_cache

@cache(group_name='some_group_name')
def func(x=None):
    print("call")
    return "xyz"

clear_cache('some_group_name') # Clear all cached data inside some_group_name
```

#### Keep cache ignoring some keys

```python
from cachecall import cache

@cache(ignore_keys=("y"))
def func(x=1, y=None):
    print("call")
    return x

func(1, y=1) # Cache data
func(1, y=2) # the y value is ignored and the cached data is returned
func(1, y=12345) # the y value is ignored and the cached data is returned
func(2, y=12345) # Add new data in cache
```
- When some parameter is defined in ignore keys tuple these parameter value is ignored in cache creation, and even though the value of this parameter is changed the data will be kept cached.
- Currently this feature only works if the ignored parameters are used as named arguments.  


### Documentation

#### Cache Decorator

```python
from cachecall import cache
```

##### Parameters
- max_size: *Optional[int]*
    - Number of values (or call to cached function) that will be stored in cache.
    - If None the cache size will be unlimited.
    - Default value is None.

- group_name: *Optional[str]*
    - Group name for cached data.
    - Each function has their own cache group, but if you wish to use the same cached data for different functions just use the same *group_name* for these functions. 
    - If None the group name default will be "function-name_uuid4".
    - Default value is None.

- ttl: *Optional[Union[int, float]]*
    - (Time To Live) Value in seconds to expire the cached data.
    - If None the value never expires.
    - Default value is None.

- expire_time: *Optional[ExpireTime]*
    - Specific time to expire a cached value daily.
    - If None the value never expires.
    - Default value is None.

- ignore_keys: *Optional[Tuple[str]]*
    - Tuple of parameters name that will be ignored in cache. 
    - If some cached data exists and the value of some ignored_key is changed, the data will be continued cached because all arguments mapped in ignore_keys tuple will be ignored.
    - Currently this feature only works if the ignored parameters are used as named arguments.
    - *Observation*:
        - Even though the cached function (function decorated by *cachecall.cache* decorator) no has any defined parameter the data of this function will be cached using a "no-key" key for identifying cached data.

        - The ```ignore_keys``` have be used if your cached function (function decorated by *cachecall.cache* decorator) has at least two parameters or more, because, if your function has only one parameter and this parameter is defined in ignore_keys, that means that your function will cache data normally because the only defined parameter will be ignored. Therefore, using ```ignore_keys``` for cached functions with only one defined parameter and this only parameter name passed as value in *ignore_keys* will have no effect and the data will be cached normally.

        - This behavior also will occur if you have many parameters and all parameters names are passed to ```ignore_keys```. 

#### Expire Time class
```python
from cachecall import ExpireTime

@dataclass
class ExpireTime:
    hour: int
    minute: int
    second: int
```