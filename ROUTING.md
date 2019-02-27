# Routing in hug

Hug router support applying routes to functions, methods and objects. All hug routers enforce type annotation and enable automatic argument supplying via directives.

Routers share 3 attributes:
-   Can be used directly as function decorators
-   Can be used separately from the function
-   Can be stored, modified, and chained before being used

## Using router as decorator
~~~python
import hug

@hug.get('/home')
def root():
    return 'Welcome home!'
~~~

## Declaring a router separate from a function
~~~python
# internal.py
def root():
    return 'Welcome home!'
~~~

~~~python
# external.py
import hug

import internal

router = hug.route.API(__name__)
router.get('/home')(internal.root)
~~~
or:
~~~python
# external.py
import hug

import internal

api = hug.API(__name__)
hug.get('/home', api=api)(internal.root)
~~~

## Chaining routers
~~~python
import hug

api = hug.get(on_invalid=hug.redirect.not_found)

@api.urls('/do-math', examples='number_1=1&number_2=2')
def math(number_1: hug.types.number, number_2: hug.types.number):
    return number_1 + number_2

@api
def happy_birthday(name, age: hug.types.number):
    """Says happy birthday to a user"""
    return "Happy {age} Birthday {name}!".format(**locals())
~~~
> **Note:**  when chaining routers you can call the argument you would normally pass in to the routers init function as a method on the existing router

## Common router parameters
There are a few parameters that are shared between all router types, as they are globally applicable to all currently supported interfaces

-   `api`: The API to register the route with. You can always retrieve the API singleton for the current module by doing  `hug.API(__name__)`
-   `transform`: A function to call on the the data returned by the function to transform it in some way specific to this interface
-   `output`: An output format to apply to the outputted data (after return and optional transformation)
-   `requires`: A list or single function that must all return  `True`  for the function to execute when called via this interface (commonly used for authentication)

# HTTP Routers
in addition to `hug.http` hug includes convenience decorators for all common HTTP METHODS 
- `hug.connect`
- `hug.delete`
-  `hug.get`
-  `hug.head`
-  `hug.options`
-  `hug.patch`
- `hug.post`
-  `hug.put`
-  `hug.get_post`
-  `hug.put_post`
-  `hug.trace`

These methods are functionally the same as calling `@hug.http(accept=(METHOD, ))` and are otherwise identical to the http router.

-   `urls`: A list of or a single URL that should be routed to the function. Supports defining variables within the URL that will automatically be passed to the function when  `{}`notation is found in the URL:  `/website/{page}`. Defaults to the name of the function being routed to.
-   `accept`: A list of or a single HTTP METHOD value to accept. Defaults to all common HTTP methods.
-   `examples`: A list of or a single example set of parameters in URL query param format. For example:  `examples="argument_1=x&argument_2=y"`
-   `versions`: A list of or a single integer version of the API this endpoint supports. To support a range of versions the Python builtin range function can be used.
-   `suffixes`: A list of or a single suffix to add to the end of all URLs using this router.
-   `prefixes`: A list of or a single prefix to add before all URLs using this router.
-   `response_headers`: An optional dictionary of response headers to set automatically on every request to this endpoint.
-   `status`: An optional status code to automatically apply to the response on every request to this endpoint.
-   `parse_body`: If  `True`  and the format of the request body matches one known by hug, hug will run the specified input formatter on the request body before passing it as an argument to the routed function. Defaults to  `True`.
-   `on_invalid`: A transformation function to run outputed data through, only if the request fails validation. Defaults to the endpoints specified general transform function, can be set to not run at all by setting to  `None`.
-   `output_invalid`: Specifies an output format to attach to the endpoint only on the case that validation fails. Defaults to the endpoints specified output format.
-   `raise_on_invalid`: If set to true, instead of collecting validation errors in a dictionary, hug will simply raise them as they occur.

# CLI Routing

-   `name`: The name that should execute the command from the command line. Defaults to the name of the function being routed.
-   `version`: The optional version associated with this command line application.
-   `doc`: Documentation to provide to users of this command line tool. Defaults to the functions doc string.
~~~python
import hug  
  
API = hug.API('git')  
  
@hug.object(name='git', version='1.0.0', api=API)  
class GIT(object):  
    """An example of command like calls via an Object"""  
  
  @hug.object.cli  
  def push(self, branch='master'):  
        return 'Pushing {}'.format(branch)  
  
    @hug.object.cli  
  def pull(self, branch='master'):  
        return 'Pulling {}'.format(branch)  
  
  
if __name__ == '__main__':  
    API.cli()
~~~
Example:
~~~bash
$ python my_app/router_cli.py pull -h

usage: git pull [-h] [-b BRANCH]

optional arguments:
  -h, --help            show this help message and exit
  -b BRANCH, --branch BRANCH
~~~

## Local Routing
By default all hug APIs are already valid local APIs (Python functions). Local router is used to apply type annotations and/or directives.

-   `validate`: Apply type annotations to local use of the function. Defaults to  `True`.
-   `directives`: Apply directives to local use of the function. Defaults to  `True`.
-   `version`: Specify a version of the API for local use. If versions are being used, this generally should be the latest supported.
-   `on_invalid`: A transformation function to run outputed data through, only if the request fails validation. Defaults to the endpoints specified general transform function, can be set to not run at all by setting to  `None`.
-   `output_invalid`: Specifies an output format to attach to the endpoint only on the case that validation fails. Defaults to the endpoints specified output format.
-   `raise_on_invalid`: If set to  `True`, instead of collecting validation errors in a dictionary, hug will simply raise them as they occur.

**Example with local router:**
~~~python
import hug  
   
def get_username_from_id(_id):
    return f"resolved username for id {_id}"  
  
@hug.get("/user/{id}", map_params={"id": "username"})  
@hug.local()  
def get_user(username: get_username_from_id):  
    return username  
   
@hug.get("/with-id-one")  
def get_user_with_id_one():  
    return get_user(1)
~~~
**Calls:**
~~~bash
GET http://localhost:8000/user/2

HTTP/1.0 200 OK
Date: Wed, 27 Feb 2019 08:19:54 GMT
Server: WSGIServer/0.2 CPython/3.6.5
content-type: application/json; charset=utf-8
content-length: 30

"resolved username for id two"
~~~

~~~bash
GET http://localhost:8000/with-id-one

HTTP/1.0 200 OK
Date: Wed, 27 Feb 2019 08:21:35 GMT
Server: WSGIServer/0.2 CPython/3.6.5
content-type: application/json; charset=utf-8
content-length: 30

"resolved username for id one"
~~~

**Without local router:**

~~~python
@hug.get("/user/{id}", map_params={"id": "username"})  
def get_user(username: get_username_from_id):  
    return username  
  
@hug.get("/with-id-one")  
def get_user_with_id_one():  
    return get_user("username")
~~~
**Call:**
~~~bash
GET http://localhost:8000/with-id-one

HTTP/1.0 200 OK
Date: Wed, 27 Feb 2019 08:22:41 GMT
Server: WSGIServer/0.2 CPython/3.6.5
content-type: application/json; charset=utf-8
content-length: 10

"username"
~~~
