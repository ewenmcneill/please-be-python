# Please Be Python

## Usage

To use this, copy and paste the majority of the Bourne Shell / Python
polyglot [please-be-python](please-be-python) into the top of your Python
script (before any other Python content), up to the point where it says
"Python code begins here" (which can be your Python code).

Make sure your program starts with:

    #! /bin/sh

You will probably also want to comment out or delete the "`echo`"
statements of where Python was found once you are confident that
it is working.

The default as-supplied Python code just prints out the Python version found.

## Caveats

You can *not* use "compiler hints" like:

    from __future__ import print_function 

with this work around to find Python, as they have to be at the top
of the file, before any Python code; and this polyglot relies on
injecting some "fake Python" at the top to hide the shell script
prefix.  (It may be possible to change this, but it would involve
finding a way to hide this from the initial shell script interpreter.
Note that you can still use simple `print(...)` calls with Python
2 without that future import, it is just that some extended function
parameters will no work.)

Syntax highlighters and linters will probably be confused by this polyglot.


## Background

The Python language developers [decided to make Python 3 incompatible
with Python 2](https://www.python.org/dev/peps/pep-3000/), and [stop
developing Python 2](https://www.python.org/doc/sunset-python-2/).

This lead to [considerable discussion of the future of
`/usr/bin/python`](https://www.python.org/dev/peps/pep-0394/).
Ultimately several Linux distributions decided *not to ship
`/usr/bin/python` at all* (by default), apparently on the rationale
that *no `/usr/bin/python`* was better than the "wrong Python version".
(They were wrong about that, but we get to live with the mess for the
next 10 years :-( )

In particular [RedHat Enterprise Linux 8 does not ship with
`/usr/bin/python`](https://developers.redhat.com/blog/2019/05/07/what-no-python-in-red-hat-enterprise-linux-8),
and in fact what might be considered the "system Python" is well
hidden away (in `/usr/libexec/platform-python`; eg that is what
Ansible uses).  [Ubuntu 20.04 ships `/usr/bin/python3` by default,
but not
`/usr/bin/python`](https://wiki.ubuntu.com/FocalFossa/ReleaseNotes#Python3_by_default).

All of which means that the days of starting a Python script with:

    #! /usr/bin/python

or even:

    #! /usr/bin/env python

and expecting it to run in a "reasonably useful" Python interpreter
are behind us.

On "desktops" or "application servers" it might be reasonable to expect
that some steps have been taken to install a reasonable Python that is
reachable in a reasonable location.

But for *systems programming* especially around system setup (eg,
discovery steps around Ansible automation for instance) it's basically
essential to rely only on things that are installed on the OS by
default (this is the whole reason that standards like POSIX, Linux
Standards Base, etc were created).  And the decisions of the Python
language developers and the Linux distributions effectively remove
Python -- or at least `/usr/bin/python` -- from the "always there"
base of things that one can rely on :-/


## How it works

As initially run, the `#! /bin/sh` line causes the Bourne shell interpreter
to parse the polyglot.  From here, the very first declaration is both
valid Python and valid shell script, but the interpretation is different.
In particular the Python `"""` multi-line syntax is used to start an extended
Python comment, to hide the shell script section.  For the Bourne shell
that triple double-quote is interpreted as two separate strings, one empty
(`""`) and one spanning to the next line (`"` / `"`).  After that, the
program text is all shell script, until the quoting gets back in sync between
the two languages, after the `# End of shell script, start of shared code`
line.

In the shell script, we use `command -v` to probe for *likely* locations
of a Python interpreter, and if found then we `exec` that Python interpreter
over the current script with the current arguments, and it is very similar
to as if we had run, eg, `/usr/bin/python3 PROGRAM ARGS` in the beginning.

If the `command -v` of likely locations fails, we try more desperate
measures like looking for other "known to be written in Python"
programs and copying how they run Python, or just guessing locations
where Linux distributions have been known to hide System Python.

This also relies on the fact that the Bourne shell interprets the file it
is running line by line, so that if it never reaches the Python section
of the file, it is never confused by those lines.

(Also note that `command -v` is used to probe for programs, rather
than `which` because [Debian deprecated
`which`](https://lwn.net/Articles/874049/), so `which` too is no longer
portal.  Sigh..)


## Is this a good idea?

No.

Nothing mentioned in this README was a good idea.


## Why did you write this then?

Desperation.

A succession of bad ideas (Python 3000, no `/usr/bin/python`) leads to
another compensating bad idea.

In particular I have Python code that has to be Python, which needs to
run on "out of the box" RedHat Enterprise Linux 8 and Ubuntu 20.04 :-(


## Should I use this in production?

Probably not.

If you have any chance of installing Python in a predictable location
on all your systems, I would strongly suggest you do that instead.

In addition it almost certainly does not cover all the possible ways that
the Python language developers and the Linux distributions have conspired
to make systems Python usage difficult; so if it does not work for you, then
you might need to further extend the desperation with which it tries to
find a Python interpreter for you.


## License

This code is licensed under the [0BSD](https://opensource.org/licenses/0BSD)
license:

    Permission to use, copy, modify, and/or distribute this software
    for any purpose with or without fee is hereby granted.

    THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL
    WARRANTIES WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED
    WARRANTIES OF MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE
    AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL
    DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA
    OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER
    TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR
    PERFORMANCE OF THIS SOFTWARE.
