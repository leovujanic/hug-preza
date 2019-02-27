# Extending hug

~~~py
# app.py
import hug

from my_app import error_module, directives, resource_one

@hug.extend_api()  
def apis():  
   return [directives, error, setup, models]
~~~

# Running hug

### For development
~~~bash
$ hug -f my_app/app.py
------

$ hug -h

usage: hug [-h] [-v] [-f FILE] [-m MODULE] [-ho HOST] [-p PORT] [-n] [-ma]
           [-i INTERVAL] [-c COMMAND] [-s]

Hug API Development Server

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  -f FILE, --file FILE  file
  -m MODULE, --module MODULE
                        module
  -ho HOST, --host HOST
                        host
  -p PORT, --port PORT  A Whole number
  -n, --no_404_documentation
                        Providing any value will set this to true
  -ma, --manual_reload  Providing any value will set this to true
  -i INTERVAL, --interval INTERVAL
                        A Whole number
  -c COMMAND, --command COMMAND
                        command
  -s, --silent          Providing any value will set this to true
~~~

### Production:
~~~bash
$ uwsgi --http 0.0.0.0:8000 --wsgi-file my_app/app.py --callable __hug_wsgi__

$ gunicorn --bind=0.0.0.0:8000 my_app.app:__hug_wsgi__
~~~

## Testing

~~~py
def test_route():
	response = hug.test.get(app, "v1/organizations/1/models")
  
	self.assertEqual(response.status, HTTP_200)  
	self.assertEqual(response.data, {})
~~~

~~~py
# test_api.py

from my_app import api_module, error_module

# ignore app.py and bootstraping

# noinspection PyPep8Naming  
def setUpModule():  
   hug.API(api_module.__name__).extend(api_error_module)

def test_route():
	resp = hug.test.post(api_module, "/route", headers={})  
	 
	self.assertEqual(resp.status, HTTP_400)
~~~