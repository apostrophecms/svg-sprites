<div align="center">
  <img src="https://raw.githubusercontent.com/apostrophecms/apostrophe/main/logo.svg" alt="ApostropheCMS logo" width="80" height="80">

  <h1>SVG Sprites for Apostrophe</h1>
  <p>
    <a aria-label="Apostrophe logo" href="https://v3.docs.apostrophecms.org">
      <img src="https://img.shields.io/badge/MADE%20FOR%20Apostrophe%203-000000.svg?style=for-the-badge&logo=Apostrophe&labelColor=6516dd">
    </a>
    <a aria-label="Test status" href="https://github.com/apostrophecms/svg-sprite/actions">
      <img alt="GitHub Workflow Status (branch)" src="https://img.shields.io/github/workflow/status/apostrophecms/svg-sprite/Tests/main?label=Tests&labelColor=000000&style=for-the-badge">
    </a>
    <a aria-label="Join the community on Discord" href="http://chat.apostrophecms.org">
      <img alt="" src="https://img.shields.io/discord/517772094482677790?color=5865f2&label=Join%20the%20Discord&logo=discord&logoColor=fff&labelColor=000&style=for-the-badge&logoWidth=20">
    </a>
    <a aria-label="License" href="https://github.com/apostrophecms/svg-sprite/blob/main/LICENSE.md">
      <img alt="" src="https://img.shields.io/static/v1?style=for-the-badge&labelColor=000000&label=License&message=MIT&color=3DA639">
    </a>
  </p>
</div>

This module provides an Apostrophe piece type that manages and renders SVG sprites. Sprites can be imported from files in a website codebase or an external source via a URL.

SVG sprites must be generated by a separate process. The module does not provide functionality to build the sprite files. [See below](#sprite-file-markup) for sprite markup requirements.

## Installation

To install the module, use the command line to run this command in an Apostrophe project's root directory:

```
npm install @apostrophecms/svg-sprite
```

## Usage

Configure the SVG Sprite module in the `app.js` file:

```javascript
require('apostrophe')({
  shortName: 'my-project',
  modules: {
    '@apostrophecms/svg-sprite': {}
  }
});
```

The SVG Sprites module should then be configured in its own `index.js` file with information about the sprite maps. Sprite files should be registered in the `maps` option, set to an array of configuration objects.

```javascript
// modules/@apostrophecms/svg-sprite/index.js
module.exports = {
  options: {
    maps: [
      {
        label: 'Places Icons',
        name: 'places',
        file: 'svg/places.svg'
      },
      {
        label: 'Service Icons',
        name: 'services',
        file: 'svg/services.svg'
      }
    ]
  }
}
```

The configuration objects include:

- `label`: A clear label for the group of sprites.
- `name`: A string with no whitespace that is unique within the project.
- `file`: The location of the file. This may be a local file or a URL, as discussed below.

The sprites can be imported into Apostrophe as pieces by running the module's `import` task. This task will look for each registered sprite file and generate pieces (Apostrophe content) for each SVG. On the command line this task could be started from the project root with the following command:

```bash
node app @apostrophecms/svg-sprite:import
```

### Sprite file location options

There are three options for registering SVG file locations:

1. Use a partial file path to a specific file, e.g., `'svg/places.svg'`.
2. Use a partial file path with wild card symbols, e.g., `'svg/*-icons.svg'`. See the [Glob package documentation](https://www.npmjs.com/package/glob#glob-primer) for acceptable patterns.
3. Use a URL for an externally hosted file, e.g., `'http://myfiles.net/svg/icons.svg'`.

When using the partial file path options, the module will look for those files in its own `public` directory: `modules/@apostrophecms/svg-sprite/public/`. For example, `'svg/places.svg'` would reference a file at `modules/@apostrophecms/svg-sprite/public/svg/places.svg` in the code base.

### Sprite file markup

Sprite files use the [SVG `symbol` element](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/symbol) to include multiple SVG images within a single `svg` tag. See the [CSS-Tricks](https://css-tricks.com/svg-symbol-good-choice-icons/) guide for more information about how to construct and use these sprite files.

**Requirements for this module include:**
- Sprite maps must be formatted so that all `<symbol>...<symbol/>` elements are on the same node level. This simply mean that `symbol` tags should not be nested within other `symbol` tags.
- `symbol` tags must have an `id` attribute, e.g., `<symbol id="bicycle">...</symbol>`.
- `symbol` tags can *optionally* have a `title` attribute that will be used as the imported piece's title field, e.g., `<symbol title="Bicycle icon">...</symbol>`

Here is an example of sprite file markup (with the `path` values abbeviated):

```html
<svg xmlns="http://www.w3.org/2000/svg">
	<symbol width="24" height="24" viewBox="0 0 24 24" id="coffee_cup" >
		<path d="..." />
	</symbol>
	<symbol width="24" height="24" viewBox="0 0 24 24" id="bicycle" >
		<path d="..." />
	</symbol>
</svg>
```

### Using SVG sprite pieces

The primary properties we use to reference individual SVG symbols are:
- `file`: The file path or URL to the full sprite map. This file still includes all symbols that were part of that original map file.
- `svgId`: The `id` property of the individual SVG symbol. We have to use this in combination with the file path to get a specific symbol.
- `map`: The `map` name property can be used to quickly find one of the sprite maps that were configured in the module's `maps` array.

HTML markup using an individual SVG symbol might look like the example below. The example uses `svgSprite` as a reference to the individual piece content. A project might reference sprite pieces using a relationship field, for example.

```django
<svg>
  <use xlink:href="{{ svgSprite.file }}#{{ svgSprite.id }}"></use>
</svg>
```
