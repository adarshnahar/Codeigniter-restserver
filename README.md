#Codeigniter-restserver
======================

This is rest server implemented with codeigniter php framework


A fully RESTful server implementation for CodeIgniter using one library, one
config file and one controller.

## Requirements

1. PHP 5.2 or greater
2. CodeIgniter 2.1.0 to 3.0-dev

## Handling Requests

When your controller extends from `REST_Controller`, the method names will be appended with the HTTP method used to access the request. If you're  making an HTTP `GET` call to `/books`, for instance, it would call a `Books#index_get()` method.

This allows you to implement a RESTful interface easily:

	class Books extends REST_Controller
	{
		public function index_get()
		{
			// Display all books
		}

		public function index_post()
		{
			// Create a new book
		}
	}

`REST_Controller` also supports `PUT` and `DELETE` methods, allowing you to support a truly RESTful interface.


Accessing parameters is also easy. Simply use the name of the HTTP verb as a method:

	$this->get('blah'); // GET param
	$this->post('blah'); // POST param
	$this->put('blah'); // PUT param

The HTTP spec for DELETE requests precludes the use of parameters.  For delete requests, you can add items to the URL

		public function index_delete($id)
		{
    		$this->response(array(
        		'returned from delete:' => $id,
    		));			
		}

## Content Types

`REST_Controller` supports a bunch of different request/response formats, including XML, JSON and serialised PHP. By default, the class will check the URL and look for a format either as an extension or as a separate segment.

This means your URLs can look like this:

	http://example.com/books.json
	http://example.com/books?format=json

This can be flaky with URI segments, so the recommend approach is using the HTTP `Accept` header:

	$ curl -H "Accept: application/json" http://example.com



## Responses

The class provides a `response()` method that allows you to return data in the user's requested response format.

Returning any object / array / string / whatever is easy:

	public function index_get()
	{
		$this->response($this->db->get('books')->result());
	}


This will automatically return an `HTTP 200 OK` response. You can specify the status code in the second parameter:

	public function index_post()
	{
		// ...create new book

		$this->response($book, 201); // Send an HTTP 201 Created
	}

If you don't specify a response code, and the data you respond with `== FALSE` (an empty array or string, for instance), the response code will automatically be set to `404 Not Found`:

	$this->response(array()); // HTTP 404 Not Found
	
	
## Authentication

This class also provides rudimentary support for HTTP basic authentication and/or the securer HTTP digest access authentication.

You can enable basic authentication by setting the `$config['rest_auth']` to `'basic'`. The `$config['rest_valid_logins']` directive can then be used to set the usernames and passwords able to log in to your system. The class will automatically send all the correct headers to trigger the authentication dialogue:

	$config['rest_valid_logins'] = array( 'username' => 'password', 'other_person' => 'secure123' );

Enabling digest auth is similarly easy. Configure your desired logins in the config file like above, and set `$config['rest_auth']` to `'digest'`. The class will automatically send out the headers to enable digest auth.

Both methods of authentication can be secured further by using an IP whitelist. If you enable `$config['rest_ip_whitelist_enabled']` in your config file, you can then set a list of allowed IPs.

Any client connecting to your API will be checked against the whitelisted IP array. If they're on the list, they'll be allowed access. If not, sorry, no can do hombre. The whitelist is a comma-separated string:

	$config['rest_ip_whitelist'] = '123.456.789.0, 987.654.32.1';

Your localhost IPs (`127.0.0.1` and `0.0.0.0`) are allowed by default.


## API Keys

In addition to the authentication methods above, the `REST_Controller` class also supports the use of API keys. Enabling API keys is easy. Turn it on in your **config/rest.php** file:

	$config['rest_enable_keys'] = TRUE;


You'll need to create a new database table to store and access the keys. `REST_Controller` will automatically assume you have a table that looks like this:

	CREATE TABLE `keys` (
	  `id` int(11) NOT NULL AUTO_INCREMENT,
	  `key` varchar(40) NOT NULL,
	  `level` int(2) NOT NULL,
	  `ignore_limits` tinyint(1) NOT NULL DEFAULT '0',
	  `date_created` int(11) NOT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=MyISAM DEFAULT CHARSET=utf8;

The class will look for an HTTP header with the API key on each request. An invalid or missing API key will result in an `HTTP 403 Forbidden`.

By default, the HTTP will be `X-API-KEY`. This can be configured in **config/rest.php**.

$ curl -X POST -H "X-API-KEY: some_key_here" http://example.com/books


