# Book review - Learning Alfresco Web Scripts

(by Paolo Predonzani, originally posted on December 4, 2014)


["Learning Alfresco Web Scripts"](https://www.packtpub.com/web-development/learning-alfresco-web-scripts) by Ramesh Chauhan was published just a few
weeks ago. I bought it, quickly read
the interesting bits, then re-read it from cover to cover during some long
journeys by train over the past week.

Overall it is a good book so I though I'd write a review to help you judge if
it can be useful to you too for improving your understanding of Alfresco and for
your projects.

## Webscripts, webscripts, webscripts

So the book is about Alfresco webscripts. In Alfresco, webscripts are central in
many ways. Webscripts are involved in Share's presentation and its extensions,
in the generation of other custom UI's you may want to write, in the integration
of external systems with Alfresco and in the communication between Share and the
repository.

Webscripts are essentially Alfresco's flavour of restful web services. Alfresco
comes with many built-in webscripts and in your Alfresco projects you are likely
to develop new ones. Some webscripts output html, others generate json, atom
feeds or other formats, depending on the consumer's needs.

Any book on the subject is expected to guide the readers in the steps to set up
their first webscript. "Learning Alfresco Web Scripts" does a good job at
introducing the parts that make up a webscript (the descriptor, the Freemarker
template, the Javascript controller, the config xml file, the i18n bundles),
clearly explaining which are required and which are optional, and guiding in the
webscript installation and testing.

After the first tutorial pages, what is interesting is the direction the book
takes.

## What it means to be a webscript developer

In my opinion, the book's main goal is to show the reader what it means to be a 
webscript developer. It illustrates the fundamental knowledge, tools and
techniques that a developer has to master to become a good Alfresco developer.

Some of these include:

*   choosing a convenient webscript location for rapid development, deployment
    and testing
*   mappings URLs to webscripts
*   handling URL query/template parameters
*   producing output in different formats
*   writing controller implementations at increasing levels of complexity (from
    JavaScript to Java) 
*   reloading the webscripts 
*   listing the active webscipts
*   invoking/testing webscripts from the browser
*   testing with other REST clients (browser plugins, command line tools)
*   logging
*   debugging
*   organising webscript files in a structured maven project

If you've done Alfresco development before, all these items sound familiar and
you probably can remember a point in your career when you had to address an
issue with each of these point either to simply make a webscript work, or to use
the framework at its full potential.

If you've never done Alfresco development before, some items may sound new even
if you have previous experience with Java or web development. You may even think
that some points are trivial and question why they're worth discussing in a
book. But, trust the book, all these points have a specific meaning in Alfresco
and the book can help you progress through the learning curve.

## Beyond expectations

The book deals with the subjects at a technical level. For some subjects, at a
surprisingly deep technical level.

For example, there is a very interesting comparison between DeclarativeWebScript
and AbstractWebScript when writing Java-backed webscripts, as well as an
explanation of how a Java controller can coexist and interact with a JavaScript
controller.

Another example is the explanation of how to make a new "root object" available
to JavaScript webscripts.

Yet another example is the explanation of the internal wiring of the webscript
framework.

Apart from the specific examples, the interesting approach of the book is that
it often points to Alfresco's core configuration files and source code. It shows
the reader what the relevant files are, what sections to look at, and how things
work behind the scenes.

It's is a good way of teaching and learning by example. Instead of making up new
examples, the book points at Alfresco itself as a source of examples and
inspiration.

I personally like the approach. The power of open source software is the fact
that, if something is not clear, you can always look at the source code. It's a
bottom-up approach but it's helped me in several situations.

I think that the author found out how certain things work in Alfresco by looking
at the source code directly.

I also think that the author wants to make the reader independent and able to go
beyond the book (any book) and find the answers by himself, looking directly at
Alfresco's code.

In the chapters that deal with the basic webscript skills, these deeper details
are inserted here and there to keep the experienced reader motivated but without
distracting the newcomer too much from the main discussion.

On the other hand some chapters (specifically Ch. 3 Understanding the Web Script
Framework and Ch. 10 Extending the Web Script Framework) are probably targeted
at experienced readers.

## What's not in the book

The subject of webscripts is vast and a 180-pages book cannot cover them all.

Specifically what you won't find is a good discussion of the root objects and of
their APIs. Some root objects are mentioned and some ScriptNode examples are
provided, but in a real project you'll need to complement the book with other
documentation.

Also the book is slightly biased to repository webscripts. The subtitle, which
mentions "integration solutions for Alfresco", is a hint at this.

As a result, the book is abstracted from other contexts where webscripts can be
used. For example, webscripts are also at the core of Surf and Aikau, Share's
two UI frameworks. To write webscripts for these frameworks a lot of additional
knowledge is required, that the reader will have to find from other sources.

Again, the subject is broad and the book cannot be blamed for not covering all
possible details and applications.

## Conclusions

Overall this book covers the core webscript skills well and is appealing to both
beginners and experts. Beginners will find all the tutorials and introductory
material to get started. Experts will come back to this book for the depth of
the more technical topics.

Packt's website presents this as a book for beginners but I think that this can
only be interpreted in the sense that the book provides the foundations, but
practice and real projects are required to achieve real mastery.

## Other reviews

The book was also reviewed by others:

*   [Piergiorgio Lucidi](http://www.open4dev.com/journal/2014/10/31/book-contribution-learning-alfresco-web-scripts.html) (one of the book's technical reviewers)
*   [Martin Bergljung](http://www.marversolutions.com/wordpress/2015/01/21/book-review-learning-alfresco-web-scripts/)
