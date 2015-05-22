# Paradigm
A framework for object-centric modular web development.

## Goals

* Free web developers to think, design, and code _simple objects_ that can be combined like puzzle pieces. Not web pages.
* In so doing, to simplify the approach to responsive web design while simultaneously making complex responsive sites easier to design, create, and manage.

## Request handling, basic overview

Paradigm will handle HTTP(S) requests.

* Layer 1: DBAL. Input: request object. Output: data from the datastore, ideally identified only by URI (though other items in the request object such as cookies, POST data, etc. may be used).

* Layer 2: Template Model Mapper. Input: data from the datastore. Output: the same data in the Paradigm Template Model (PaTMo) standard object format.

* Layer 3: Template Renderer. Input: PaTMo object. Output: the same data rendered on the templates specified in the PaTMo.

* Layer 4: Template Finalizer. Input: template-rendered data. Output: template-rendered data with final house-keeping rules applied.

* Layer 5: Responder. Input: finalized template-rendered data. Output: response object suitable for responding to the client with this data.

## Details

### DBAL

Implemented by the user. Provide an example.

### Template Model Mapper

Implemented by the user. Provide an example.

### Template Renderer

Parse the PaTMo:

1. Rendering data into the PaTMo-specified template
2. Recursively render children

### PaTMo

A flexible JSON model of a template.

* A single JSON object
* Any property name other than '\_children' and '\_template' may be used to define data and metadata properties of the template. Your template can render these values simply by refering to them directly by name.
* '\_template' is a string that identifies (by filename) the template that the template renderer should use to render the response.",
* '\_children' should be expected by your templates to be an ordered array specifying the child data (usually other template objects) to be rendered in the parent template.
  * Simple strings, numbers, and other JS primitives may be children of a template object.
  * Or another template may be included simply by including a JSON object with the '\_template' and '\_children' properties set.

#### Example

    {
        "_template": "page.html",
        "title": "A sample HTML page for a blog post containing a podcast.",
        "author": "Jon Snow",
        "navigation": {
            "comment": "Apparently this template has navigation which can support highlighting a selection.",
            "selectedItem": "/podcasts"
        },
        "_children": [
            "Introductory text for this post.",
            {
                "_template": "podcast.html",
                "title": "Title of a podcast.",
                "audioUrl": "https://example.com/link-to-podcast-audio.ogg",
                "_children": [ ]
            }
        ]
    }

### Templates

Templates MUST:

1. Ignore the PaTMo \_template value.
2. Ignore out-of-scope PaTMo values. Stay in your scope.
3. Loop through the \_children array in whatever order is deemed appropriate.
4. Support 0 to inifinitely many values in the \_children array.
5. Safely render the template data not found in \_children no matter what value is found there. Make no assumptions.
6. Be written in such a way that a request could be fulfilled according to spec using only that template, no matter how minor the template.

#### HTML Templates

I'm trying to make this thing not HTML specific, but let's be honest, compared to XML, JSON, etc. HTML has a lot of weird rules and exceptions.

Thus, to fulfill item 6 above, HTML templates MUST:

1. Be wrapped in an <html></html> tag. It's HTML, so declare it.
2. Include their own CSS style declarations in their own <head></head> tag.
3. Include their own JavaScript code in their own <script></script> tag.
4. ONLY include JavaScript that executes inside an event handler.

These rules make it very easy to write modular HTML templates that:

* can be served alone as the response to a request,
* free developers to code and design objects that can be seamlessly combined,
* can improve client-side performance via significant reduction of HTTP requests.

The Template Finalizer will do a little magic to clean up the obvious problems that arise from combining templates.

### Template Finalizer

Based on the request object, a Template Finalizer is chosen to do last-minute cleanup of the response
rendered by the Renderer. This should usually be chosen based on response type: html, json, xml, etc.

#### HTML Finalizer

The Paradigm HTML Finalizer will:

* Collapse all <html></html> elements into one by effectively removing all but the first <html> and all but the last </html>.
* Collapse all <head></head> elements into one, maintaining child order, placing it directly before the <body> tag in the at html top. The additional values already specified by a tag in a parent template for any tags in the head will be dropped except for <style> tags
* Collapse all <script></script> tags into one, maintaining child order, minifying the code and placing it as the last element in the <body> tag.
* Minify and gzip the resulting HTML.

