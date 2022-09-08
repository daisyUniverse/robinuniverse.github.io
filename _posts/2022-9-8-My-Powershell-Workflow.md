---
title: My Powershell Scripting Workflow
subtitle: How I craft functional PS Toolchains at work
---

I started writing PowerShell about a year ago - Specifically for this job I got, when I took the interview, they had a kind of bench test, 
I had no idea to expect this, as it was an IT job and not a dev job, I was actually pleasantly surprised at first - but then I realized, oh!
I don't know powershell at all! ( the whole test was me googling powershell functions and stringing them together as if I were using very cursed bash )

I've come a long way since that bench test a year ago, and I would even go so far as to say that I'm becoming competant at it. I wrote
this originally as a kind of reminder for myself, to keep me on track, but I figured such a high level overview of my process may be useful to some

##  1. Concept
Come up with an idea for a script or project that you can see being useful - and remember to roughly consider scope!
For example, you may have an idea for a script that automatically, say, locks every computer on campus, but if you don't consider the scope
of such a project, you will likely waste a lot of time. Restrict your scope to something realistic and plan all the functions you would
like beforehand, ideally having a rough idea of how you might implemnent them

##  2. Experiment / Learning
Start just writing the quickest and dirtiest code you can to get from nowhere to doing most of what you want to do. If this requires the
use of some language or set of skills you have not yet learned - good news! you're about to learn them. Start small, work your way up, and
eventually you will have a rough prototype or draft of what you're trying to achieve

##  3. PS1 / Final Prototypes
This is the part where I would create a dedicated folder for any given project in this folder, and this is likely where a lot of projects will 
end up at, being functional enough to use, if a little cumbersome. Look at your first prototype, and clean up as much as you can, where it
makes sense to make something a function, do so, where you can clean up output to be less debug and more user friendly, do so. You should
end up with a fairly functional, fast, and easy to read 'Final Prototype'. It's perfectly understandable for projects to end here! if they
do the job you set out to do, and they do it well, and you don't need to frequently use them, this is a perfect stopping point

##  4. Modularization Prototyping
If you have done a particularly good job with this project, you might find yourself using it quite often, and perhaps even using it
frequently as components of other scripts, when your projects reach this point, you can benefit greatly from converting your project
from a standalone .PS1 file to a .PSM1, or a module. The structure of these are a little different than raw scripts, and they are quite
a lot more of a pain to debug, but that's why we start all projects as standalone scripts and from there we convert to modules.
Converting a project to a module has a lot of benefits, from allowing any user with the module imported to access it from anywhere, easily,
to being able to very easily incroperate it as part of other scripts. This is when a project enters the 'Module Testing' folder

##  5. Module Finalization
Once you are happy with the functionality of a module, you should consider letting it rest - Stamp a 'v1' on the project, and leave
it in the modules folder. At this point, you want to focus on cementing that modules behavior, and any tweaking you do from here on out
should not interfere with the existing functionality, so other scripters can use this module as part of their scripts and get the same
reliable output and behavior from now on.

##  6. Module Distribution
After you're finished with a module and you've got it where you want, it is a good idea to announce a release to your team so they can start to
experiment and play with what all they can do with it, currently, I have a simple script that simply copies the content of my modules folder to the users
local powershell modules folder, and this works well enough, but I would like to create a centralized network accessable repo for modules that any user
can import and instantly get up to date modules