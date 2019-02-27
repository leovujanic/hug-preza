# Authentication in Hug
- Number of authentication methods which handle the http headers for you and lets you very simply link them with your own authentication logic
- To use hug's authentication, when defining an interface, you add a `requires` keyword

  
  ## Available auth providers
Type of Authentication | Hug Authenticator Wrapper | Header Name | Header Content | Arguments to wrapped verification function  
----------------------------|----------------------------------|-----------------|-------------------------|------------  
Basic Authentication | `hug.authenticaton.basic` | Authorization | "Basic XXXX" where XXXX is username:password encoded in Base64| username, password  
Token Authentication | `hug.authentication.token` | Authorization | the token as a string| token  
API Key Authentication | `hug.authentication.api_key` | X-Api-Key | the API key as a string | api-key

Example:
~~~python
def token_verify(token):  
    secret_key = 'super-secret-key-please-change'  
  try:  
        return jwt.decode(token, secret_key, algorithm='HS256')  
    except jwt.DecodeError:  
        return False  
  
token_key_authentication = hug.authentication.token(token_verify)  

@hug.get('/token_authenticated', requires=token_key_authentication)  # noqa  
def token_auth_call(user: hug.directives.user):  
    return 'You are user: {0} with data {1}'.format(user['user'], user['data'])

~~~
Example with user directive:
~~~python
class APIUser(object):  
    """A minimal example of a rich User object"""  
  
  def __init__(self, user_id, api_key):  
        self.user_id = user_id  
        self.api_key = api_key  
  
  
def api_key_verify(api_key):  
  magic_key = '5F00832B-DE24-4CAF-9638-C10D1C642C6C' # Obviously, this would hit your database  
  if api_key == magic_key:  
        # Success!  
	  return APIUser('user_foo', api_key) 
  
api_key_authentication = hug.authentication.api_key(api_key_verify)  
 
@hug.get('/key_authenticated', requires=api_key_authentication)  # noqa  
def basic_auth_api_call(user: hug.directives.user):  
    return 'Successfully authenticated with user: {0}'.format(user.user_id)
~~~




