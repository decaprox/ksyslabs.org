ksyslabs.org
============

This is a new version of ksyslabs.org

1. Install nanoc and dependencies:

	sudo gem install nanoc kramdown builder adsf

2. git clone <..>

3. add new page:

	nanoc ci new_page //new page will be create in /content/

4. Modify content/new_page.html.

5. Compile:

	nanoc compile

The output is a set of static files.

6. Run:

	nanoc view

You will see pages at http://localhost:3000/

7. Submit:

	git add new_page.html

8. Then submit a pull requests


Markdown:
===========

[Nice examples](http://dillinger.io)

Language hightligting:

~~~
printf("hello wrld\n");
~~~
{:.language-c}

language bash does not supported

## color ##
Put your data into the &lt;pre&gt; tag. To hightligt a line, use
~~~
<label style="color:blue">text</label>
~~~

To convert *diff* to html, it is convenient to use regex in VIM
~~~
76,102s/^-\(.*\)/\<label style="color:red"\>\1\<\/label\>/g
76,102s/^+\(.*\)/\<label style="color:blue"\>\1\<\/label\>/g
~~~
