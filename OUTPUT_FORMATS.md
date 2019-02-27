# Output formats

- The default output format for all hug APIs is JSON, but can be changed with:
	~~~python
	import hug

	hug.API(__name__).http.output_format = hug.output_format.html
	~~~
	or
	~~~python
	@hug.default_output_format()
	def my_output_formatter(data, request, response):
	    # Custom output formatting code
	    return data
	~~~
	or by specifying an output_format for a specific endpoint
	~~~python
	@hug.get(output=hug.output_format.html)
	def my_endpoint():
	    return "<html></html>"
	~~~

 - Output format may be a collection of different output formats that get used conditionally

	~~~python
	suffix_output = hug.output_format.suffix({'.js': hug.output_format.json,
	                                          '.html':hug.output_format.html})

	@hug.get(('my_endpoint.js', 'my_endoint.html'), output=suffix_output)
	def my_endpoint():
	    return ''
	~~~
## Built-in hug output formats

-   `hug.output_format.json`: The default hug output formatter for all endpoints; outputs in Javascript Serialized Object Notation (JSON).
-   `hug.output_format.text`: Outputs in a plain text format.
-   `hug.output_format.html`: Outputs Hyper Text Markup Language (HTML).
-   `hug.output_format.json_camelcase`: Outputs in the JSON format, but first converts all keys to camelCase to better conform to Javascript coding standards.
-   `hug.output_format.pretty_json`: Outputs in the JSON format, with extra whitespace to improve human readability.
-   `hug.output_format.image(format)`: Outputs an image (of the specified format).
    
    -   There are convenience calls in the form `hug.output_format.{FORMAT}_image for the following image types: 'png', 'jpg', 'bmp', 'eps', 'gif', 'im', 'jpeg', 'msp', 'pcx', 'ppm', 'spider', 'tiff', 'webp', 'xbm', 'cur', 'dcx', 'fli', 'flc', 'gbr', 'gd', 'ico', 'icns', 'imt', 'iptc', 'naa', 'mcidas', 'mpo', 'pcd', 'psd', 'sgi', 'tga', 'wal', 'xpm', and 'svg'. Automatically works on returned file names, streams, or objects that produce an image on read, save, or render.
-   `hug.output_format.video(video_type, video_mime, doc)`: Streams a video back to the user in the specified format.
    
    -   There are convenience calls in the form `hug.output_format.{FORMAT}_video for the following video types: 'flv', 'mp4', 				'm3u8', 'ts', '3gp', 'mov', 'avi', and 'wmv'. Automatically works on returned file names, streams, or objects that produce a video on read, save, or render.
-   `hug.output_format.file`: Will dynamically determine and stream a file based on its content. Automatically works on returned file names and streams.
    
-   `hug.output_format.on_content_type(handlers={content_type: output_format}, default=None)`: Dynamically changes the output format based on the request content type.
    
-   `hug.output_format.suffix(handlers={suffix: output_format}, default=None)`: Dynamically changes the output format based on a suffix at the end of the requested path.
-   `hug.output_format.prefix(handlers={suffix: output_format}, default=None)`: Dynamically changes the output format based on a prefix at the beginning of the requested path.

## Creating a custom output format
~~~python
@hug.format.content_type('file/text')
def format_as_text(data, request=None, response=None):
    return str(content).encode('utf8')
~~~
Apply specific output format on valid request
~~~python
@hug.output_format.on_valid('file/text')
def format_as_text_when_valid(data, request=None, response=None):
    return str(content).encode('utf8')
~~~
>output format is simply a function with a content type attached that takes a data argument, and optionally a request and response, and returns properly encoded and formatted data