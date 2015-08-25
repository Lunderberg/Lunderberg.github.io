---
layout: post
title: C++ Makefile, Pretty Output
tags: [c++, make]
---

All the previous posts about makefiles have focused on flexibility and ease of use.
This time, we will be improving the readability of the output of the makefile.

By default, make will print each command as it is being run.
For short commands, this is useful.
However, once a compilation command starts having all sorts of library flags,
  it becomes less useful.
Often, it can even be difficult to tell which file is being built.
Therefore, it is better to replace the default messages with a custom message.

Let's start by looking at our rule to compile a single file.

```make
build/%.o: %.cc
	mkdir -p $(@D)
	$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $< -o $@
```

We can silence the output of the default printing by adding a `@` before the command.

```make
build/%.o: %.cc
	@mkdir -p $(@D)
	@$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $< -o $@
```

Now, we have no output at all.
Time to add in some print statements.

```make
build/%.o: %.cc
	@mkdir -p $(@D)
	@printf "Compiling $@\n"
	@$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $< -o $@
```

Very nice.
It will now only print out any
Note that any output directly printed by the program is still printed to screen,
  despite the `@`.
We wouldn't want to suppress the error messages from the compiler, after all.

Most terminals support colored output using ANSI escape sequences.
Let's define a few variables for ease.

```make
COM_COLOR   = \033[0;34m
OBJ_COLOR   = \033[0;36m
OK_COLOR    = \033[0;32m
ERROR_COLOR = \033[0;31m
WARN_COLOR  = \033[0;33m
NO_COLOR    = \033[m

OK_STRING    = "[OK]"
ERROR_STRING = "[ERROR]"
WARN_STRING  = "[WARNING]"
COM_STRING   = "Compiling"
```

Now, using those variables to pretty up the output.

```make
build/%.o: %.cc
	@mkdir -p $(@D)
	@printf "%b" "$(COM_COLOR)$(COM_STRING) $(OBJ_COLOR)$(@)$(NO_COLOR)\n";
	@$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $< -o $@
```

Note the use of the `"%b"` format specifier, rather than the more familiar `"%s"`.
In the ANSI color sequences,
  we are using an escape sequence `\033` to specify a non-printing ASCII.
The `"%b"` indicates that we want that non-printing ASCII character sent to the terminal,
  rather than the literal string `\033`.

As you may have guessed from the list of variables,
  we want to give different colors based on the result of the compilation.
Big red letters make it easier to notice when something has gone wrong.
We can use the error code of the compiler to see if it has failed entirely.
In addition, if we redirect the compiler to a file,
  we can see whether it has given us any warnings.

```make
build/%.o: %.cc
	@mkdir -p $(@D)
	@printf "%b" "$(COM_COLOR)$(COM_STRING) $(OBJ_COLOR)$(@)$(NO_COLOR)\n";
	@$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $< -o $@ 2> $@.log; \
        RESULT=$$?; \
        if [ $$RESULT -ne 0 ]; then \
            printf "%-60b%b" "$(COM_COLOR)$(COM_STRING)$(OBJ_COLOR) $@" "$(ERROR_COLOR)$(ERROR_STRING)$(NO_COLOR)\n"; \
        elif [ -s $@.log ]; then \
            printf "%-60b%b" "$(COM_COLOR)$(COM_STRING)$(OBJ_COLOR) $@" "$(WARN_COLOR)$(WARN_STRING)$(NO_COLOR)\n"; \
        else  \
            printf "%-60b%b" "$(COM_COLOR)$(COM_STRING)$(OBJ_COLOR) $(@F)" "$(OK_COLOR)$(OK_STRING)$(NO_COLOR)\n"; \
        fi; \
        cat $@.log; \
        rm -f $@.log; \
        exit $$RESULT
```

This became quite a bit more complicated, so let's break it down a bit.
First, we store the error code into the variable `RESULT`.
Next, if the error code is non-zero, we print that it was an error.
Otherwise, if something was sent to the log file, we print that there was a warning.
If neither of those are the case, then the file was compiled without issue.
Finally, we print the log, clean up after ourselves, and return the result.

Note that I end each line with a backslash.
This is necessary, so that the entire block is executed in a single shell.
Otherwise, each line would be started in a separate shell, which is the default behavior.
We could override this variable by defining the `.ONESHELL` target,
  but that causes significant changes throughout the makefile.

This gives pretty output, but it would require quite a bit of repeated code.
Let's refactor it out into a variable.

```make
define run_and_test
printf "%b" "$(COM_COLOR)$(COM_STRING) $(OBJ_COLOR)$(@F)$(NO_COLOR)\r"; \
$(1) 2> $@.log; \
RESULT=$$?; \
if [ $$RESULT -ne 0 ]; then \
  printf "%-60b%b" "$(COM_COLOR)$(COM_STRING)$(OBJ_COLOR) $@" "$(ERROR_COLOR)$(ERROR_STRING)$(NO_COLOR)\n"   ; \
elif [ -s $@.log ]; then \
  printf "%-60b%b" "$(COM_COLOR)$(COM_STRING)$(OBJ_COLOR) $@" "$(WARN_COLOR)$(WARN_STRING)$(NO_COLOR)\n"   ; \
else  \
  printf "%-60b%b" "$(COM_COLOR)$(COM_STRING)$(OBJ_COLOR) $(@F)" "$(OK_COLOR)$(OK_STRING)$(NO_COLOR)\n"   ; \
fi; \
cat $@.log; \
rm -f $@.log; \
exit $$RESULT
endef

build/%.o: %.cc
	@mkdir -p $(@D)
	@$(call run_and_test,$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $< -o $@)
```

Now, we can easily change any line to print its output in a nice and colorful way.
Even easier, we can throw it into an include file, so that it doesn't clutter up the rest of the makefile.
This include file is available at
  [github](https://github.com/Lunderberg/sample_makefiles/blob/master/PrettyPrint.inc).