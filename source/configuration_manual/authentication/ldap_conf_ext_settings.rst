.. _authentication-ldap_conf_ext_settings:

============================
Settings in ldap conf ext
============================

This page enlists settings being configured in the ldap configuration files
(e.g, conf.d/auth-ldap.conf.ext) for userdb, passdb or retieving sieve script
from ldap.

.. Note:: The ldap configuration files are opened as root, so should be owned by root and mode 0600.


.. _ldap_conf_ext_setting-auth_bind:

``auth_bind``
---------------

- Default: ``no``
- Values:  :ref:`boolean`

Set yes to use authentication binding for verifying password's validity.
This works by logging into LDAP server using the username and password given by client.
The :ref:`ldap_conf_ext_setting-pass_filter` is used to find the DN for
the user. Note that the pass_attrs is still used, only the password field
is ignored in it. Before doing any search, the binding is switched back
to the default DN.

.. note::
  If you're not using authentication binding, you'll need to give
  dovecot-auth read access to userPassword field in the LDAP server.
  With OpenLDAP this is done by modifying /etc/ldap/slapd.conf.

.. todo::
   I omitted below for now
   There should already be something like this:
   access to attribute=userPassword
   by dn="<dovecot's dn>" read # add this
   by anonymous auth
   by self write
   by * none


.. _ldap_conf_ext_setting-auth_bind_userdn:

``auth_bind_userdn``
--------------------

- Default: ``no``
- Values:  :ref:`boolean`

If authentication binding is used, you can save one LDAP request per login
if users' DN can be specified with a common template. The template can use
the standard %variables (see :ref:`ldap_conf_ext_setting-user_filter`).
Note that you can't use any pass_attrs if you use this setting.

If you use this setting, it's a good idea to use a different
dovecot-ldap.conf.ext for userdb (it can even be a symlink, just as long as
the filename is different in userdb's args). That way one connection is used
only for LDAP binds and another connection is used for user lookups.
Otherwise the binding is changed to the default DN before each user lookup.

Example:

.. code-block:: none

   auth_bind_userdn = cn=%u,ou=people,o=org


.. _ldap_conf_ext_setting-base:

``base``
--------

- Default: <empty>
- Values:  :ref:`string`

LDAP base. %variables (see :ref:`config_variables`) can be used here.


Example:

.. code-block:: none

   base = dc=mail, dc=example, dc=org


.. _ldap_conf_ext_setting-blocking:

``blocking``
------------

- Default: ``no``
- Values:  :ref:`boolean`

By default all LDAP lookups are performed by the auth master process.
If blocking=yes, auth worker processes are used to perform the lookups.
Each auth worker process creates its own LDAP connection so this can
increase parallelism. With blocking=no the auth master process can
keep 8 requests pipelined for the LDAP connection, while with blocking=yes
each connection has a maximum of 1 request running. For small systems the
blocking=no is sufficient and uses less resources.


.. _ldap_conf_ext_setting-debug_level:

``debug_level``
---------------

- Default: ``0``
- Values:  :ref:`uint`

LDAP library debug level as specified by LDAP_DEBUG_* in ldap_log.h.
Value ``-1`` means everything. You may need to recompile OpenLDAP with debugging enabled
to get enough output.


.. _ldap_conf_ext_setting-dn:

``dn``
------

- Default: <empty>
- Values:  :ref:`string`

Specify the Distinguished Name (the username used to login to the LDAP server).
Leave it commented out to bind anonymously (useful with :ref:`ldap_conf_ext_setting-auth_bind` = yes).

Example:

.. code-block:: none

   dn = uid=dov-read,dc=ocn,dc=ad,dc=jp,dc=.


.. _ldap_conf_ext_setting-dnpass:

``dnpass``
------------

- Default: <empty>
- Values:  :ref:`string`

Password for LDAP server, used if :ref:`ldap_conf_ext_setting-dn` is specified.


.. _ldap_conf_ext_setting-default_pass_scheme:

``default_pass_scheme``
-----------------------

- Default: ``crypt``
- Values:  :ref:`string`

Default password scheme. "{scheme}" before password overrides this.

See :ref:`authentication-password_schemes` for a list of supported schemes.


.. _ldap_conf_ext_setting-deref:

``deref``
---------

- Default: ``never``
- Values:  ``never``, ``searching``, ``finding``, ``always``

Specify dereference which is set as an LDAP option.


.. _ldap_conf_ext_setting-hosts:

``hosts``
---------

- Default: <empty>
- Values:  :ref:`string`

A space separated list of LDAP hosts to connect to.
Configure either this setting or :ref:`ldap_conf_ext_setting-uris` to specify
what LDAP server(s) to conenct to.
You can also use host:port syntax to use different ports.

Example:

.. code-block:: none

   hosts = 10.10.10.10 10.10.10.11 10.10.10.12 

See also :ref:`ldap_conf_ext_setting-uris`


.. _ldap_conf_ext_setting-iterate_attrs:

``iterate_attrs``
-----------------

- Default: <empty>
- Values:  :ref:`string`

Attributes to get a list of all users
See also :ref:`authentication-ldap_backend_configuration`

Example:

.. code-block:: none

   iterate_attrs = mailRoutingAddress=user


.. _ldap_conf_ext_setting-iterate_filter:

``iterate_filter``
------------------

- Default: <empty>
- Values:  :ref:`string`

Filter to get a list of all users
See also :ref:`authentication-ldap_backend_configuration`

Example:

.. code-block:: none

   iterate_filter = (objectClass=smiMessageRecipient)


.. _ldap_conf_ext_setting-ldaprc_path:

``ldaprc_path``
---------------

- Default: <empty>
- Values:  :ref:`string`


If a non-empty value is set, it will be set to the LDAPRC environment variable.


.. _ldap_conf_ext_setting-ldap_version:

``ldap_version``
----------------

- Default: ``3``
- Values:  :ref:`uint`

LDAP protocol version to use. Likely 2 or 3.


.. _ldap_conf_ext_setting-pass_attrs:

``pass_attrs``
--------------

- Default: <empty>
- Values:  :ref:`string`

Specify user attributes to be retrived from LDAP in passdb look up.
See also :ref:`authentication-ldap_backend_configuration`

Password checking attributes:
* user: Virtual user name (user@domain), if you wish to change the user-given username to something else
* password: Password, may optionally start with {type}, eg. {crypt}

Example:

.. code-block:: none

   pass_attrs = uid=user,userPassword=password,\
                homeDirectory=userdb_home,uidNumber=userdb_uid,gidNumber=userdb_gid


There are also other special fields which can be returned.
See :ref:`authentication-password_database_extra_fields`

If you wish to avoid two LDAP lookups (passdb + userdb), you can use
userdb prefetch instead of userdb ldap in dovecot.conf. In that case you'll
also have to include user_attrs in pass_attrs field prefixed with ``userdb_``
string.


.. _ldap_conf_ext_setting-pass_filter:

``pass_filter``
---------------

- Default: <empty>
- Values:  :ref:`string`

Filter for password lookups (passdb lookup)
See also :ref:`authentication-ldap_backend_configuration`

Example:

.. code-block:: none

   pass_filter = (&(objectClass=posixAccount)(uid=%u))


.. _ldap_conf_ext_setting-sasl_authz_id:

``sasl_authz_id``
-----------------

- Default: <empty>
- Values:  :ref:`string`

SASL authorization ID, ie. the dnpass is for this "master user", but the
dn is still the logged in user. Normally you want to keep this empty.


.. _ldap_conf_ext_setting-sasl_bind:

``sasl_bind``
-------------

- Default: ``no``
- Values:  :ref:`boolean`

Set yes to use SASL binding instead of the simple binding. Note that this changes
ldap_version automatically to be 3 if it's lower.


.. _ldap_conf_ext_setting-sasl_mech:

``sasl_mech``
-------------

- Default: <empty>
- Values:  :ref:`string`

SASL mechanism names (a space-separated list of candidate mechanisms) to use.

.. todo:: may need to list such mech names?


.. _ldap_conf_ext_setting-sasl_realm:

``sasl_realm``
--------------

- Default: <empty>
- Values:  :ref:`string`

SASL realm to use.


.. _ldap_conf_ext_setting-scope:

``scope``
---------

- Default: ``subtree``
- Values:  ``base, onelevel, subtree``

This specifies the search scope.


.. _ldap_conf_ext_setting-tls:

``tls``
-------

- Default: ``no``
- Values:  :ref:`boolean`

Set to yes to use TLS to connect to the LDAP server.


.. _ldap_conf_ext_setting-tls_ca_cert_file:

``tls_ca_cert_file``
--------------------

- Default: <empty>
- Values:  :ref:`string`

Specify a value for TLS ``tls_ca_cert_file`` option.
Currently supported only with OpenLDAP:


.. _ldap_conf_ext_setting-tls_ca_cert_dir:

``tls_ca_cert_dir``
-------------------

- Default: <empty>
- Values:  :ref:`string`

Specify a value for TLS ``tls_ca_cert_dir`` option.
Currently supported only with OpenLDAP.


.. _ldap_conf_ext_setting-tls_cipher_suite:

``tls_cipher_suite``
--------------------

- Default: <empty>
- Values:  :ref:`string`

Specify a value for TLS ``tls_cipher_suite`` option.
Currently supported only with OpenLDAP.


.. _ldap_conf_ext_setting-tls_cert_file:

``tls_cert_file``
-----------------

- Default: <empty>
- Values:  :ref:`string`

Specify a value for TLS ``tls_cert_file`` option.
Currently supported only with OpenLDAP.


.. _ldap_conf_ext_setting-tls_key_file:

``tls_key_file``
----------------

- Default: <empty>
- Values:  :ref:`string`

Specify a value for TLS ``tls_key_file`` option.
Currently supported only with OpenLDAP.


.. _ldap_conf_ext_setting-tls_require_cert:

``tls_require_cert``
--------------------

- Default: <empty>
- Values: ``never, hard, demand, allow, try``

Specify a value for TLS ``tls_require_cert`` option.
Currently supported only with OpenLDAP.


.. _ldap_conf_ext_setting-user_attrs:

``user_attrs``
--------------

- Default: <empty>
- Values:  :ref:`string`

Specify user attributes to be retrived from LDAP (in userdb look up)
See also :ref:`authentication-ldap_backend_configuration`
User attributes are given in LDAP-name=dovecot-internal-name list.
The internal names are:

======== ========================
name      Description
======== ========================
uid      System UID
gid      System GID
home     Home directory
mail     Mail location
======== ========================

There are also other special fields which can be returned.

See :ref:`authentication-user_extra_field`

Example:

.. code-block:: none

   user_attrs = homeDirectory=home,uidNumber=uid,gidNumber=gid


.. _ldap_conf_ext_setting-user_filter:

``user_filter``
---------------

- Default: <empty>
- Values:  :ref:`string`

Filter for user lookup (userdb lookup). 
See also :ref:`authentication-ldap_backend_configuration`

Below variables can be used.

======== =============  ================================================================
Variable Long name      Description
======== =============  ================================================================
%u       %{user}        username
%n       %{username}    user part in user@domain, same as %u if there's no domain
%d       %{domain}      domain part in user@domain, empty if user there's no domain
======== =============  ================================================================

See :ref:`config_variables` for full list.

Example:

.. code-block:: none

   user_filter = (&(objectClass=posixAccount)(uid=%u))


.. _ldap_conf_ext_setting-userdb_warning_disable:

``userdb_warning_disable``
--------------------------

- Default: ``no``
- Values:  :ref:`boolean`

This setting is obsolete, and ignored regardless of the value being configured.


.. _ldap_conf_ext_setting-uris:

``uris``
--------

- Default: <empty>
- Values:  :ref:`string`

LDAP URIs to use.
Configure either this setting or :ref:`ldap_conf_ext_setting-hosts` to specify
what LDAP server(s) to conenct to.
Note that this setting isn't supported by all LDAP libraries.
The URIs are in syntax ``protocol://host:port``.

Example:

.. code-block:: none

   uris = ldaps://secure.domain.org

See also :ref:`ldap_conf_ext_setting-hosts`


