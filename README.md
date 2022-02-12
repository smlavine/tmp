# tmp - Simple temporary file uploads over SSH

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
name must match ```[a-zA-Z]_-\.```. Notice, **no spaces**. A file ending
will be appended if one is not provided.

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

The ```--``` is necessary to prevent ```ssh``` from parsing the ```-t```
as its own. This syntax is a bit too long for my liking, but I may
consider instead persuing something like

	$ ... | ssh tmp@tmp.example.com t=1w n=file.txt
	$ ... | ssh tmp@tmp.example.com n=recording.mp4 foo=baz bar

Options and arguments are exposed to the program server-side through the
```SSH_ORIGINAL_COMMAND``` environment variable. Spaces within arguments
cannot be differentiated from the separation of arguments, so spaces
cannot be included in any command line input to ```tmp```.

This is still in the design stage. No code has been written yet.

# Copyright

Copyright (C) 2021 Sebastian LaVine <mail@smlavine.com>

tmp is free software: you can redistribute it and/or modify it under
the terms of the GNU Affero General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

tmp is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.

---

This project is hosted at <https://sr.ht/~smlavine/tmp>.

To browse the source code repository, see
<https://git.sr.ht/~smlavine/tmp>.

For development discussion and patches related to the tmp project, see
the mailing list at <https://lists.sr.ht/~smlavine/tmp-devel>.

For announcements related to the tmp project, see the mailing list at
<https://lists.sr.ht/~smlavine/tmp-announce>.
