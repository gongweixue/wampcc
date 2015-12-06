C++ abstraction library for JSON
================================

There already exists a plethora of JSON implementations for C/C++, so why the
need for another?

A downside of having so many JSON libraries available is that the one chosen for
your next project might become obsolete, perhaps abandoned by its community of
support or superseded by others.

If that happens, application code will need to be modified to introduce a
replacement JSON library. This sort of re-factoring is typically neither an easy
or quick task, specifically because the JSON object model of the deprecated
library will typically have found is way through-out many parts of application
code, making it difficult to extract and replace.

This is where jalson comes in.  The idea is that it can be used to wrap the
public interface of another JSON library (the *implementation*), allowing
application code to be decoupled and isolated from the data model of the
library.  This then allows the parsing and generation features of the
implementation library to be use while preventing its API, in terms of its
public classes and function, from getting exposed and mixed in to the end
application.

Then if the JSON implementation needs to be swapped out for another, it can be
done so entirely with in the jalson layer, and so hugely reducing the disruption
for the end application code.

Goals
-----

So jalson is not a JOSN parser / generator like other JSON libraries.  Instead
it offers an object model and API for building and working with JSON data; and
aims to abstract away the underling details of the parsing and text generation
(which is delegated to the implementation library).

In addition, jalson is designed along the following guidelines:

* STL based --  JSON containers types (Object and Array) are represented using to the obvious C++
  counterparts, std::map & std::vector. Similary std::string is used for JSON String.

* intuitive unsurprising interface -- no attempt is made to cleverly replicate
  python or JQuery syntax in C++ code, which can lead to code constructs
  unfamiliar to C++ developers; i.e. adherence to the principle of least
  astonishment.

* minimally complete -- aim to avoid unnecessary bells & whistles, while
  providing just about the most basic interface required for working with JSON
  data.  The motivation is to simplify integration of jalson into end user code,
  so by aiming to be minimal, there is less for application code to rely upon.

Supported Implementations
-------------------------

Currently jalson comes with support for the C library `jansson` [http://www.digip.org/jansson/].

Example
-------

This example shows basic usage of jalson for encoding, decoding and accessing
JSON values for reading and writing. The code, together with a basic makefile,
can be found under `examples/hello`. Note that when this example is built, it
must be linked to both the jalson library (`libjalson`) *and* the vendor library
that provides the JSON parsing & generation:

```C++
/* Basic example of use jalson */

#include <jalson/jalson.h>

#include <iostream>

int main(int, char**)
{
  // obtain details about the JSON implementation wrapped inside jalson

  jalson::vendor_details details;
  jalson::get_vendor_details(&details);

  std::cout << "JSON implementation: "
            << details.vendor << " "
            << details.major_version << "."
            << details.minor_version << "."
            << details.micro_version << "\n";

  // --- Build a JSON object ---

  jalson::json_value v = jalson::decode("[\"hello\", {}, 2015]");

  // --- Add some items ---

  // using methods of the stl container
  v.as_array().push_back("world");
  v.as_array().push_back(1);
  v.as_array().push_back(true);
  v.as_array().push_back( jalson::json_object() );
  v.as_array().push_back( jalson::json_array() );

  // using helper methods of json_value type
  v[1]["vendor"]  = details.vendor;
  v[1]["version"] = details.major_version;

  // Print
  std::cout << v << "\n";
  return 0;
}
```


Source Configuration & Build
----------------------------

Jalson uses the GNU Autotools for its builds system, i.e., the makefiles are
generated by running the `configure` script.  The configure script is not
present in the repository sources, but it can be generated by running a helper
script found in the root directory of the source, `autotools_setup.sh`.  An
additional script `autotools_clean.sh` can be run to clean up the autotools
files which get created.

When running the `configure` script, jalson must be told which JSON
implementation it will work with.  In this example, the jalson source is
configured to use a version of the `jansson` library which has previously been
built and installed at a particular location:

    configure --prefix=/opt/jalson-1.0 --with-jansson=/opt/jansson-2.7

Failure to specify a vendor implementation will lead to configure error:

    configure: error: You have not configured a JSON implement ion vendor.

If all goes well, the project makefiles will be generated, and jalson can be
built and installed by running :

    make
    make install
