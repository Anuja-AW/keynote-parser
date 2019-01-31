# keynote-parser

`keynote-parser` is a Python module for unpacking and re-packing
[Apple Keynote](https://www.apple.com/keynote/) `.key` files. It supports Keynote
files generated by Keynote version 8.3 (current as of January 2019).

Keynote uses a proprietary, compressed binary format to store its presentations.
This format is comprised of a zip file containing images and videos, as well as
[Snappy](https://github.com/google/snappy)-compressed
[Protobuf](https://github.com/protocolbuffers/protobuf) `.iwa` files containing
metadata, text, and all other definitions used in the presentation.

`keynote-parser` unpacks these component files into `.yaml` files in a directory,
making them editable by text editors and/or scripts, then allows re-packing of these
files into a working Keynote archive.

What could you use this for? Well, I use it to allow versioning of Keynote files in Git,
which makes diffs more understandable (rather than binary), as well as modifying text
in Keynote files in response to external scripts.
(e.g.: figures that update from databases before giving a presentation)

## Installation

```bash
pip install keynote-parser
```

## Usage

```bash 
# Unpack MyPresentation.key into ./MyPresentation/
keynote-parser unpack MyPresentation.key

# Re-pack ./MyPresentation/ into MyPresentation.out.key 
keynote-parser pack ./MyPresentation/

# List the files within a Keynote archive
keynote-parser ls MyPresentation.key

# Dump a particular .iwa file into its yaml representation on stdout
keynote-parser cat MyPresentation.key /Index/Slide-00001.iwa

# Replace text within a Keynote file in-place
keynote-parser replace MyPresentation.key --find "hello world" --replace "hello dolly"
```

### Formats

`keynote-parser` supports reading a list of replacements from a JSON file passed in
as `--replacements`. This file must have the form:

```json
{
  "replacements": [
    {
      "find": "regexp to search for",
      "replace": "string to replace with"
    },
    ...
  ]
}
```

This argument can be passed to `keynote-parser replace` to replace text in a Keynote
file in-place. It can also be passed to `keynote-parser pack` to pack a directory
into a Keynote file, replacing text along the way.


## Updates

As `keynote-parser` includes private Protobuf definitions extracted from a copy of Keynote,
new versions of Keynote will inevitably create `.key` files that cannot be read by `keynote-parser`.
As new versions of Keynote are released, the following steps must be undertaken:

* Run [proto-dump](https://github.com/obriensp/proto-dump) on the new copy of Keynote to dump new Proto files.
  * Any `.` characters in the Protobuf definitions must be changed to `_` characters manually.
* Connect to a running copy of `Keynote` with `lldb` (or any other debugger) and manually copy
  the results of `[TSPRegistry sharedRegistry]` into `mapping.py`.
  * Versions of macOS >= 10.11 may protect Keynote from being attached to by a debugger -
    to attach, [temporarily disable System IntegrityProtection](https://apple.stackexchange.com/questions/208478/how-do-i-disable-system-integrity-protection-sip-aka-rootless-on-macos-os-x)
    to get this data.

## Credits

`keynote-parser` was built by [Peter Sobot](https://petersobot.com) but heavily based on [prior
work](https://github.com/obriensp/iWorkFileFormat/blob/master/Docs/index.md) by [Sean Patrick O'Brien](http://www.obriensp.com).
A copy of O'Brien's format documentation is included in the `docs` folder for posterity.

## License

All code in this repository is licensed under the MIT License, save for `protobuf_patch.py`, which contains its own license.

```
Copyright 2019 Peter Sobot

Permission is hereby granted, free of charge, to any person obtaining a copy of this software
and associated documentation files (the "Software"), to deal in the Software without restriction,
including without limitation the rights to use, copy, modify, merge, publish, distribute,
sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or
substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING
BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM,
DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```
