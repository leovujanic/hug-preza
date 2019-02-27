# Context in hug

There is a concept of a 'context' in falcon, which is a dict that lives through the whole request. It is used to integrate for example SQLAlchemy library. However, in hug's case you would expect the context to work in each interface, not only the http one based on falcon. That is why hug provides its own context, that can be used in all interfaces

## Create context
By default, the hug creates also a simple dict object as the context. However, you are able to define your own context by using the context_factory decorator.

~~~py
@hug.create_context()
def context_factory(*args, **kwargs):
    return dict()
~~~

## Delete context
After the call is finished, the context is deleted. If you want to do something else with the context at the end, you can override the default behaviour by the delete_context decorator
~~~py
@hug.delete_context()
def delete_context(context, exception=None, errors=None, lacks_requirement=None):
    pass
~~~


> Context can be everything. For example you can keep all the necessary database sessions in the context and also you can keep there all the resources that need to be dealt with after the execution of the endpoin

## Example
~~~py
# context.py
class Context:  
  def __init__(self):  
	  self._db = session_factory()  
  
  @property  
  def db(self) -> Session:  
	  return self._db  
  
  def cleanup(self, exception=None):  
      try:  
          if exception is None:  
              self.db.commit()  
          else:  
              self.db.rollback()  
      except Exception:  
	       self.db.close()  
           # re-raise exception for error handlers  
		   raise
~~~

~~~py
# app.py
@hug.context_factory()  
def create_context(*args, **kwargs):  
	return Context()  
  
@hug.delete_context()  
def delete_context(context: Context, exception=None, **kwargs):  
	context.cleanup(exception)
~~~

~~~py
# directives.py
@hug.directive()  
class DBSession(Session):  
  def __new__(cls, *args, context: Context = None, **kwargs):  
	  return context.db
~~~