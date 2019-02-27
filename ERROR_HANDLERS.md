# Error handlers

Catch all exceptions
~~~py
@hug.exception()  
def exception_handler(exception, response=None):  
    logger.error("Unhandled exception: {}".format(exception))  
    response.status = HTTP_500  
    return format_error("Internal server error", INTERNAL_SERVER_ERROR)
~~~

Catch exceptions of specific type
~~~py
@hug.exception(IntegrityError)  
def integrity_error_handler(exception, response):  
	logger.error(exception)  
    response.status = HTTP_409
    message = ("The request could not be completed due to a conflict "  
			 "with the current state of the resource")  
    return format_error(message, INTEGRITY_ERROR)
~~~


## Not found handler
- defaults to documentation
~~~py
@hug.not_found()  
def not_found_exception_handler(request=None):  
	msg = "Requested path does not exists: {}".format(request.path)  
    return format_error(msg, ENDPOINT_DOES_NOT_EXISTS)
~~~


## Custom error handler

~~~

~~~
