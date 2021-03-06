---
layout: biwascheme
title: Sequential Programming
---
<div id="term"></div>

<div id="testarea" style="background-color:lightgrey; padding:20px; margin: 30px 0 30px 0;">
  Test Area 1: Simple Sequentially Programmed Web App
  <div>
    <button id="button1" style="height:60px; width:100px;">Button #1</button>
    <button id="button2" style="height:60px; width:100px;">Button #2</button>
    <div id="div1"
	 style="float:left; height:30px; width:80px; background-color:yellow; padding:15px;">
      Div #1
    </div>
    <div id="div2"
	 style="float:left; height:30px; width:80px; background-color:pink; padding:15px;">
      Div #2
    </div>
  </div>
</div>
<div id="testarea2" style="height:auto; overflow:auto; width:200px; background-color:lightgrey; padding:20px; margin: 30px 0 30px 0;">
  Test Area 2: Calculator Web App, Programmed Sequentially<br/>
</div>

<h1>Sequentially Programmed Web App Demo</h1>

This brief introduction to sequentially programmed web apps
includes a live demonstration of such an app
to show how the programming is done.
Basically "sequentially programmed" means the program's structure mirrors
its execution flow.
The introduction then goes on to explain the motivation for writing
programs in this manner.

<h2>Try It Now</h2>

Enter the following code into the BiwaScheme console above.
(Always terminate code in the console with a RETURN.)

{% highlight lisp %}

(load "mini-framework.scm")
(define (test)
  (with-handlers ((click-handler "#div1")
                  (click-handler "#div2")
                  (keydown-handler "#button1")
                  (keydown-handler "#button2")
                  (timeout-handler test-timeout 10000))
                 (display (get-input))
                 (display (get-input))
                 (display (get-input))
                 (display (get-input))
                 (display (get-input)))
  (display "Test finished."))
{% endhighlight %}

The code defines a Scheme function named ```test```.
Now whenever you enter ```(reset (test))``` into the console the test will run.
Try it: enter ```(reset (test))``` into the console.
```with-handlers``` sets up two click handlers, two keydown handlers,
and a timeout handler.
Until one of those events occurs, the program will not proceed through
the first ```get-input```.
When any of the events occurs ```get-input``` returns
the event's data, ```display``` prints the type of event,
and the program pauses until one of the handlers is triggered again,
and so forth, until all five calls to ```get-input``` have returned.
Then the program prints "Test finished."

To try a more complicated web app, whose code nevertheless still looks
quite simple, enter the following code into the BiwaScheme console above.

{% highlight lisp %}

(load "calculator.scm")

{% endhighlight %}

The app is a simple calculator with addition, subtraction, multiplication,
and division. Here is its main loop:

{% highlight lisp %}

  (with-handlers ((click-handler ".button"))
    (let* ((btn (js-ref (second (get-input)) "target"))
           (text (js-ref btn "innerText")))
      (case text
        (("+" "-" "*" "/")
         (when (not (= value2 0))
           (set! value1 value2))
         (set! value2 0)
         (set! op (string->symbol text)))
        (("=")
         (when op
           (set! value1 ((eval op) value1 value2))
           (set! value2 0)))
        (else
         (set! value2 (+ (* value2 10) (string->number text)))))
      (if op
          (format #t "~a ~a ~a~%" value1 (symbol->string op) value2)
          (format #t "~a~%" value2))))
{% endhighlight %}

<h2>What Is Sequential Programming?</h2>

Sequential programming of a web application is radically different
from the dominant style of web-application programming, but it should
be remembered that the dominant style of programming for many (probably most)
*offline* applications is sequential. The sequential style of programming is
suitable for a wide range of applications and application programmers
because it models a program after the stepwise logic used to complete a
program's goal. The sequential style of programming is the easiest style to
master, all things being equal, because it straightforwardly mirrors our
step-by-step thinking about what the program must do to fulfil its purpose.
It is interesting to explore the possibility of writing web applications
in a sequential style because the dominant, non-sequential styles
make it difficult to follow execution flow through a program's code.
This difficulty complicates program development, testing, and refactoring.
The discussion thread
["Node.js - A giant step backwards?"](https://news.ycombinator.com/item?id=3510758)
presents some of the issues a programmer faces when he writes
a relatively new kind of non-sequential web application,
namely a web application written for an event-driven web server.

This introduction to sequentially programmed web apps
briefly explains how asynchronous events are handled both
by a traditional web application and by a
web application written for an event-driven web server (*ala* Node.js).
Then it points out a working example of an application written for a
continuation-based web server and online resources for learning important
things about such applications.
Finally it presents a sequentially programmed, continuation-based
web application that runs entirely in the web browser--a single-page
web application.

<h2>How Asynchronous Events Are Handled in Web Applications</h2>

Any program, simple or complex, that uses I/O (user I/O, disk reads,
disk writes, or transfers of data between computers) must have a means of
synchronizing itself with the completion of that I/O. For desktop applications,
the operating system provides system calls, a scheduler, and library functions
that allow the programmer to structure his program to mirror
the program's execution flow. For desktop applications, the waiting
for completion of I/O can usually be neatly hidden within some function
like read(), write(), getchar(), etc., but in most web applications written
up to 2020, this mirroring is not possible, due to the stateless nature of
the web. Not even a web application written to run entirely in the
web browser (a single-page web application) can mirror program flow,
due to the event-driven nature of JavaScript in the web browser.

In 2020, web servers can be classified as event-driven (like Node.js) and
non-event-driven (traditional web servers). An event-driven web server
can react to many asynchronous events in real time, without holding up
the main event loop.

Web applications written for an event-driven web server and typical
single-page web applications are structured using callbacks, deferreds,
promises, or some form of continuation-passing style. Thus, their structure
cannot mirror their flow. However, a web application written using <em>true</em>
continuations <em>can</em> be structured to mirror its flow.

## Traditional Web Applications vs. Web Applications Built upon Server-Side Web Continuations

Very often, web applications interact with the user by building request
pages that pass program state information from web page to web page in
cookies or hidden form fields, something like this Racket Scheme code:

{% highlight lisp %}

(define (sum query)
  (build-request-page "First number:" "/one" ""))

(define (one query)
  (build-request-page "Second number"
                      "/two"
                      (cdr (assq 'number query))))

(define (two query)
  (let ([n (string->number (cdr (assq 'hidden query)))]
	[m (string->number (cdr (assq 'number query)))])
    `(html (body "The sum is " ,(number->string (+ m n))))))

(hash-set! dispatch-table "sum" sum)
(hash-set! dispatch-table "one" one)
(hash-set! dispatch-table "two" two)

{% endhighlight %}

Don't worry if you can't understand all of that.
Get the gist of it: program flow is from page to page and
each page handles a particular event, the input of a number by the user.
That is the typical, traditional programming style of a web application.
Such a style is more complicated and unwieldy than
a direct, straightforward style employing server-side web continuations:

{% highlight lisp %}

(define (sum2 query)
  (define m (get-number "First number:"))
  (define n (get-number "Second number:"))
  `(html (body "The sum is " ,(number->string (+ m n)))))

{% endhighlight %}

Don't worry if you can't understand that yet.
The main thing is to understand that it works,
that the programming of the solution at the top level is perfectly direct,
and that client-side web continuations even further simplify the programming.

We will describe client-side web continuations in the next section, but
for the curious, we now describe the functions ```sum2``` and ```get-number```.
The funtion ```sum2``` gets dispatched when the user visits
```http://localhost:8080/sum2```.
The function ```get-number``` does something remarkable,
something only possible in Scheme language:
it sends a response to the user's request of a page like
```http://localhost:8080/sum2``` or
```http://localhost:8080/k1578504361599.834?number=73&hidden=&enter=Enter```
that looks to the user like this:
<form action="/k1578504381390.28" method="get">First number:
  <input type="text" name="number" value="">
  <input type="hidden" name="hidden" value="">
  <input type="submit" name="enter" value="Enter">
</form>
or
<form action="/k1578504381390.28" method="get">Second number:
  <input type="text" name="number" value="">
  <input type="hidden" name="hidden" value="">
  <input type="submit" name="enter" value="Enter">
</form>
then it *saves its place and suspends program execution until 
the user submits a reply*,
upon which a new connection is created to respond to the submission.
The ```get-number``` function then *revives program execution
exactly where it left off*.
It converts the submitted number to a string and returns that,
whereupon it is stored in a variable (```m``` or ```n``` in the above case).
``` `html``` is a template that creates and serves a web page.

Both the web server code and both versions of the application code are
fully described in the section
[Continuations](https://docs.racket-lang.org/more/#%28part._.Continuations%29")
of the page
[More: Systems Programming with Racket](https://docs.racket-lang.org/more/#%28part._.Continuations%29")

The reader is strongly encouraged to
[download Racket](https://download.racket-lang.org/),
load the
[the finished Racket Scheme code](https://docs.racket-lang.org/more/step9.txt)
and run the code—a five- to ten-minute exercise.
The code can be loaded and run in Racket something like this (where 8080 is
the port number to which the server responds and &quot;step9.txt&quot; is the full or
relative pathname of the Racket Scheme code):

{% highlight shell %}

$ racket
Welcome to Racket v7.5.
> (enter! "step9.txt")
"step9.txt"> (serve 8080)
#<procedure:...webcon/step9.txt:17:2>
"step9.txt">

{% endhighlight %}

(```$``` and ```>``` are prompts.)
After starting the program, you can use the web application locally
by typing <a href="http://localhost:8080/sum2">http://localhost:8080/sum2</a>
into your web browser's address bar.
The program asks for and waits for one number, then jumps to a second web page,
where it asks for and waits for a second number, then jumps to a third web page,
where it sums the two
numbers. Along the way it stores continuations to remember its each halt
after serving a page, even saving the first and second numbers.
In case the user presses his browser's back button once or twice anywhere
along the way, or retypes the URL of the second or first page, the program
recalls its state when a number was typed into the respective page,
and shows the number and again in its input form,
just as the user originally typed it, and the user can change the number
or accept it and continue the program as before.

Section 5.2 of Christian Queinnec's paper
<a href="https://pages.lip6.fr/Christian.Queinnec/PDF/www.pdf">'Inverting back the inversion of control or, Continuations versus page-centric programming'</a>
describes a very similar web application but does not detail its implementation.

## Client-Side Web Continuations

Server-side web continuations are interesting because they simplify
the creation of web applications. However, the amount of memory they consume
easily becomes a problem. 
To overcome this problem,
it would be convenient to put the continuations in the browser,
This way, the memory burden of 10,000 or 1,000,000
simultaneous users is not placed on the web server,
but instead spread across all the web browsers visiting the website.
This brief introduction to sequentially programmed web apps
demonstrates a way to create a web application using
client-side (in-the-browser) continuations.
The demo incorporates
[BiwaScheme](https://github.com/biwascheme/biwascheme),
a Scheme language implementation in JavaScript.
See [Further Reading](reading.html).

The core of the test program is just this:

{% highlight lisp %}
(with-handlers ((click-handler "#div1")
                (click-handler "#div2")
                (keydown-handler "#button1")
                (keydown-handler "#button2")
                (timeout-handler test-timeout 10000))
               (display (get-input))
               (display (get-input))
               (display (get-input))
               (display (get-input))
               (display (get-input)))
{% endhighlight %}

The macro ```with-handlers``` sets up any number of
event handlers (```click```, ```mousedown```, ```mouseup```, ```mouseover```,
```keydown```, ```keyup```, ```timout```, etc.) and removes them
when execution exits its block.
The function ```get-input``` sets up a continuation that returns
execution to that point only after an event has occurred.
The invocation of ```with-handlers``` in our test program
sets up five event handlers:

1. A handler triggered by a click on the ```div``` element having the ID
```div1```,

1. a handler triggered by a click on the ```div``` element having the ID
```div2```,

1. a handler triggered by a keypress when the keyboard input focus is on the
```button``` element having the ID ```button1```,

1. a handler triggered by a keypress when the keyboard input focus is on the
```button``` element having the ID ```button2```,

1. a handler triggered after 10 seconds if no other event is triggered first.

To set the focus on an element such as a button, you can click on that element.
