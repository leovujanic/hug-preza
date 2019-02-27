# DIRECTIVES
Directives are arguments that have been registered to automatically provide a parameter value based on knowledge known to the interface.

### Example - built-in session directive:

~~~python
@hug.directive()
def session(context_name='session', request=None, **kwargs):
    """Returns the session associated with the current request"""
    return request and request.context.get(context_name, None) or None
~~~
Usage:
~~~python
@hug.get()
def my_endpoint(session: hug.directives.session):
    session # is here automatically, without needing to be passed in

# prefixed with `hug_`
@hug.get()
def my_endpoint(hug_session='alternative_session_key'):
    session # is here automatically, without needing to be passed in
~~~

##  Built-in directives
-   `hug.directives.Timer` (`hug_timer=precision`): Stores the time the interface was initially called, returns how much time has passed since the function was called, if casted as a float. Automatically converts to the time taken when returned as part of a JSON structure. The default value specifies the float precision desired when keeping track of the time passed.
-   `hug.directives.module` (`hug_module`): Passes along the module that contains the API associated with this endpoint.
-   `hug.directives.api` (`hug_api`): Passes along the hug API singleton associated with this endpoint.
-   `hug.directives.api_version` (`hug_api_version`): Passes along the version of the API being called.
-   `hug.directives.documentation` (`hug_documentation`): Generates and passes along the entire set of documentation for the API that contains the endpoint.
-   `hug.directives.session` (`hug_session=context_name`): Passes along the session associated with the current request. The default value provides a different key whose value is stored on the request.context object.
-   `hug.directives.user` (`hug_user`): Passes along the user object associated with the request.
-   `hug.directives.CurrentAPI` (`hug_current_api`): Passes along a smart, version-aware API caller, to enable calling other functions within your API, with reassurance that the correct function is being called for the version of the API being requested.

## Building custom directives
Hug provides the `@hug.directive()` to enable creation of new directives. It takes one argument: apply_globally, which defaults to False. If you set this parameter to True, the hug directive will be automatically made available as a magic `hug_` argument on all endpoints outside of your defined API. This is not a concern if you're applying directives via type annotation.

~~~python
@hug.directive()  
class DBSession(Session):  
    """Database session directive """  
  def __new__(cls, *args, context: Context = None, **kwargs):  
        """  
	  :param context: Context, application context  :return: Session, see Context.db docs  
	 """  
	 return context.db
~~~
Usage:
~~~python
from my_app.directives import DBSession as Db

@router.get("/models")  
def fetch_all(db: Db):  
	query = db.query(MyModel)  
    models = query.order_by(MyModel.id.desc()).all()
    return _serializer.serialize(models)
~~~

## Common directive key word parameters

Independent of interface, the following key word arguments will be passed to the directive:

-   `interface`  - The interface that the directive is being run through. Useful for conditionally injecting data (via the decorator) depending on the interface it is being called through, as demonstrated at the bottom of this section.
-   `api`  - The API singleton associated with this endpoint.
~~~python
@directive()
def my_directive(default=None, interface=None, **kwargs):
    if interface == hug.interface.CLI:
        return 'CLI specific'
    elif interface == hug.interface.HTTP:
        return 'HTTP specific'
    elif interface == hug.interface.Local:
        return 'Local'

    return 'unknown'
~~~

## HTTP directive key word parameters

Directives are passed the following additional keyword parameters when they are being run through an HTTP interface:

-   `response`: The HTTP response object that will be returned for this request.
-   `request`: The HTTP request object that caused this interface to be called.
-   `api_version`: The version of the endpoint being hit.

## CLI directive key word parameters

Directives get one additional argument when they are run through a command line interface:

-   `argparse`: The argparse instance created to parse command line arguments.