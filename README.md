SuperDump
=========

*SuperDump* is a service for **_automated crash-dump analysis_**.

SuperDump was made with these goals in mind: 

 * Make crash-dump analysis easy for people who are unexperienced with it, or don't have the necessary tools installed.
 * Speed up first assessment of a crash-dump, by automatically preparing crash-dump analysis up-front. A developer should be quicker in determining if it's an already known crash.

What SuperDump is not: 

  * A replacement for in-depth analysis tools such as WinDbg.
  * A windows kernel dump analysis tool.

Features
========
 * Dump analysis can be triggered via web-frontend (HTTP-upload) or via REST-API.
 * Windows-crash-dumps (Fulldump or Minidump) can be analyzed (`.dmp` files). Only process-dumps, not kernel-dumps.
 * Linux-crach-dumps can be analyzed (`.core` files).
 * `.zip` files containing multiple crash-dumps are also supported. Each contained dump is processed.
 * Report results are stored as `.json` files and can be queried via REST-API. But they can also be viewed in SuperDump directly.
 * SuperDump report shows: 
   * Basic information (bitness, system/process uptime, lastevent, ...)
   * Loaded modules and versions
   * Stacktraces of all threads (native and .NET frames)
   * AppDomains
   * Basic memory analyis (number of bytes used by .NET types)
   * Linux Signals
 * SuperDump detects exceptions (native and managed) and marks the responsible threads.
 * Deadlock detection.
 * SuperDump also invokes a number of `WinDbg` commands automatically and logs them to a separate log-file.
 * It also invokes DebugDiag Analysis. An `.mht` file is created automatically and can be downloaded.
 * You can enter "interactive mode" for every dump. This will spin up `cdb.exe` (basically WinDbg for the command line) and create a websocket-based console terminal in the browser which lets you analyze the dump more deeply, with out the need to download it and have debugging tools installed locally. (Isn't that awesome?)
 * Linux coredumps (`.core`) are supported too. The analysis is triggered via a docker container (the actual command is configurable via  `LinuxAnalysisCommand`. Note, that linux dumps must be uploaded in archives in a specific format. In addition to the `.core` file, it must also contain linux system libraries as `libs.tar.gz`, otherwise symbols cannot be resolved correctly. If you're interested in seriously using this, please get in touch and we'll document this better. 
 * "Interactive mode" for linux coredumps is possible as well, but again it's just a command that is invoked (`LinuxInteractiveCommand`).
 * Slack Notifications for finished analysis (see `SlackNotificationUrls` config setting)
 * (Upcoming) Elastic search integration for statistics. Every dump analysis is pushed into elastic search instance, which allows to run statistics on crash dumps. (this is still in a branch: https://github.com/Dynatrace/superdump/tree/elasticsearch-support)
 
 
<a href="doc/img/mainpage.png"><img src="doc/img/mainpage.png" title="main page" width="200"/></a>
<a href="doc/img/managednativestacktrace.png"><img src="doc/img/managednativestacktrace.png" title="native managed"  width="200"/></a>
<a href="doc/img/nativeexception.png"><img src="doc/img/nativeexception.png" title="native exception" width="200"/></a>
<a href="doc/img/managedexception.png"><img src="doc/img/managedexception.png" title="managed exception" width="200"/></a>

Demo
============
Demo-Video: https://youtu.be/XdyDjkW8MDk

Slides about SuperDump (explaining some of the architecture): https://www.slideshare.net/ChristophNeumller/large-scale-crash-dump-analysis-with-superdump

Technologies
============
 * [CLRMD] for analysis.
 * [ASP.NET Core] and [Razor] for web-frontend and api.
 * [Hangfire] for task scheduling.
 * [websocket-manager] for in-browser terminal session for interactive WinDbg/Gdb session.
 * [Docker for Windows], [libunwind] and [gdb] for Linux analysis.
 
 [CLRMD]: https://github.com/Microsoft/clrmd
 [ASP.NET Core]: https://github.com/aspnet/Home
 [Razor]: https://github.com/aspnet/Razor
 [Hangfire]: https://github.com/HangfireIO/Hangfire
 [websocket-manager]: https://github.com/radu-matei/websocket-manager
 [Docker for Windows]: https://docs.docker.com/docker-for-windows/
 [gdb]: https://www.gnu.org/software/gdb/
 [libunwind]: http://www.nongnu.org/libunwind/
 
 <a href="doc/img/superdump-architecture.png"><img src="doc/img/superdump-architecture.png" title="managed exception" width="200"/></a>

Build
=====

 * Prerequisites:
   * Visual Studio 2017
   * .NET Core Tooling 1.0
   * .NET Core 1.1.1
   * .NET Framework 4.6
   * Docker for Windows (for building the docker image for linux analysis)
   * LocalDB (optional, see `UseInMemoryHangfireStorage` setting)
   * DebugDiag (for automatic DebugDiag analysis)
   * Windows Debugging Tools (`cdb.exe`) (optional, for interactive mode)
 * Build via `building/build.cmd`
 * Run via `build/runsuperdump.cmd` (defaults to port 5000)

State of the project
====================
SuperDump has been created at [Dynatrace] as an internship project in 2016. It turned out to be pretty useful so we thought it might be useful for others too. Thus we decided to opensource it.

Though it currently works great for us at Dynatrace, there are areas that need to be improved to make it a high-quality and generally useful tool:

 * Test-Coverage: A couple of unit tests are there, but there is currently no CI to automatically run them. The tests partially depend on actual dump-files being available, which obviously are not in source control. We'd need some binary-store, a prepare/download step, etc to make those run.
 * Some stuff is tailored for our needs at Dynatrace. E.g. we have special detection for Dynatrace Agent stackframes. While this feature probably won't hurt anyone else, it is kind of unclean to have such special detection in place.
  * There is no authentication/authorization implemented. Every crash-dump is visible to everyone and can be downloaded by everyone. This is an important fact, because crash-dump contents can be highly security critical.

Future
======
We've open sourced SuperDump, because we believe it can be helpful for others. Anyone is welcome to contribute to SuperDump. In small ways, or in ways we have not thought about yet. Feedback, github tickets, as well as PR's are welcome.

Some high-level ideas we've been poking around: 

 * _Pluggable analyzers:_ Possibility to write your own analyzers, detached from the main project and pluggable.
 * _Duplication Detection:_ Find a way to detect if the same crash has already been reported.
 * _Descriptive summaries:_ The idea is to put the most likely crash-reason in a short descriptive summary text. This is useful if a crash is entered as a bug in a ticket system.

Credit
======
Most of the initial code base was written by [Andreas Lobmaier] in his summer internship of 2016. It's been maintained and further developed since then by [Christoph Neumüller] and other folks at [Dynatrace]. [Dominik Steinbinder] also contributed large parts, such as Linux analysis, elastic search integration and much more. Thank you!

[Andreas Lobmaier]: https://github.com/alobmaier
[Christoph Neumüller]: https://github.com/discostu105
[Dominik Steinbinder]: https://github.com/dotstone
[Dynatrace]: https://www.dynatrace.com

License
=======
[MIT]

[MIT]: https://github.com/Dynatrace/superdump/blob/master/LICENSE
