# How to trace Python source code with settrace()
Imagine you have a test which works in one environment, and fails in a different environment.

If it is reproducible, then you can add this snippet, and compare the files created in `/tmp/out*log`.

You can modify `skip_filename()` and `truncate_filename()` to you needs.

You can compare two created files with `vimdiff` on the shell, with the GUI [meld](https://meldmerge.org/) or any
other diff-tool.

This is copy+paste code, use it only temporarily.

```
        stream = io.open('/tmp/out-%s.log' % sys.version.split()[0], 'wt', encoding='utf8')
        def tracer(frame, event, arg):
            filename, source = get_path_and_source_from_frame(frame)
            if skip_filename(filename):
                return
            filename = truncate_filename(filename)
            source_line = source[frame.f_lineno - 1]
            if filename.startswith('<'):
                return
            stream.write('{event}, {filename}:{line_number} {source_line}\n'.format(
                event=event, filename=filename, source_line=source_line, line_number=frame.f_lineno))
            return tracer

        def truncate_filename(filename):
            for remove in ['/home/me/foo/projects/']:
                filename = filename.replace(remove, '')
            return filename

        def skip_filename(filename):
            if 'lib/python' in filename:
                # Skip /usr/lib/ and lib/python/site-packages
                return True
        sys.settrace(tracer)
```


# Related

* [Güttli's opinionated Programming Guidelines](https://github.com/guettli/programming-guidelines)
* [Güttli's opinionated Python Tips](https://github.com/guettli/python-tips)
* [Güttli working-out-loud](https://github.com/guettli/wol)




