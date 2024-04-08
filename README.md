WebTeX
------

WebTeX is a simple servlet that creates PNG images for publishing on the web
given TeX expressions as input. It is used to show nicely formatted math
expressions on web pages and as a back end for a sort-of wysiwyg tinymce 
plugin math editor to make such images at a site at KTH, the Royal institute
of technology, Stockholm, Sweden, http://www.kth.se/.

Latest version of WebTex is available on github, http://github.com/KTHse/webtex.

You may also be interested in the TinyMCE front end plugin Tiny WebTex available
at http://github.com/KTHse/tinywebtex.

## API

Once the application is started, see Installation below, there is a fully working
demo page available at the root of the context (http://localhost:8080/webtex) 
which also contains brief documentation of the two available interfaces.
The interfaces are made up of the supplied JavaScript and the servlet interface.

### JavaScript

The included JavaScript webtex.js encapsulates a lot of the interaction
with the servlet and additionally provides for automatic vertical alignment
of formulas.

A full example is given below. You write the LaTeX expression as an alt attribute
prefixed by the string `tex:`. You notify the webtex.js script that it is 
a math image by adding `webtex` to the `class` attribute of the image.

Full example, paths have to be amended according to your configuation:
```
<html>
<head>
   ...
   <link rel="stylesheet" type="text/css" href=".../webtex/css/webtex.css"/>
   <script type="text/javascript" src=".../webtex/js/jquery.js"/>
   <script type="text/javascript" src=".../webtex/js/webtex.js"/>
   ...
</head>
<body>
   <p>
      <img class="webtex" alt="tex:a^2+b^2=c^2">
   </p>
</body>
</html>
```

NOTE: The only change from the original Mathtran behaviour is that the images 
are identified by the `webtex` class, rather than a non-existing `src` attribute. 
This makes it possible to use WebTex while still having validating pages, since
the `src` attribute is mandatory for `img` tags in validating pages.

### Servlet interface

The servlet request interface is identical to the original Mathtran interface. 
A couple of headers are added with more information about the generated image.

#### Request

HEAD and GET requests are supported. Both include the same headers but the 
HEAD request does not contain the image data (obviously).

`tex` A URLencoded LaTeX expression.

`D` An integer 1-10 specifying the size of the image.

NOTE: It is identical with the difference that the expressions are all expressions
supported by LaTeX with the mathtools package enabled, which should be a super
set of the expressions supported by the mathtran package this code was built to
replace.

#### Response

The servlet response contains non-standard headers with information about the 
generated picture, both in HEAD and GET requests.

`X-MathImage-tex` JavaScript encodeURIComponent-encoded string with the requested
TeX expression. This is currently equivalent to the `tex` request parameter but may
in a future release be a normalized version of the TeX expression.

`X-MathImage-log` JS encodeURIComponent-encoded string with error information or
'OK' if successful. 

`X-MathImage-depth` integer with base line offset information in approximate pixels.

`X-MathImage-width` integer with image width in pixels.

`X-MathImage-height` integer with image height in pixels.

Apart from these headers standard headers with cache control information are sent.

### Monitor

Not really a part of the interface, there are two pages at the URLs _about and 
_monitor providing version information and cache statistics respectively. This 
is an implementation of a minimal web application monitoring scheme we use. Use
them as you wish, but you should be aware that they are there. 

### Cache

The images are kept in a FIFO cache with a fixed size of 10000 entries. This may 
become more easily configured in a later version.

## Running

This application is now containerized, start with:
```docker run -p 8080:8080 --env-file environment.in kthse/webtex:latest```

Where the file environment can be created using the skeleton environment.in in the Git repository.

The container is based on the tomcat/9 container with it's options for ports and
configuration.

### SSL configuration

SSL is optional, unless both SSL variables are given, the tomcat SSL endpoint on 8443
will not be enabled.

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| SSL_CERTIFICATE_FILE | no |  | The path to a certificate PKCS12 keystore. |
| SSL_CERTIFICATE_KEY | no |  | The path to a file containing the passphrase for the certificate. |

## Rationale and history

### Mathtran

The servlet was originally intended to be an almost drop-in replacement for 
the Python script used by www.mathtran.org which is available as free source
under GPL.

Mathtran uses a pre-forked TeX binary that expressions are piped 
through. This was one way to increase speed of the service, but used only
one or a pool of TeX binaries. The produced images were not stored anywhere,
but recreated every time the image was requested. Since many requests
shared a TeX instance, the secsty format was necessary and made it difficult
to support newer constructs users normally use in documents. The service
also suffered a bit from somewhat weird behaviour of the TeX IPC mechanism.


### WebTeX

WebTex was written as a servlet dropping the IPC mechanism and simply
running TeX for each requests, but caching the output. In reality this proved
to be as fast or faster for our use cases and the new service was much more
reliable and less complicated. Each servlet thread runs its own LaTeX and 
multiple requests are handled by the servlet container threading up as necessary.

Since then, also the JavaScripts have been rewritten using jQuery and
optimized and very little of the original code is left (mostly some css).

The fact that LaTeX is not run in IPC mode means that we can get rid of the
TeX secsty format and the limitations it brings, and use a full LaTeX with
mathtools enabled providing many more math features.


## Contact

WebTex is created and maintained by the Infosys group <infosys@kth.se> 
at KTH, Kungliga tekniska högskolan, http://www.kth.se/.


## License and acknowledgements

The WebTex source is released under GPLv3, see COPYING.txt. The distribution
also contains a library, infosysutil.jar the source for which is available in
a separate repository https://github.com/KTHse/infosysutil

The WebTex service is inspired by and to some extent based on the Mathtran 
service, http://www.mathtran.org/, the source for which is available as 
free software under GPL at http://sourceforge.net/projects/mathtran/.

There is not much left of the original code by now apart from some css,
but the interfaces are much the same.
