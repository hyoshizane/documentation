.. _authentication-ldap_authentication:

=========================
LDAP Authentication
=========================

See :ref:`LDAP <authentication-ldap>` for more details.

.. code-block:: none

  passdb {
    args = /etc/dovecot/dovecot-ldap.conf.ext
    driver = ldap
  }
  userdb {
    driver = prefetch
  }
  userdb {
    args = /etc/dovecot/dovecot-ldap.conf.ext
    driver = ldap
  }

These enable ``LDAP``to be used as ``passdb`` and ``userdb``. The userdb
prefetch allows ``IMAP`` or ``POP3`` logins to do only a single LDAP lookup by
returning the userdb information already in the passdb lookup.
:ref:`authentication-prefetch_userdb` has more details on the prefetch
userdb.

For those setting parameters available in the ldap conf.ext (i.e,
/etc/dovecot/dovecot-ldap.conf.ext shown in the above example), 
see :ref:`authentication-ldap_conf_ext_settings` 
