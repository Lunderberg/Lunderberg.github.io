---
layout: post
title: Unique server colors
tags: [linux]
---

When working on a command line, it can be easy to forget which machine one is on,
as there are few visual indications of the server.
Enter the PS1 command line variable.
This is the text shown in the prompt.

```
export PS1="\u@\h \w $"
```

As a first iteration, this isn't bad at all.
We now are shown the username, hostname, and working directory for each command.

Next, I would like to add a bit of color to it.
Compilation errors may spew out pages upon pages of text,
  and it may be difficult to find where the error messages start.
A colored prompt helps to visually distinguish between boundaries.

```
export PS1="\[\e[1;32m\]\u@\h\[\e[1;34m\] \w $\[\e[m\]"
```

Each of the `\e[x;ym` tags changes the printing color.
They are wrapped in `\[...\]` to tell bash that these are non-character codes,
  and therefore should not be included in calculating the line length.
Now, the username and host are green, and the working directory is blue.

I'd like to go one step further.
It is nice to have the name of the server,
  but I would like each server to be colored uniquely.
That way, I am less likely to mistake one server for another.
Note that my implementation here works only in terminals that support 256 colors.

First, I write a short python helper script.

```python
#!/usr/bin/env python

import random
import socket

# Colors that are bright enough to read on a black background
valid_colors = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 20,
                21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33,
                34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46,
                47, 48, 49, 50, 51, 55, 56, 57, 58, 59, 60, 61, 62,
                63, 64, 65, 66, 67, 69, 70, 71, 72, 73, 74, 75, 76,
                77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 89, 90,
                91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102,
                103, 104, 105, 106, 107, 108, 109, 110, 111, 112,113,
                114, 115, 116, 117, 118, 119, 120, 121, 122, 123,124,
                125, 126, 127, 128, 129, 130, 131, 132, 133, 134,135,
                136, 137, 138, 139, 140, 141, 142, 143, 144, 145,146,
                147, 148, 149, 150, 151, 152, 153, 154, 155, 156,157,
                158, 159, 160, 161, 162, 163, 164, 165, 166, 167,168,
                169, 170, 171, 172, 173, 174, 175, 176, 177, 178,179,
                180, 181, 182, 183, 184, 185, 186, 187, 188, 189,190,
                191, 192, 193, 194, 195, 196, 197, 198, 199, 200,201,
                201, 202, 203, 204, 205, 206, 207, 208, 209, 210,211,
                212, 213, 214, 215, 216, 217, 218, 219, 220, 221,222,
                223, 224, 225, 226, 227, 228, 229, 230, 231, 255]

random.seed(socket.gethostname())
color = random.choice(valid_colors)
print '38;5;{}'.format(color)
```

This script selects a random color, based on the name of the host.
The list of colors are the list of color codes that are bright enough
  to serve as text overtop a black background.
The printed value is the color code to be used.
Place this script somewhere in your PATH.

Now, in my `.bashrc`, adding the following lines.
```bash
# Select a color for the server, random based on hostname.
if which server_name_color.py > /dev/null; then
    SERVER_COLOR=`server_name_color.py`
else
    SERVER_COLOR="1;32"
fi
export PS1="\[\e[1;32m\]\u@\[\e[${SERVER_COLOR}m\]\h\[\e[1;34m\] \w \$\[\e[m\] "
```

As before, the color codes are used.
The server color from the script is applied only to the name of the server itself.