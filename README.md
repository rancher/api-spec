# What is this thing? #
This document defines the REST API specification implemented by public [Rancher](http://rancher.io) APIs.

Our goal is to make APIs that are as easy to use as possible.  Each service has documentation available, but this should be a supplement, not required reading.  Armed with just the URL and credentials, a user should be able to navigate their way through the API in a web browser to learn about what resources and operations it has.  In other words, the API should be _discoverable_.  Once you're familiar with what's available, requests can be made with simple tools like cURL or any basic HTTP request library.

# The Specification #
The full [technical specification](./specification.md) contains the complete details that drive the development of [clients and libraries](#what-if-i-just-want-to-use-one-of-your-apis) which handle many nuances automatically so you can worry about what you want to do and not catering to the specification.

# What if I just want to use one of your APIs? #
See the documentation for the specific API you are interested in.  There are clients for various languages available.

# Why are you publishing it? #
Maybe you'd like to build your own tools that work with our services.  Or maybe you want to build your own service that follows the same style as ours.  We'd love it if you did either one and want to give you all the information you need.  This is also the same documentation our internal teams use to build their APIs.

# Contact #
For questions, comments, corrections, suggestions, etc., open an issue in [rancher/rancher](//github.com/rancher/rancher/issues) with a title starting with `[api-spec] `.

Or just [click here](//github.com/rancher/rancher/issues/new?title=%5Bapi-spec%5D%20) to create a new issue.

# License #
Copyright &copy; 2012-2014 [Go Daddy Operating Company, LLC ](http://godaddy.com)

Copyright &copy; 2014-2015 [Rancher Labs, Inc.](http://rancher.com)

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
