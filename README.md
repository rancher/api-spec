# What is this thing? #
This document defines the REST API specification implemented by public Go Daddy&reg; APIs.

Our goal is to make APIs that are as easy to use as possible.  Each service has documentation available, but this should be a supplement, not required reading.  Armed with just the URL and credentials, a user should be able to navigate their way through the API in a web browser to learn about what resources and operations it has.  In other words, the API should be _discoverable_.  Once you're familiar with what's available, requests can be made with simple tools like cURL or any basic HTTP request library.

# The Specification #
The full [technical specification](./specification.md) contains the complete details that drive the development of [clients and libraries](#what-if-i-just-want-to-use-one-of-your-apis) which handle many nuances automatically so you can worry about what you want to do and not catering to the specification.

# Common Use Cases #
Several use cases require a bit more guidance but are not appropriate for inclusion into the technical specification but are worth review.  These can be found in the [scenarios guide](./scenarios.md) and establish a set of best practices of common situations while remaining compliant with the specification.

# Why are you publishing it? #
Maybe you'd like to build your own tools that work with our services.  Or maybe you want to build your own service that follows the same style as ours.  We'd love it if you did either one and want to give you all the information you need.  This is also the same documentation our internal teams use to build their APIs.

# What if I just want to use one of your APIs? #
See the documentation for the specific API you are interested in.  There are clients for various languages available:
  - [PHP](https://github.com/godaddy/gdapi-php)
  - [C#](https://github.com/godaddy/gdapi-csharp)
  - Python &mdash; coming soon
  - Command Line &mdash; coming soon
  - Node.js &mdash; coming soon
  - Java &mdash; coming soon

# Why not use other RESTful API Specifications? #
This is a good question, but difficult to answer.  In short, we needed a specification to ensure a common experience across all of our services that was general enough to create clients and libraries that implemented the boiler plate code allowing information to be consumed or exposed consistently and more rapidly.

Until a true standard emerges for RESTful API's we will continue to use and evolve this specification with the help of industry best practices and core concepts.  This specification may not apply for every API out there, but it works for us and hope it will for you.

# Contact #
For questions, comments, corrections, suggestions, etc., open an [issue](https://github.com/godaddy/gdapi/issues) or email us at [apispec@godaddy.com](mailto:apispec@godaddy.com).  

Please use our normal [support page](http://support.godaddy.com/) for questions or problems about a specific API, product, or account.

# License #
Copyright &copy; 2012-2013 Go Daddy Operating Company, LLC 

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

