---
title: LDAP Intro
date: 2020-01-31 20:25:59
categories: Web
tags:
    - LDAP
    - Web
top:
---
# 1. What's LDAP?

LDAP stands for ***Lightweight Directory Access Protocol***. It is a lightweight client-server protocol for accessing directory services. It runs over **TCP/IP or other connection oriented transfer services**. 

# 2. What's directory? 

Similar to database, but contain more **descriptive, attribute-based information**. It has some features:

1. Read much more often than it is written. 
2. Directories are tuned to give quick response to high volume look up or search operations.
3. Have the ability to replicate information widely in order to increase availability and reliability


# 3. How does LDAP work? 

LDAP directory serice base on a client-server model. One or more LDAP servers contain the data making up the LDAP directory tree or LDAP backend database. An LDAP client connects to an LDAP server and asks it a question. The server responds with the answer, or with a pointer to where the client can get more information (typically, another LDAP server). No matter what LDAP server a client connects to, it sees the same view of the directory; a name presented to one LDAP server references the same entry it would at another LDAP server. This is an important feature of a global directory service, like LDAP.

# 4. Directory Structure

The protocol provides an interface with directories that follow the x.500 model:
+ An entry consists of a set of attributes
+ An attribute has a name(an attribute type or attribute description) and one or more values. Attrs are defined in a schema. 
+ Each entry has a unique identifier - distinguished Name(DN).This consists of its Relative Distinguished Name (RDN), constructed from some attribute(s) in the entry, followed by the parent entry's DN.

     dn: cn=John Doe,dc=example,dc=com
     cn: John Doe
     givenName: John
     sn: Doe
     telephoneNumber: +1 888 555 6789
     telephoneNumber: +1 888 555 1232
     mail: john@example.com
     manager: cn=Barbara Doe,dc=example,dc=com
     objectClass: inetOrgPerson
     objectClass: organizationalPerson
     objectClass: person
     objectClass: top
     
`"dn"` is the distinguished name of the entry; it is neither an attribute nor a part of the entry. `"cn=John Doe"` is the entry's RDN (Relative Distinguished Name), and `"dc=example,dc=com"` is the DN of the parent entry, where `"dc"` denotes `'Domain Component'`. The other lines show the attributes in the entry. Attribute names are typically mnemonic strings, like `"cn"` for common name, `"dc"` for domain component, `"mail"` for e-mail address, and `"sn"` for surname.

A server holds a subtree starting from a specific entry, e.g. `"dc=example,dc=com"` and its children. Servers may also hold references to other servers, so an attempt to access `"ou=department,dc=example,dc=com"` could return a referral or continuation reference to a server that holds that part of the directory tree. The client can then contact the other server. Some servers also support chaining, which means the server contacts the other server and returns the results to the client.

LDAP rarely defines any ordering: The server may return the values of an attribute, the attributes in an entry, and the entries found by a search operation in any order. This follows from the formal definitions - an entry is defined as a set of attributes, and an attribute is a set of values, and sets need not be ordered. 

# 5. Reference

1.[LDAP服务器的概念和原理简单介绍](https://segmentfault.com/a/1190000002607140)
2.[What's LDAP](https://www.tldp.org/HOWTO/LDAP-HOWTO/whatisldap.html)
3.[Wiki: Lightweight_Directory_Access_Protocol](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)
