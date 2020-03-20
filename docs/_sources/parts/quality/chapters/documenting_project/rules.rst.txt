1. The seven rules of technical writing
***************************************

Writing good documentation is easier in many aspects than writing code, but many
developers think otherwise. It will become easy once you start following a simple set of
rules regarding technical writing.

We are not talking here about writing a novel or poems, but a comprehensive piece of text
that can be used to understand software design, an API, or anything that makes up the
code base.

Every developer can produce such material, and this section provides the following seven
rules that can be applied in all cases:

- **Write in two steps**: Focus on ideas, and then on reviewing and shaping your text.
- **Target the readership**: Who is going to read it?
- **Use a simple style**: Keep it straight and simple. Use good grammar.
- **Limit the scope of the information**: Introduce one concept at a time.
- **Use realistic code examples**: Foos and bars should be avoided.
- **Use a light but sufficient approach**: You are not writing a book!
- **Use templates**: Help the readers get used to the common structure of your
  documents.

These rules are mostly inspired and adapted from "Agile Documentation: A Pattern Guide to
Producing Lightweight Documents for Software Projects, Wiley", a book by Andreas Rüping that
focuses on producing the best documentation in software projects.

1.1. Write in two steps
+++++++++++++++++++++++

Peter Elbow, in "Writing With Power: Techniques for Mastering the Writing Process, Oxford
University Press", explains that it is almost impossible for any human being to produce a
perfect text in one shot. The problem is that many developers write documentation and try
to directly come up with some perfect text. The only way they succeed in this exercise is by
stopping the writing after every two sentences to read them back, and do some corrections.
This means that they are focusing both on the content and the style of the text.

This is too hard for the brain, and the result is often not as good as it could be. A lot of time
and energy is spent on polishing the style and shape of the text before its meaning is
completely thought through.

Another approach is to drop the style and organization of the text and at first focus on its
content. All ideas are laid down on paper, no matter how they are written. You start to
write a continuous stream of thoughts and do not pause, even if you know that you
are making obvious grammatical mistakes, or know that what you just wrote may read
silly. At this stage, it does not matter if the sentences are barely understandable, as long as
the ideas are written down. You just write down what you want to say, and apply only
minimal structuring to your text.

By doing this, you focus only on what you want to say and will probably get more content
out of your mind than you would initially expect.

Another side effect of doing this *free writing* is that other ideas that are not directly related
to the topic will easily go through your mind. A good practice is to write them down on the
side as soon as they appear, so they are not lost, and then get back to the main writing.

The second step obviously consists of reading back the draft of your document and
polishing it so that it is comprehensible to everyone. Polishing a text means enhancing its
style, correcting mistakes, reorganizing it a bit, and removing any redundant information it
has.

A rule of thumb is that both steps should take an equal amount of time. If your time for
writing documentation is strictly limited, then plan it accordingly.

.. important:: Focus on the content first, and then on style and cleanliness.

1.2. Target the readership
++++++++++++++++++++++++++

When writing content, there is a simple, but important, question the writer should consider:
Who is going to read it?

This is not always obvious, as documentation is often written for every person that might
get and use the code. The reader can be anyone from a researcher who is looking for an
appropriate technical solution to their problem, or a developer who needs to implement a
new feature in the documented software.

Good documentation should follow a simple rule: each text should target one kind of
reader. This philosophy makes the writing easier, as you will precisely know what kind of
reader you're dealing with.

A good practice is to provide a small introductory document that explains in one sentence
what the documentation is about, and guides different readers to the appropriate parts of
documentation, for example:

    *"Atomisator is a product that fetches RSS feeds and saves them in a database, with a*
    *filtering process.*
    *If you are a developer, you might want to look at the API description (api.txt).*
    *If you are a manager, you can read the features list and the FAQ (features.txt).*
    *If you are a designer, you can read the architecture and infrastructure notes (arch.txt)."*

.. important:: Know your readership before you start to write.

1.3. Use a simple style
+++++++++++++++++++++++

Simple things are easier to understand. That's a fact.

By keeping sentences short and simple, your writing will require less cognitive effort for
their content to be extracted, processed, and then understood. Writing technical
documentation aims to provide a software guide to readers. It is not a fiction book, and
should be closer to your microwave operation manual than to a Dickens novel.

The following are a few tips to keep in mind:

- Use short sentences. They should be no longer than 100–120 characters (including
  spaces). This is the length of two lines in a typical paperback.
- Each paragraph should be composed of three to four sentences at most, which
  express one main idea. Let your text breathe.
- Don't repeat yourself too much. Avoid journalistic styles where ideas are
  repeated again and again to make sure they are understood.
- Don't use several tenses. The present tense is enough most of the time.
- Do not make jokes in the text if you are not a really fine writer. Being funny in a
  technical book is really hard, and few writers master it. If you really want to
  distill some humor, keep it in code examples and you will be fine.

.. important:: You are not writing fiction; keep the style as simple as possible.

1.4. Limit the scope of information
+++++++++++++++++++++++++++++++++++

There's a simple sign of bad documentation in software: you cannot find specific
information in it, even if you're sure that it is there. After spending some time reading the
table of contents, you are starting to search through text files using ``grep`` with several word
combinations and still cannot find what you are looking for. But you're sure the
information is there because you saw it once.

This often happens when writers do not organize their texts well with meaningful titles and
headings. They might provide tons of information, but it won't be useful if the reader is not
able to scan through all the documentation for a specific topic.

In a good document, paragraphs should be gathered under a meaningful heading for a
given section, and the document title should synthesize the content in a short phrase. A
table of contents could be made of all the sections' titles, in order to help the reader scan
through the document.

A simple yet effective practice to compose your titles and headings is to
ask yourself, "What phrase would I type in Google to find this section?"

1.5. Use realistic code examples
++++++++++++++++++++++++++++++++

Unrealistic code examples simply make your documentation harder to understand.

For instance, if you have to provide some string literals, the Foos and bars are really bad
choices. If you have to show your reader how to use your code, why not to use a real-world
example? A common practice is to make sure that each code example can be cut and pasted
into a real program.

To show an example of bad usage, let's assume we want to show how to use the
``parse()`` function from the ``atomisator`` project, which aims to parse RSS feeds. Here is the
usage example using an unrealistic imaginary source:

.. code-block:: python

    >>> from atomisator.parser import parse
    >>> # Let's use it:
    >>> stuff = parse('some-feed.xml')
    >>> next(stuff)
    {'title': 'foo', 'content': 'blabla'}

A better example, such as the following, would be using a data source that looks like a
valid URL to RSS feed and shows output that resembles the real article:

.. code-block:: python

    >>> from atomisator.parser import parse
    >>> # Let's use it:
    >>> my_feed = parse('http://tarekziade.wordpress.com/feed')
    >>> next(my_feed)
    {'title': 'eight tips to start with python', 'content': 'The first tip is..., ...'}

This slight difference might sound like overkill but, in fact, makes your documentation a lot
more useful. A reader can copy those lines into a shell, understand that ``parse()`` expects a
URL as a parameter, and that it returns an iterator that contains web articles.

Of course, giving a realistic example is not always possible or viable. This is especially true
for very generic code. Anyway, you should always strive to reduce the amount of such unrealistic
examples to a minimum.

.. important:: Code examples should be directly reusable in real programs.

1.6. Use a light but sufficient approach
++++++++++++++++++++++++++++++++++++++++

In most agile methodologies, documentation is not the first citizen. Making software that
just works is more important than the detailed documentation. So, a good practice, as Scott
Ambler explains in his book "Agile Modeling: Effective Practices for eXtreme Programming and
the Unified Process, John Wiley & Sons", is to define the real documentation needs, rather than
try to document everything possible.

For instance, let's look at some example documentation of a simple project that is available
on GitHub. ``ianitor`` (available at `<https://github.com/ClearcodeH1/ianitor>`_) is a tool
that helps to register processes in the Consul service discovery cluster, and it is mostly
aimed at system administrators. If you take a look at its documentation, you will realize
that this is just a single document (the ``README.md`` file). It explains only how it works and
how to use it. From the administrator's perspective, this is sufficient. They only need to
know how to configure and run the tool, and there is no other group of people expected to
use ``ianitor``. This document limits its scope by answering one question, "How do I use
ianitor on my server?"

1.7. Use templates
++++++++++++++++++

Many pages on Wikipedia look similar. There are boxes on the right-hand side that are
used to summarize some information for documents belonging to the same area. The first
section of the article usually contains a table of contents with links that refer to anchors in
the same text. There is always a reference section at the end.

Users get used to it. For instance, they know they can have a quick look at the table of
contents, and if they do not find the information they are looking for, they will go directly
to the reference section to see if they can find another website on the topic. This works for
any page on Wikipedia. Once you learn the format of Wikipedia articles, you become more
efficient in finding useful information.

So, using templates forces a common pattern for documents, and therefore enables more
efficient searching for information. Users get used to the common structure of information
and know how to read it quickly.

Providing a template for each kind of document also provides a quick start for writers.