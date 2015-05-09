---
layout: post
title: Debugging Gulp.js Tasks
categories:
    - Development
---

If you're like me, most of your Gulp.js code is pretty straightforward and doesn't have too much going on that you can't easily grok. Occasionally though, you write something that is a bit more complicated and the need to debug comes about. Until recently, I've just stared at the code until I see what's wrong. This gets pretty frustrating and is very inefficient.

Even though I've worked with the [node-inspector](https://github.com/node-inspector/node-inspector) project before on some Node.js applications I've built, I never quite managed to put it together that it could be used on Gulp,js, a Node.js application. It says it quite clearly how to do it in the project's README, but that part either didn't exist last time I checked or I glossed over it too quickly!

The first thing to do is make sure node-inspector is installed globally on your system:

{% highlight bash %}
[18:36:54] gg:project $ npm install -g node-inspector
... Some boring stuff about node-inspector installing ...
[18:37:57] gg:project $
{% endhighlight %}

**NOTE #1**: Just like most globally installed npm packages on OS X, you might run into some permissions issues installing. I just went ahead and used sudo (`sudo npm install -g node-inspector`) to do it, but if this makes you queasy, you can read about some other options over in the article, [sudo npm install -g is an antipattern](https://jesse.sh/sudo-npm-install/).

Second, start up the node-debugger:

{% highlight bash %}
[18:37:57] gg:project $ node-debug $(which gulp) myTask

Node Inspector is now available from http://127.0.0.1:8080/debug?port=5858
Debugging `/usr/local/opt/nvm/v0.10.33/bin/gulp`

debugger listening on port 5858
{% endhighlight %}

**NOTE #2**: If you're on Windows, you'll need to use the full path to where the gulp binary is installed instead of the handy `$(which gulp)`, which uses the output of the `which` command in a subshell to get the full path to the gulp binary. I don't have a Windows system around to test with, but some more information seemed to be on the README for the [node-inspector](https://github.com/node-inspector/node-inspector) project.

This will automatically open up the node-inspector in your web browser (which works just like Chrome's regular Web Developer Tools debugger) and pause execution of the Gulp script before it has started processing your gulpfile.js. You can go find your gulpfile.js in the navigator on the left, set a breakpoint, hit "Start," and do your debugging magic.

It can be a little messy trying to find the gulpfile.js in the sources panel sometimes, so if you're having trouble finding it (or just aren't real comfortable yet with how the debugger works), `Ctrl+C` in the Terminal window where you ran `node-debug` to stop Gulp and node-inspector. You can then go add a `debugger` line to your Gulpfile where you want to stop to check things out and restart `node-debug`. Example:

{% highlight javascript %}
gulp.task('myTask', function() {
  debugger;
  gulp
    .src('myPath/files/**/*')
    .pipe(someAction())
    .pipe(gulp.dest('newPath/files'));
});
{% endhighlight %}

You could also put the debugger at the top of your gulpfile if you were having trouble with a require statement or something.

Happy debugging!
