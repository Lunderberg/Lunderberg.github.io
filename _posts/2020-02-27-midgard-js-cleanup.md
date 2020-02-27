---
layout: post
title: Midgard, JavaScript Cleanup
tags: [c++, midgard]
---

I've been focusing mostly on the C++ side of the program, using the
JavaScript entirely for rendering.

First up, allowing for a better file structure.  Currently, all the
Javascript is inside a single `.js` file.  While this is easy to embed
into a website, it is not the greatest for maintenance.  While I could
add several `<script>` tags, that would be tedious as well.  If I were
doing this professionally, I would set up a Javascript bundler, to
read in files, follow all calls to `require()`, pull from npm, and
output a single minified `.js` file.  Since none of that sounds
interesting for a hobby project, let's throw a kludge together
instead.

Since C++17, the `std::filesystem` can be used to list files.  Rather
than generating a minified `.js` files, I'm going to cat them all
together on request.  That way, I can edit the .js file, refresh the
page, and immediately see updates.

```c++
std::string cat_js_files(const std::string& dir) {
  std::vector<std::string> js_files;
  for(auto& p : fs::directory_iterator(dir)) {
    if(p.path().extension() == ".js") {
      js_files.push_back(p.path());
    }
  }

  std::sort(js_files.begin(), js_files.end());

  std::vector<char> text;
  for(auto& filename : js_files) {
    std::ifstream ifile(filename);
    text.insert(text.end(),
                std::istreambuf_iterator<char>(ifile),
                std::istreambuf_iterator<char>());
  }

  return std::string(text.begin(), text.end());
}
```

With this change, I can now split the single javascript file up into
separate files for the websocket connection, the canvas drawing, and
the user interaction.

Next up on the cleaning path, making the creature-drawing be more
sensible.  The first implementation had the size of the world
hard-coded, because that was available only from the grass growth
messages.  Now, the world state, including its size, is remembered and
used in the draw step.  This also has the possibility of future
improvements, such as only sending changes, and not the full state of
the world.

The end result of this refactoring has the exact same display as it
did previously, so all is well in the world.