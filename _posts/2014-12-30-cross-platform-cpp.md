---
layout: post
title: Cross-platform C++
tags: [c++]
---

I like developing in linux.
The ecosystem is nice, tools cooperate together nicely,
  and the command line is a joy.
However, my main computer and my friends' computers tend to be Windows computers.
If I write programs, I want them to run anywhere.

Furthermore, I'm lazy.
I don't want to switch to a different computer or open a virtual machine
  with a separate environment and compile there.
I want to compile all versions from within the linux environment,
  from a single build script.

For this, I turn to `scons`.
`scons` is a build tool, similar to `make`, with the build script written in python.
Most importantly, it is easy to set up many related build targets,
  one for each target platform.

`scons` works by setting up an environment, then building targets based on that environment.
Therefore, to set up a cross-compiling target, we need to first set up appropriate environments.
This is the backbone of the `SConstruct` file, which is the equivalent of a `Makefile`.

```python
win32 = Environment()
win64 = Environment()
linux = Environment()

#Define what will be the working directory
win32['SYS'] = 'win32'
win64['SYS'] = 'win64'
linux['SYS'] = 'linux'

#Define the compilers, using C++11
win32.Replace(CC='i686-w64-mingw32-gcc')
win32.Replace(CXX='i686-w64-mingw32-g++')
win32.Append(CXXFLAGS=['-std=c++0x'])
win64.Replace(CC='x86_64-w64-mingw32-gcc')
win64.Replace(CXX='x86_64-w64-mingw32-g++')
win64.Append(CXXFLAGS=['-std=c++0x'])
linux.Append(CXXFLAGS=['-std=c++11'])

#Define the appropriate file formats
win32.Replace(SHLIBPREFIX='')
win32.Replace(SHLIBSUFFIX='.dll')
win32.Replace(PROGSUFFIX='.exe')
win32.Append(LINKFLAGS='-static')
win64.Replace(SHLIBPREFIX='')
win64.Replace(SHLIBSUFFIX='.dll')
win64.Replace(PROGSUFFIX='.exe')
win64.Append(LINKFLAGS='-static')
```

The different compilers specified are those provided by [mingw-w64](http://mingw-w64.sourceforge.net).
These produce Windows binaries when run on a linux environment.
In Ubuntu, these can be installed with `sudo apt-get install g++-mingw-w64-i686 g++-mingw-w64-x86-64`.

After this, any usual `scons` command such as `linux.Program('main.cc')`.
However, that would require us to repeat the build commands for each target,
  which would not be desirable.
Instead, let's make a separate script, which will be called for each environment.
This goes into a file named `SConscript`, in the base directory.
This is a simple example, which compiles a program from `Program.cc`
  and any `*.cc` files found in the `src` directory.

```python
Import('env')
env.Append(CPPPATH=['include'])
exe = env.Program(['Program.cc',Glob('src/*.cc')])
Return('exe')
```

Now, to call this from the main file.

```python
for env in [win32,win64,linux]:
    build_dir = os.path.join('build',env['SYS'])
    exe = SConscript('SConscript',
                     variant_dir=build_dir,
                     src_dir='.',
                     exports=['env'])
    inst_dir = env['SYS']
    env.Install(inst_dir,exe)
    Clean('.',inst_dir)
```

The key thing is the `variant_dir` argument,
  which causes each target to be built in a separate directory.

An example is shown in one of my github repositories,
  [here](https://github.com/Lunderberg/minimal_cross-platform).
It also contains a few extra niceties,
  such as adding static resources to each of the final build folders.