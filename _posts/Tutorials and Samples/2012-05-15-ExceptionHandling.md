---
layout: article
title: Exception Handling
categories: [photon-cloud, photon-server, tutorials, references]
tags: [how-to, sample, overview]
---
{% include globals %}

## Photon 2.6 and before:

With Photon 3.0 the exception handling went through a complete redesign.
In previous versions the strategy was to catch all exceptions inside the
Photon Framework (photonsocketserver.dll) and log them through a logging
facade. This design had several drawbacks:

-   **unhandled exception event handler not being called**: the expected
    CLR default behavior is when exceptions are not handled by the
    developers code to be notified on the unhandled exception event
    handler.
-   **exception handling dependency on logging**: developers had to
    implement a logging facade, failing to do so could lead to "lost"
    exceptions.
-   **only one behavior for not handled exceptions**: exceptions not
    handled by the developer where caught by photon, logged and then
    ignored. Effectively working in a similar way the "runtime backstop"
    in .Net 1.x did.
-   **inconsistent behavior**: unhandled exceptions in threads
    controlled by the developer raised the unhandled exceptions handler
    and terminated the process

## New with Photon 3.0:

-   **unhandled exception event handler is always called**: exceptions
    not handled by the developers code will always raise the unhandled
    exception event handler.
-   **non intrusive logging**: we encourage developers to implement the
    unhandled exceptions handler, but independently of the developers
    code unhandled exceptions are logged by photon. For details see
    "Logging" section below.
-   **three policies for unhandled exceptions**: Photon supports one of
    three policies when an unhandled exeption is raised - ignore,
    restart application, end process
-   **consistent behavior**:

There are a couple of cases where the exception handling policy might be
overridden by the CLR e.g. ThreadAbortException won't necessarily
restart or end the process - see [Exceptions in Managed
Threads](http://msdn.microsoft.com/en-us/library/ms228965.aspx), the
StackOverflowException can't be ignored - see [StackOverflowException
Class](http://msdn.microsoft.com/en-us/library/system.stackoverflowexception.aspx).
For best practices on exception throwing and handling developers please
refer to the MSDN documentation [Handling and Throwing
Exceptions](http://msdn.microsoft.com/en-us/library/5b2yeyab(v=VS.100).aspx)

Note: the Application Compatibility Flag
(legacyUnhandledExceptionPolicy) described in [Exceptions in Managed
Threads](http://msdn.microsoft.com/en-us/library/ms228965.aspx) isn't
supported by Photon on a per Application config. This mode is set per
instance (UnhandledExceptionPolicy = "Ignore") - see "Configuring the
UnhandledException Policy" section below.

## Suggestions for the usage of the UnhandledExceptionPolicies

Microsoft introduced the "Terminate Process" as new default policy with
.Net 2.0 with following statement quoted from [Exceptions in Managed
Threads](http://msdn.microsoft.com/en-us/library/ms228965.aspx): **

When threads are allowed to fail silently, without terminating the
application, serious programming problems can go undetected. This is a
particular problem for services and other applications which run for
extended periods. As threads fail, program state gradually becomes
corrupted. Application performance may degrade, or the application might
hang.

Allowing unhandled exceptions in threads to proceed naturally, until the
operating system terminates the program, exposes such problems during
development and testing. Error reports on program terminations support
debugging.

But depending on the stage or situation your project is (in development,
in QA, LIVE without known issues, etc) you might consider using one of
the different policies supported by Photon:

-   **Restart Application**
-   **Ignore**
-   **Terminate Process**

While developing setting "**Terminate Process**" launches the
debugger/visual studio on process termination. Are you handling with a
multithreaded issue you might prefer to keep the process running and the
application loaded then setting the policy "**Ignore**" would be what
you prefer.

During a QA Phase either during the run of stress tests or manual
testing using "**Terminate Process**" ensures you won't get flooded by
errors that stem from the root-error.

Live Stable - assuming your service is stable a policy you can consider
using is the "**Restart Application**". In the rare event of an
unhandled exception the application would be restarted by Photon
minimising the risks - this is the fastest way for the sytem to be up
again. Faster than setup Photon as a Service with "Terminate Process" as
policy and the auto-restart windows services feature. Note: you should
monitor your service and log files in case this happens more often.

Live with a known issue - if the system shows an unexpected exception
quite often and you need time to fix - depending on the exception it
might be possible to keep a reasonably stable system by setting the
policy to "**Ignore**".

Note on StackOverflows: usually the stacktrace loged during the
unhandled exception will point you to the fix. The
StackOverflowException is a case where the stacktrace can get lost if
the unhandled exception policy is set to "**Ignore**" or
"**RestartApplication**".

Note on Debugging Core: for the rare cases where the Photon core raises
an unexpected exception we recommend setting "ProduceDump" is always set
for details see Photon Core Debugging" - sending us the generated dump
files and logs will help us to fix the issue.

## Logging Unhandled Exceptions

As depicted in the figure below an unhandled exception event is raised
first in the context of the custom application (1) where the developer
can log the exception in any way he requires to c) - please also refer
to sample code Lite, LiteLobby, etc. In a second step the CLR then
raises the event in the default application domain - which in the case
of Photon is the PhotonHostRuntime. Because of this Photon is able to
log the exception in b).

<figure>
<img src="{{ IMG }}/ExceptionHandling-1.png" />
<figcaption>Logging of Unhandled Exeptions</figcaption>
</figure>

Photon Loggingfile default path/filenames

-   [\$PhotonBaseDir]**\\bin\_**[\$OS]**\\log\\Photon-**[\$InstanceName]-[\$date]**.log**
-   [\$PhotonBaseDir]**\\bin\_**[\$OS]**\\log\\PhotonCLR.log**
-   [\$PhotonBaseDir]**\\log\\**[\$ApplicationName]**.log**

## Configuring the UnhandledException Policy

~~~~ {.code}
        
            <Runtime
              Assembly="PhotonHostRuntime, Culture=neutral"
              Type="PhotonHostRuntime.PhotonDomainManager"
              
              UnhandledExceptionPolicy="Ignore"
        
    
~~~~

UnhandledExceptionPolicy can be: ReloadAppDomain, Ignore or
TerminateProcess

## Photon Core Debugging

~~~~ {.code}
        
            <!-- Instance settings -->
            <Instance1
                ...
                ProduceDumps = "TRUE"
                DumpType = "Full"
                MaxDumpsToProduce = "2"
                ...
        
    
~~~~

ProduceDumps can be: TRUE or FALSE DumpType: Full, Maxi or Mini
