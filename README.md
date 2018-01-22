# Implementation Plan

Use the VS code request / reply / event schema over the Jupyter channels. We’ll start sending the events over the io_pub channel and the request/events over the shell channel. We’re not 100% sure that there’s a perfect match here, but it looks pretty close, and the VSC debugger protocol serves as a good multi-lingual starting point.

From a UI / interactions standpoint:

We’ll start off with the VS model of having a single debug UI which spawns JupyterLab. That is there’ll be a current active kernel/process, with threads, and a current call stack. This means one set of tool windows which will control the debugger instead of multiple tool windows embedded within each notebook window.

We’ve broken out some high level steps, some of them are smaller than others:

## IPython kernel -> source mapping & code identity

This is presumably a couple of small changes to the existing IPython kernel to ensure that code is identified by the hash of the code. 
It’s also ensuring that the code will be accessible to debuggers after it has been executed.
The hash changes are the minimal viable set of changes here – they could allow the debug client to associate code that it’s read from a notebook file which would allow the user to set breakpoints and step.

## pydevd kernel-aware

This change will make pydevd be able to understand Jupyter kernel code. The minimal viable set of changes here, which could potentially be none, would be returning the hash of the code to the client. If that’s in the code’s filename that would just fall out. 
A more elaborate set of changes would allow the client to download the code for when it’s otherwise not available (e.g. a cell has been modified after it was executed).
This is similar in scope to the work that allows the debugger to be aware of Django template debugging.

## Attach to Process Integration

Technically orthogonal, but allows us to quickly see how debugging with the IPython kernel is working early on. This will enable debug->attach to process to a running kernel, downloading of the code running inside of the kernel, and allow setting breakpoints, stepping, etc… It relies upon the source mapping changes and having pydevd to be aware of kernel’s.

## IPython kernel / debugger integration

This change will involve updating the IPython kernel to load the debugger extension and configure it to send the debug protocol over the Jupyter ZMQ channels. This will build on the work to support VSC protocol for pydevd.

## JupyterLab UI

Finally the UI needs to be built inside of JupyterLab which will enable to user to control the debug sessions. This includes switching processes / kernels, switching threads, switching call stacks, inspecting locals, evaluating custom expressions, setting breakpoints, stepping in/out/over, breaking into a running process, etc…

It also includes consuming all of the VSC protocol, sending messages, receiving them, etc…
