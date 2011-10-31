# Thinking in C++ :: Volume 1 :: Formatted for Kindle

The book by Bruce Eckel. I took the HTML and wrote a script to
reformat it for my 6-inch Kindle. Work is still in progress, but
results are good.

This is Volume 1. I'm reading it and I'll format Volume 2 when I'll
get to it.

## License

I couldn't find a license to the book, so I contacted the author,
Mr. Bruce Eckel, asking for permission. He graciously granted me
permission to do this reformatting.

## Building it Yourself

I haven't made the builder particularly user friendly (for example it
hardcodes the path to the kindlegen utility, expecting it to be at
"/opt/kindlegen/kindlegen").

Even so, you can run these commands:

```bash
cd builder/
bundle install
rake
```