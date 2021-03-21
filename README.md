# svgs2ttf

## Description

A quick & dirty FontForge + Python script to generate fonts from a directory of
SVG files (or other outline formats importable by FontForge) and some JSON
metadata.

Not to be confused with [svg2ttf](http://fontforge.github.io/generate.html).
Users with more sophisticated needs probably want
[Font Custom](https://github.com/FontCustom/fontcustom/),
[grunt-webfont](https://github.com/sapegin/grunt-webfont), or
[IcoMoon](https://icomoon.io/).

## Dependencies

* [FontForge](http://fontforge.github.io/)
  (A version with python. Developed on 20140101)
* [Python](https://www.python.org) >= 2.6 or 3

## Usage

```
$ svgs2ttf metadata.json
```

### Example metadata.json:

```
{ "props":
  { "ascent": 800
  , "descent": 200
  , "em": 1000
  , "family": "Example"
  }
, "input": "src"
, "output": [ "example.ttf" ]
, "glyphs":
  { "0x41": { "src": "a.svg" }
  , "0x42": { "src": "b.svg" }
  , "67": "c.svg"
  }
}
```

* `props` holds three kinds of font properties:
  * Shortcuts: `family` and `style` (name your font), and `lang`
    (default language for entries in the sfnt `name` table).
  * Any properties of
    [fontforge.font](http://fontforge.github.io/python.html#Font);
    get attached to the font object as an attribute.
    The lengths `ascent`, `descent`, and `em` are probably useful to set.
  * Everything else gets added to the sfnt `name` table.
* `input`, if specified, is the directory where images are found.
  (default: '.') All paths are relative to the JSON file.
* `output` is an array of output filenames. FontForge guesses the type
  from the extension. Supported types are listed in
  [FontForge's menu](http://fontforge.github.io/generate.html)
  and include SVG, TTF, and WOFF.
  (EOT currently requires a 3rd-party converter, e.g.
  [ttf2eot](http://code.google.com/p/ttf2eot/).)
* `glyphs` is a mapping whose keys are integer codepoints
  (as strings in base 2, 8, 10, or 16).
  * Values are filenames or dictionaries with the filename in `src`.
  * Other dictionary entries become attributes of the glyph object. See
    [supported attributes](http://fontforge.github.io/python.html#Glyph).
    Useful ones may include `altuni`, `comments`, `width`, and `vwidth`.
  * If a filename is not provided (value is an empty string or a dict
    without the `src` key), then `(codepoint).svg` is guessed.
* `sfnt_names`, if specified, lists entries to add directly to the
  sfnt `name` table.

See `examples/example.json` for an example that specifies more options.
Some customizations might be best done by editing the script. (See the
[FontForge python reference](http://fontforge.github.io/python.html).)

### Input file restrictions

* The JSON must be valid JSON. The format is fairly unforgiving of misplaced
  commas and single quotes, though if you want to write comments you can
  interpose [yajl](https://lloyd.github.io/yajl/)'s `json_reformat`
  like so:
  
  ```
  $ svgs2ttf <(json_reformat < with_comments.json)
  ```

* It's better if the input pictures are just plain filled paths. FontForge
  tries to do the right thing for stroked paths and unions of overlapping
  regions, but support for clipping is missing
  ([fontforge#1276](https://github.com/fontforge/fontforge/issues/1276)),
  and some interactions between stroke expansion and overlap removal (e.g.
  [fontforge#1374](https://github.com/fontforge/fontforge/issues/1374))
  were only recently corrected.

