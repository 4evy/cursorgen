# cursorgen

`cursorgen` converts Windows cursors (`.cur` and animated `.ani`) and existing
Xcursor files into Xcursor files for Linux cursor themes on X11 and Wayland

It is a fork of [win2xcur](https://github.com/quantum5/win2xcur), using OpenCV
and NumPy to preserve transparency, scale hotspots, and retain animation timing.
Exact nearest-neighbor scaling keeps pixel edges sharp and generates 13 cursor
sizes from 22 to 96 pixels. Python installs use headless OpenCV without GUI
dependencies. ImageMagick is not required

CUR entries use the PNG default image when an embedded PNG contains animation.
ANI files retain their frame order and timing

Xcursor round trips without resizing preserve the stored premultiplied pixel
values. Hotspots can lie on the right or bottom image boundary and scale with
the image, including when converting between 24-pixel and 32-pixel cursors

## Installation

Python 3.7 or newer is required. Install from a checkout with pip:

```sh
git clone https://github.com/meanvoid/cursorgen.git
cd cursorgen
python -m venv .venv
. .venv/bin/activate
python -m pip install .
```

On Windows, activate the environment with `.venv\Scripts\activate` instead

## Usage

Convert one file or a cursor pack:

```sh
cursorgen arrow.cur -o output
cursorgen windows-cursors/*.cur windows-cursors/*.ani -o output
cursorgen --help
```

The output directory is created automatically. `arrow.cur` becomes
`output/arrow`, with no extension. Files with the same stem, such as
`arrow.cur` and `arrow.ani`, need separate output directories. Existing output
files are replaced unless they refer to an input file, including through a
symlink or hardlink. Output files must also be distinct from each other

Try the bundled samples from the checkout:

```sh
cursorgen sample/crosshair.cur -o output/static
cursorgen sample/crosshair.ani -o output/animated
```

Run `python fetch_samples.py` to checksum-verify the samples and download fresh
copies if needed. The static crosshair comes from win2xcur.
The animated sample is a text-selection cursor from
[Yorumu's Matikanefukukitaru release](https://www.pixiv.net/artworks/93099359)

For use in a theme, put converted files in `<theme>/cursors/` under role names
such as `left_ptr` for the pointer and `xterm` for text selection, then select
the theme in your desktop settings. See the
[Xcursor theme documentation](https://www.x.org/archive/X11R7.5/doc/man/man3/Xcursor.3.html#toc6)
for directory layout and inheritance

## Nix

The flake supplies Python and all dependencies, pinned through `flake.lock`.
It supports `x86_64-linux`, `aarch64-linux`, and `aarch64-darwin`. Enable
[Nix flakes](https://nix.dev/concepts/flakes.html) and `nix-command`, then run
from the checkout:

```sh
nix run . -- sample/crosshair.cur -o output/nix
nix build .
./result/bin/cursorgen --help
```

## Development

Use Python 3.10 or newer for development tools and the existing test suite:

```sh
python -m pip install -e '.[dev]'
cursorgen sample/crosshair.cur -o output/dev
python -m unittest discover -s tests -v
pre-commit run --all-files
python -m build
```

CI also checks pip installation and sample conversion on every supported Python
version, including Python 3.7

For a Nix development shell, run `nix develop`, then use
`python -m cursorgen` to run with Nix's Python and libraries. Run
`nix flake check` to build the Nix package, run the existing unittest suite, and
check its Python imports. Build a wheel using the shell's build dependencies
with `python -m build --wheel --no-isolation`

Licensed under [GPL-3.0-or-later](LICENSE)
