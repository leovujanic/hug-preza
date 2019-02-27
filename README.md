# HUG

- The cleanest way to create **HTTP REST APIs** on **Python3**
- It’s **NOT** a Web API Framework (one of the functions it performs)
- It is a framework for exposing idiomatically correct and standard internal Python APIs externally

### What does it mean?
- Currently, this means that you can expose existing Python functions / APIs over HTTP and CLI in addition to standard Python.
- As time goes on more interfaces will be supported (**???**)



~~~python
import hug  
  
  
@hug.cli()  
@hug.post("/user")  
@hug.local()  
def register_user(username: str, password: str):  
    print(f"Creating user {username} with password '{password}'")  
  
    return {  
        "id": 1,  
        "username": username,  
    }  
  
  
if __name__ == "__main__":  
    register_user.interface.cli()
~~~

**Usage:**
* CLI interface:
	```bash
	$ python my_app/app.py admin admin
	Creating user admin with password 'admin'
	{'id': 1, 'username': 'admin'}
	```
*  HTTP interface
		**Request:**
	~~~bash
	POST http://localhost:8000/user  
	Content-Type: application/json  
	  
	{  
	  "username": "leo",  
	  "password": 123456  
	}  
	  
	###
	~~~
	**Response:**
	~~~bash
	POST http://localhost:8000/user

	HTTP/1.0 200 OK
	Date: Wed, 27 Feb 2019 07:02:21 GMT
	Server: WSGIServer/0.2 CPython/3.6.5
	Content-Type: application/json; charset=utf-8
	Content-Length: 28

	{
	  "id": 1,
	  "username": "leo"
	}
	~~~
* Python native
	~~~python
	Python 3.6.5 (default, Jun 17 2018, 12:13:06) 
	Type 'copyright', 'credits' or 'license' for more information
	IPython 7.3.0 -- An enhanced Interactive Python. Type '?' for help.

	In [1]: from my_app.register_user import register_user                                                                                                                                               

	In [2]: register_user("admin", "admin")                                                                                                                                                              
	Creating user admin with password 'admin'
	Out[2]: {'id': 1, 'username': 'admin'}
	~~~

### How it works?
This central concept also frees hug to rely on the fastest and best of breed components for every interface it supports:

-   [Falcon](https://github.com/falconry/falcon)  is leveraged when exposing to HTTP for it's impressive performance at this task
-   [Argparse](https://docs.python.org/3/library/argparse.html)  is leveraged when exposing to CLI for the clean consistent interaction it enables from the command line

Hug provides several mechanisms to enable your exposed interfaces to have additional capabilities not defined by the base Python function

- **Enforced type annotations** - types are called with the data passed into that field, if an exception is thrown it's seen as invalid
- **Directives** - hug interfaces allow replacing Python function parameters with dynamically pulled data via directives
- **Requires** - hug requirements allow you to specify requirements that must be met only for specified interfaces
		`@hug.get(requires=hug.authentication.basic(hug.authentication.verify('User1', 'mypassword')))`
- **Transformations** - hug transformations enable changing the result of a function but only for the specified interface	
~~~python
import hug  

class User:  

	def __init__(self, first_name, last_name):  
		self.last_name = last_name  
		self.first_name = first_name  

	def __str__(self):  
		return f"{self.first_name} {self.last_name}" 

@hug.get("/user", transform=str)  
def test():  
	user = User("Leo", "Vujanic")  
	return user
~~~~
Response:
~~~
GET http://localhost:8000/user-transform

HTTP/1.0 200 OK
Date: Wed, 27 Feb 2019 07:26:34 GMT
Server: WSGIServer/0.2 CPython/3.6.5
content-type: application/json; charset=utf-8
content-length: 13

"Leo Vujanic"

Response code: 200 (OK); Time: 14ms; Content length: 13 bytes
~~~

- **Input / Output formats** - hug provides an extensive number of built-in input and output formats. `@hug.get(output_format=hug.output_format.json)`
	-  These formats define how data should be sent to your API function and how it will be returned
    -   All of hugs built-in output formats are found in  `hug/output_format.py`
    -   All of hugs built-in input formats are found in  `hug/input_format.py`
    -   The default assumption for output_formatting is JSON
    
    
Docs:
   - [ROUTING](ROUTING.md)
   - [TYPE_ANNOTATIONS](TYPE_ANNOTATIONS.md)
   - [DIRECTIVES](DIRECTIVES.md)
   - [OUTPUT FORMATS](OUTPUT_FORMATS.md)
   - [ERROR HANDLERS](ERROR_HANDLERS.md)
   - [CONTEXT](CONTEXT.md)
   - [AUTHENTICATION](AUTHENTICATION.md)
   - [ADDITIONAL](ADDITIONAL.md)
