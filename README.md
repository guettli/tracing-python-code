# How to trace Python source code with settrace()
Imagine you have a test which works in one environment, and fails in a different environment.

If it is reproducible, then you can add this snippet, and compare the files created in `/tmp/out*log`.

You can modify `skip_filename()` and `truncate_filename()` to you needs.

This is copy+paste code, use it only temporarily.

The code uses [sys.settrace()](https://docs.python.org/3/library/sys.html#sys.settrace) from the standard library
and [PySnooper](https://github.com/cool-RR/PySnooper) which you can install via `pip install pysnooper`.

You can copy+paste this snippet into the code which you want to trace.

In my case I wanted to see how my code behaves different under a differen Python version.

It will create a file `/tmp/out-...` with every line which the interpreter executed.

You can compare two created files with `vimdiff` on the shell, with the GUI [meld](https://meldmerge.org/) or any
other diff-tool.


```
from pysnooper.tracer import get_path_and_source_from_frame

stream = io.open('/tmp/out-%s.log' % sys.version.split()[0], 'wt', encoding='utf8')


def tracer(frame, event, arg):
    filename, source = get_path_and_source_from_frame(frame)
    if skip_filename(filename):
        return
    filename = truncate_filename(filename)
    source_line = source[frame.f_lineno - 1]
    stream.write('{event}, {filename}:{line_number} {source_line}\n'.format(
        event=event, filename=filename, source_line=source_line, line_number=frame.f_lineno))
    return tracer


def truncate_filename(filename):
    for remove in ['/home/me/foo/projects/']:  # modify this to fit your liking
        filename = filename.replace(remove, '')
    return filename


def skip_filename(filename):
    if filename.startswith('<'):
        return True
    if 'lib/python' in filename:
        # Skip /usr/lib/ and lib/python/site-packages
        # I only want to trace my code.
        return True


sys.settrace(tracer)
```

I don't think it makes sense that I create a new tracing library for Python. There are already several.

See my [Python Tips "Tracing"](https://github.com/guettli/python-tips#tracing)

# Related

* [G??ttli's opinionated Programming Guidelines](https://github.com/guettli/programming-guidelines)
* [G??ttli's opinionated Python Tips](https://github.com/guettli/python-tips)
* [G??ttli working-out-loud](https://github.com/guettli/wol)




