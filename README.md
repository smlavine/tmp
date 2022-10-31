# [tmp](https://sr.ht/~smlavine/tmp)

Simple temporary file uploads over SSH.

This is still in the design stage. No code has been written yet.

# Synopsis

	... | ssh tmp@tmp.example.com

# Description

tmp is a simple service for temporary file uploads over SSH. The file to
be uploaded is provided to ssh as stdin, and upon successful upload a
link to the file will be put to stdout. On an error, like exceeding a
size limit, a message will be put to stderr.

# Options

A custom time limit can be provided:

	$ cat report.pdf | ssh tmp@tmp.example.com 14d
	https://tmp.example.com/b4r8t75.pdf
	$ cat LICENSE | ssh tmp@tmp.example.com 2m
	https://tmp.example.com/z5rq6zr.txt

By default, files are deleted 2 days after they are uploaded.

A custom file name can be provided:

	$ cat wow.mp3 | ssh tmp@tmp.example.com 1w wow.mp3
	https://tmp.example.com/wow.mp3
	$ curl https://sr.ht/~smlavine/tmp | ssh tmp@tmp.example.com 2d tmp
	https://tmp.example.com/tmp.html

The name cannot be more than 128 bytes long, and each character in the
name must match the regex `/[a-zA-Z]_-\./`. Notice, **no spaces**. A
file ending will be appended if one is not provided.

By default, a file is given a randomly generated name. Characters in the
name are from the string "abcdefghijkmnpqrstuvwxyz23456789". Letters and
numbers that might cause confusion with others in the set are removed.

# Notes

For guidance on how to install tmp on a server, see
[SETUP.md](https://git.sr.ht/~smlavine/tmp/tree/master/item/SETUP.md).

# Bugs

A custom name cannot be provided unless a custom time limit is also
provided. Another way of designing this could have been to use options
instead:

	$ ... | ssh tmp@tmp.example.com -- -t 1w -n file.txt

The `--` is necessary to prevent `ssh` from parsing the `-t`
as its own. This syntax is a bit too long for my liking, but I may
consider instead persuing something like

	$ ... | ssh tmp@tmp.example.com t=1w n=file.txt
	$ ... | ssh tmp@tmp.example.com n=recording.mp4 foo=baz bar

Options and arguments are exposed to the program server-side through the
`SSH_ORIGINAL_COMMAND` environment variable. Spaces within arguments
cannot be differentiated from the separation of arguments, so spaces
cannot be included in any command line input to `tmp`.

# Copyright

Copyright (C) 2021 Sebastian LaVine <mail@smlavine.com>

Licensed under the AGPLv3. See [LICENSE][license].

[license]: https://git.sr.ht/~smlavine/tmp/tree/master/item/LICENSE
