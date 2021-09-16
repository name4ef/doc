```
CC          command for call of C compiler
CFLAGS      parameters for C compiler
CXX         command for call of C++ compiler
CXXFLAGS    parameters for C++ compiler
CPPFLAGS    parameters for preprocessor (place for define of macrovariables)
LD          command for call of system linker
MAKE        command for call of make utile with all parameters

$@  name of current target
$<  name of first target from list of dependencies
$^  all list of dependencies

-   ignore errors
```

#### Example
``` make
SRCMODULES = mod1.c mod2.c
OBJMODULES = $(SRCMODULES:.c=.o)
CFLAGS = -g -Wall -ansi -pedantic

%.o: %.c %.h
    $(CC) $(CFLAGS) -c $< -o $@

prog: main.c $(OBJMODULES)
    $(CC) $(CFLAGS) $^ -o $@
    
ifneq (clean, $(MAKECMDGOALS))
-include deps.mk
endif

deps.mk: $(SRCMODULES)
    $(CC) -MM $^ > $@
    
clean:
    rm -f *.o prog
```
