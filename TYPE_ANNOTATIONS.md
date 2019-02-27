# Type annotations in hug

Python3 type annotations are used for validation and API specification. Within the context of hug, annotations should be set to one of 4 things.

-   A cast function, built-in, or your own (`str`, `int`, etc) that takes a value, casts it and then returns it, raising an exception if it is not in a format that can be cast into the desired type
-   A hug type (`hug.types.text`, `hug.types.number`, etc.). These are essentially built-in cast functions that provide more contextual information, and good default error messages
-   A  [marshmallow](https://marshmallow.readthedocs.org/en/latest/)  type and/or schema. In hug 2.0.0 Marshmallow is a **first class citizen**, and all fields and schemas defined with it can be used in hug as type annotations
-   A string. When a basic Python string is set as the type annotation it is used by hug to generate documentation, but does not get applied during the validation phase

### Example
~~~python
import hug

@hug.get()
def hello(first_name: hug.types.text, last_name: 'Family Name', age: int):
    return f"Hi {first_name} {last_name}!"
~~~

# Built in hug types

-   `number`: Validates that a whole number was passed in
-   `float_number`: Validates that a valid floating point number was passed in
-   `decimal`: Validates and converts the provided value into a Python Decimal object
-   `uuid`: Validates that the provided value is a valid UUID
-   `text`: Validates that the provided value is a single string parameter
-   `multiple`: Ensures the parameter is passed in as a list (even if only one value is passed in)
-   `boolean`: A basic naive HTTP style boolean where no value passed in is seen as  `False`and any value passed in (even if its  `false`) is seen as  `True`
-   `smart_boolean`: A smarter, but more computentionally expensive, boolean that checks the content of the value for common true / false formats (true, True, t, 1) or (false, False, f, 0)
-   `delimited_list(delimiter)`: splits up the passed in value based on the provided delimiter and then passes it to the function as a list
-   `one_of(values)`: Validates that the passed in value is one of those specified
-   `mapping(dict_of_passed_in_to_desired_values)`: Like  `one_of`, but with a dictionary of acceptable values, to converted value.
-   `multi(types)`: Allows passing in multiple acceptable types for a parameter, short circuiting on the first acceptable one
-   `in_range(lower, upper, convert=number)`: Accepts a number within a lower and upper bound of acceptable values
-   `less_than(limit, convert=number)`: Accepts a number within a lower and upper bound of acceptable values
-   `greater_than(minimum, convert=number)`: Accepts a value above a given minimum
-   `length(lower, upper, convert=text)`: Accepts a a value that is within a specific length limit
-   `shorter_than(limit, convert=text)`: Accepts a text value shorter than the specified length limit
-   `longer_than(limit, convert=text)`: Accepts a value up to the specified limit
-   `cut_off(limit, convert=text)`: Cuts off the provided value at the specified index


# Extending and creating new hug types
1. inherit from the base type and then `__call__`
~~~
import hug

class TheAnswer(hug.types.number):
    """My new documentation"""

    def __call__(self, value):
        value = super().__call__(value)
        if value != 42:
            raise ValueError('Value is not the answer to everything.')
~~~

~~~python
@hug.type(extend=hug.types.number)
def the_answer(value):
    """My new documentation"""
  if value != 42:
        raise ValueError('Value is not the answer to everything.')
  return "Right choice!"

# usage

@hug.get("/answer")
def answer(user_choice: the_answer):
    return user_choice
~~~
**Calls:**
~~~bash
GET http://localhost:8000/answer?user_choice=1

HTTP/1.0 400 Bad Request
Date: Wed, 27 Feb 2019 08:46:51 GMT
Server: WSGIServer/0.2 CPython/3.6.5
content-type: application/json; charset=utf-8
content-length: 69

{
  "errors": {
    "user_choice": "Value is not the answer to everything."
  }
}
~~~

~~~bash
GET http://localhost:8000/answer?user_choice=42

HTTP/1.0 200 OK
Date: Wed, 27 Feb 2019 08:48:00 GMT
Server: WSGIServer/0.2 CPython/3.6.5
content-type: application/json; charset=utf-8
content-length: 15

"Right choice!"
~~~

# Marshmallow integration
[Marshmallow](https://marshmallow.readthedocs.org/en/latest/) is an advanced serialization, deserialization, and validation library
~~~python
import datetime as dt

import hug
from marshmallow import fields
from marshmallow.validate import Range

@hug.get('/dateadd', examples="value=1973-04-10&addend=63")
def dateadd(value: fields.DateTime(), addend: fields.Int(validate=Range(min=1))):
    """Add a value to a date."""
    delta = dt.timedelta(days=addend)
    result = value + delta
    return {'result': result}
~~~

# Voluptuous integration

~~~python
class VoluptuousType(Type):
	"""Base class for voluptuous based validators

	 .. seealso::
		 http://www.hug.rest/website/learn/type_annotation
	 """
	 schema: Schema = None

	  def __call__(self, value, *args, **kwargs):
	      try:
	           return self.schema(value)
		   except MultipleInvalid as e:
	           raise ValueError(e)
~~~

**Usage:**
~~~python
class BodyValidator(VoluptuousType):
    schema = schema({"name": str})

@hug.post("body")
def process_body(body: BodyValidator):
    return body
~~~
