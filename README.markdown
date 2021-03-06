FakeFtp
=======

This is a gem that allows you to test FTP implementations in ruby. It is a minimal single-client FTP server
that can be bound to any arbitrary port on localhost.

Why?
----

We want to ensure that our code works, in a way that is agnostic to the implementation used (unlike with stubs or mocks).

How
---

FakeFtp is a simple FTP server that fakes out enough of the protocol to get us by, allowing us to test that files get to
their intended destination rather than testing how our code does so.

Usage
-----

To test passive upload:

    require 'fake_ftp'
    require 'net/ftp'

    server = FakeFtp::Server.new(21212, 21213)
    ## 21212 is the control port, which is used by FTP for the primary connection
    ## 21213 is the data port, used in FTP passive mode to send file contents
    server.start

    ftp = Net::FTP.new
    ftp.connect('127.0.0.1', 21212)
    ftp.login('user', 'password')
    ftp.passive = true
    ftp.put('some_file.txt')
    ftp.close

    server.files.should include('some_file.txt')
    server.file('some_file.txt').bytes.should == 25
    server.file('some_file.txt').should be_passive
    server.file('some_file.txt').should_not be_active

    server.stop

To test active upload:

    server = FakeFtp::Server.new(21212)
    ## 21212 is the control port, which is used by FTP for the primary connection
    ## 21213 is the data port, used in FTP passive mode to send file contents
    server.start

    ftp = Net::FTP.new
    ftp.connect('127.0.0.1', 21212)
    ftp.login('user', 'password')
    ftp.passive = false
    ftp.put('some_file.txt')
    ftp.close

    server.files.should include('some_file.txt')
    server.file('some_file.txt').bytes.should == 25
    server.file('some_file.txt').should be_active
    server.file('some_file.txt').should_not be_passive

    server.stop

Note that many FTP clients default to active, unless specified otherwise.

References
----------

* http://rubyforge.org/projects/ftpd/ - a simple ftp daemon written by Chris Wanstrath
* http://ruby-doc.org/stdlib/libdoc/gserver/rdoc/index.html - a generic server in the Ruby standard library, by John W Small

Contributors
------------

* Eric Saxby
* Colin Shield
* liehann (https://github.com/liehann)

License
-------

The MIT License

Copyright (c) 2011 Eric Saxby

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
