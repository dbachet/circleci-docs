---

title: Interacting with the browser on CircleCI's VM
layout: doc
tags:
  - troubleshooting
  - browsers

---
## Integration Tests

Integration tests can be hard to debug, especially when they're running on a remote machine.
There are four good ways to debug browser tests on CircleCI.

## Screenshots and Artifacts 

At the end of a build on CircleCI, we will gather up all [build artifacts](/docs/build-artifacts)
and make them available from your build. This allows you to save screenshots as part of your build,
and then view them when the build finishes.

Saving screenshots is straightforward: it's a built-in feature in WebKit and Selenium, and supported by most test suites:

*   [Manually, using Selenium directly](http://docs.seleniumhq.org/docs/04_webdriver_advanced.jsp#remotewebdriver)
*   [Automaticaly on failure, using Cucumber](https://github.com/mattheworiordan/capybara-screenshot)
*   [Automaticaly on failure, using Behat and Mink](https://gist.github.com/michalochman/3175175)

To make this work with build artifacts, you need to save the screenshot to the
`$CIRCLE_ARTIFACTS` directory.

## Using a browser on your dev machine to access HTTP server on CircleCI

If you are running a test that runs a HTTP server on CircleCI, sometimes it can
be helpful to use a browser running on your local machine to debug why a
particular test is failing. Setting this up is easy with an SSH-enabled
build.

First, you should run an SSH build. You will be shown the command to log into
the build container over SSH. This command will look like this:

<pre>
ssh -p 64625 ubuntu@54.221.135.43
</pre>

We want to add port-forwarding to the command, which we do with the `-L` flag.
We want to specify a local port and remote port. In this example we will forward
requests to `http://localhost:8080` to port `3000` on the CircleCI container.
This would be useful if your build runs a debug Ruby on Rails app, which listens
on port 3000 for example.

<pre>
ssh -p 64625 ubuntu@54.221.135.43 -L 8080:localhost:3000
</pre>

You can now open your browser on your local machine and navigate to
`http://localhost:8080` and this will send requests directly to the server
running on port `3000` on the CircleCI container. You can now manually start the
test server on the CircleCI container if it is not aleady running, and you
should be able to access from the browser on you development machine.

This is a very easy way to debug things when setting up Selenium tests, for
example.

## Interact with the browser over VNC

## Spawning your own X Server

VNC allows you to view and interact with the browser that is running your tests. This will only work if you're using a driver that runs a real browser. You will be able to interact with a browser that Selenium controls, but phantomjs is headless -- there is nothing to interact with.

Before you start, make sure you have a VNC viewer installed. If you're using OSX, I recommend
[Chicken of the VNC](http://sourceforge.net/projects/chicken/).
[RealVNC](http://www.realvnc.com/download/viewer/) is also available on most platforms.

First, [start an SSH build](/docs/ssh-build)
to a CircleCI VM. When you connect to the machine, add the -L flag and forward the remote port 5901 to the local port 5902:

<pre>
daniel@mymac$ ssh -p PORT ubuntu@IP_ADDRESS -L 5902:localhost:5901
</pre>

You should be connected to the Circle VM, now start the VNC server:

<pre>
ubuntu@box159:~$ vnc4server -geometry 1280x1024 -depth 24
</pre>

Enter the password `password` when it prompts you for a password. Your connection is secured with SSH, so there is no need for a strong password. You do need to enter a password to start the VNC server.

Start your VNC viewer and connect to `localhost:5902`, enter the password you entered when it prompts you for a password. You should see a display containing a terminal window. You can ignore any warnings about an insecure or unencrypted connection. Your connection is secured through the SSH tunnel.

Next, make sure to run:

<pre>
ubuntu@box159:~$ export DISPLAY=:1.0
</pre>

to ensure that windows open in the VNC server, rather than the default headless X server.

Now you can run your integration tests from the command line and watch the browser for unexpected behavior. You can even interact with the browser &mdash; it's as if the tests were running on your local machine!

### Sharing Circle's X Server

If you find yourself setting up a VNC server often, then you might want to automate the process. You can use x11vnc to attach a VNC server to X.

Download [`x11vnc`](http://www.karlrunge.com/x11vnc/index.html) and start it before your tests:

<pre>
dependencies:
  post:
    - sudo apt-get install -y x11vnc
    - x11vnc -forever -nopw:
        background: true
</pre>

Now when you [start an SSH build](/docs/ssh-build), you'll be able to connect to the VNC server while your default test steps run. You can either use a VNC viewer that is capable of SSH tunneling, or set up a tunnel on your own:

<pre>
$ ssh -p PORT ubuntu@IP_ADDRESS -L 5900:localhost:5900
</pre>

## X11 forwarding over SSH

CircleCI also supports X11 forwarding over SSH. X11 forwarding is similar to VNC -- you can interact with the browser running on CircleCI from your local machine.

Before you start, make sure you have an X Window System on your computer. If you're using OSX, I recommend
[XQuartz](http://xquartz.macosforge.org/landing/).

With X set up on your system, [start an SSH build](/docs/ssh-build)
to a CircleCI VM, using the `-X` flag to set up forwarding:

<pre>
daniel@mymac$ ssh -X -p PORT ubuntu@IP_ADDRESS
</pre>

This will start an SSH session with X11 forwarding enabled.
To connect your VM's display to your machine, set the display environment variable to localhost:10.0

<pre>
ubuntu@box10$ export DISPLAY=localhost:10.0
</pre>

Check that everything is working by starting xclock.

<pre>
ubuntu@box10$ xclock
</pre>

You can kill xclock with Ctrl+c after it appears on your desktop.

Now you can run your integration tests from the command line and watch the browser for unexpected behavior. You can even interact with the browser &mdash; it's as if the tests were running on your local machine!

### VNC Viewer Recommendations

Some of our customers have had some VNC clients perform poorly and
others perform well.  Particuarly, on OS X RealVNC produces a better
image than Chicken of the VNC.

If you have had a good or bad experience with a VNC viewer,
[let us know](mailto:sayhi@circleci.com) so we can update this page and help
other customers.

## Troubleshooting

If you find that VNC or X11 disconnects unexpectedly, your build container may be running out of memory. See [our guide to memory limits](https://circleci.com/docs/oom) to learn more.
