Changes in this fork:
----------

Added functions compatible with Thinbus-SRP for u,M,K calculations and
a generator of Thinbus safe prime config file.
Usage is the same as with original pysrp but with a few differences:

In pysrp the dataflow expected during authentication is:

    Client -> Server: username, A
    Server -> Client: s, B
    Client - > Server: M
    Server - > Client : H(A,M,K)

While in Thinbus:

    Client -> Server: username
    Server -> Client: s, B
    Client - > Server: M, A
    Server -> Client: H(A,M,K)

So you have to provide some string instead of A on server side at 
first initialisation of `srp.Verifier` object. Examples will be added 
later.


pysrp
=====
Tom Cocagne &lt;tom.cocagne@gmail.com&gt;

pysrp provides a Python implementation of the [Secure Remote Password
protocol](http://srp.stanford.edu/) (SRP).


SRP Overview
------------

SRP is a cryptographically strong authentication
protocol for password-based, mutual authentication over an insecure
network connection.

Unlike other common challenge-response autentication protocols, such
as Kereros and SSL, SRP does not rely on an external infrastructure
of trusted key servers or certificate management. Instead, SRP server
applications use verification keys derived from each user's password
to determine the authenticity of a network connection.

SRP provides mutual-authentication in that successful authentication
requires both sides of the connection to have knowledge of the
user's password. If the client side lacks the user's password or the
server side lacks the proper verification key, the authentication will
fail.

Unlike SSL, SRP does not directly encrypt all data flowing through
the authenticated connection. However, successful authentication does
result in a cryptographically strong shared key that can be used
for symmetric-key encryption.

For a full description of the pysrp package and the SRP protocol, please refer
to the [pysrp documentation](http://pythonhosted.org/srp/)


Usage Example
-------------

```python
import srp
    
# The salt and verifier returned from srp.create_salted_verification_key() should be
# stored on the server.
salt, vkey = srp.create_salted_verification_key( 'testuser', 'testpassword' )

class AuthenticationFailed (Exception):
    pass
   
# ~~~ Begin Authentication ~~~
    
usr      = srp.User( 'testuser', 'testpassword' )
uname, A = usr.start_authentication()
    
# The authentication process can fail at each step from this
# point on. To comply with the SRP protocol, the authentication
# process should be aborted on the first failure.
    
# Client => Server: username, A
svr      = srp.Verifier( uname, salt, vkey, A )
s,B      = svr.get_challenge()

if s is None or B is None:
    raise AuthenticationFailed()
    
# Server => Client: s, B
M        = usr.process_challenge( s, B )

if M is None:
    raise AuthenticationFailed()
    
# Client => Server: M
HAMK     = svr.verify_session( M )

if HAMK is None:
    raise AuthenticationFailed()
    
# Server => Client: HAMK
usr.verify_session( HAMK )
    
# At this point the authentication process is complete.
 
assert usr.authenticated()
assert svr.authenticated()
```

Installation
------------

```
$ pip install srp
```

Implementation
--------------

It consists of 3 modules: A pure Python implementation, A ctypes +
OpenSSL implementation, and a C extension module. The ctypes &
extension modules are approximately 10-20x faster than the pure Python
implementation and can take advantage of multiple CPUs. The extension
module will be used if available, otherwise the library will fall back
to the ctypes implementation followed by the pure Python
implementation.

Note: The test_srp.py script prints the performance timings for each
combination of hash algorithm and prime number size. This may be of
use in deciding which pair of parameters to use in the unlikely
event that the defaults are unacceptable.

Installation from source:
```
   $ python setup.py install
```

Documentation:
```
   $ cd srp/doc
   $ sphinx-build -b html . <desired output directory>
```
      
Validity & Performance Testing:
```
   $ python setup.py build
   $ python test_srp.py
```   
