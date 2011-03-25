# Overview

This plugin provides the ability for your RabbitMQ server to perform
authentication (determining who can log in) and authorisation
(determining what permissions they have) by making requests to an HTTP
server.

Note: it's at an early stage of development, although it's
conceptually very simple.

# Requirements

You can build and install it like any other plugin (see
[the plugin development guide](http://www.rabbitmq.com/plugin-development.html)).

# Enabling the plugin

To enable the plugin, set the value of the `auth_backends` configuration item
for the `rabbit` application to include `rabbit_auth_backend_http`.
`auth_backends` is a list of authentication providers to try in order.

So a configuration fragment that enables this plugin *only* would look like:

    [{rabbit, [{auth_backends, [rabbit_auth_backend_http]}]}].

to use only HTTP, or:

    [{rabbit,
      [{auth_backends, [rabbit_auth_backend_http, rabbit_auth_backend_internal]}]
     }].

to use both HTTP and the internal database.

# Configuring the plugin

You need to configure the plugin to know which URIs to point at.

A minimal configuration file might look like:

    [
      {rabbit, [{auth_backends, [rabbit_auth_backend_http]}]},
      {rabbit_auth_backend_http,
       [{user_path,     "http://some-server/auth/user"},
        {vhost_path,    "http://some-server/auth/vhost"},
        {resource_path, "http://some-server/auth/resource"}]}
    ].

# What must my web server do?

This plugin will make GET requests against the URIs listed in the
configuration file. It will add query string parameters as follows:

### user_path

* `username` - the name of the user
* `password` - the password provided (may be missing if e.g. rabbitmq-auth-mechanism-ssl is used)

### vhost_path

* `username`   - the name of the user
* `vhost`      - the name of the virtual host being accessed
* `permission` - the access level to the vhost:
  * `read` (meaning learn it exists)
  * `write` (meaning log in and use it)

### resource_path

* `username`   - the name of the user
* `vhost`      - the name of the virtual host containing the resource
* `resource`   - the type of resource (`exchange`, `queue`)
* `name`       - the name of the resource
* `permission` - the access level to the resource (`configure`, `write`, `read`) - see [the admin guide](http://www.rabbitmq.com/admin-guide.html#access-control) for their meaning

Your web server should always return HTTP 200 OK, with a body
containing a single word:

* `deny`  - deny access to the user / vhost / resource
* `allow` - allow access to the user / vhost / resource
* `admin` - (for `user_path` only) - allow access, and mark the user as an administrator

# Debugging

Check the RabbitMQ logs.

# Example

In `examples/rabbitmq_auth_backend_django` there's a very simple
Django app that can be used for authentication. On Debian / Ubuntu you
should be able to run start.sh to launch it after installing the
python-django package. It's really not designed to be anything other
than an example.

See `examples/README` for slightly more information.
