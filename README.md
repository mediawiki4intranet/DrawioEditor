# MediaWiki draw.io editor extension

This is a MediaWiki extension that integrates the draw.io flow chart editor and allows inline editing of charts.

# Warnings

**Please read these warnings carefully before use**:
- This extension requires a draw.io web service.
- By default, https://www.draw.io is used, but you can self-host it and set your own URL in `$wgDrawioEditorServiceUrl`.
  Be aware that https://www.draw.io is an online service and while this plugin integrates the editor using an iframe and
  only communicates with it locally in your browser (javascript postMessage), it cannot guarantee that the code loaded
  from draw.io will not upload any data to foreign servers. **Self-host draw.io in case if you're concerned about privacy.**
- Mediawiki4Intranet includes a self-hosted draw.io.

# Features

- **draw.io chart creation and editing**.
- **SVG** and PNG support. The file type can be configured globally and changed on a per-image basis.
- **Inline Editing** and javascript uploads on save, you never leave or reload the wiki page.
- Image files are transparently **stored in the standard wiki file store**, you don't need to worry about them.
- **Versioning** is provided by the file store.
- Draw.io original XML data is stored within the image files, so only one file must be stored per chart.
- Supports multiple charts per page.
- Supports relative and fixed chart dimensions.

# Requirements

- When you intend to use SVG which is recommended, you might want to install Extension:NativeSvgHandler too. Also you need a browser that supports SVG.
- While displaying charts may work in older browsers, especially when using PNG (SVG is default and recommended), saving charts requires a fairly recent browser.
- The drawings produced by the draw.io editor make use of the xhtml namespace in SVG files.
  This namespace is not allowed for file uploads in MediaWiki 1.26 and later.
  So if you want to use SVG with this extension, you need to use a MediaWiki that is older or you need to patch it (see Installation).
  Hopefully this requirement is only temporary. She this [issue](https://www.github.com/mgeb/mediawiki-drawio-editor/issues/1) and the [one on Wikimedia's phabricator](https://phabricator.wikimedia.org/T138783).

# Installation

1. Clone this plugin into a folder named DrawioEditor within your wiki's extensions folder:

   ```shell
   cd /where/your/wiki/is/extensions
   git clone https://github.com/mgeb/mediawiki-drawio-editor DrawioEditor
   ```

3. Activate the plugin in LocalSettings.php and set configuration:

   ```
   require_once "$IP/extensions/DrawioEditor/DrawioEditor.php";
   $wgDrawioEditorServiceUrl = 'https://www.draw.io';
   ```

3. If you want so use SVG (recommended) and the version of your MediaWiki is 1.26 or newer,
   you need to add the namespace ```http://www.w3.org/1999/xhtml``` to ```$validNamespaces``` in ```includes/upload/UploadBase.php```.
   See Requirements for more information on why this is currently needed.

# Usage

## Add or upload a chart

DrawioEditor handles charts saved as file uploads with names like `File:<ChartName>.drawio.svg`.
To edit charts they are required to include a copy of the diagram source
(Export -> SVG -> "Include a copy of my diagram" in draw.io).

1. If you want to upload an existing chart, export it as SVG with "a copy of the diagram"
   and upload in into MediaWiki with a name ending in `.drawio.svg`.
2. Add the following tag to any wiki page to insert a draw.io chart:
   ```wiki
   {{#drawio:ChartName}}
   ```
   `ChartName` is the part of the file name before `.drawio.svg`.
3. Save the wiki page.
4. Click [Edit] at the top right of the page.
5. Draw your chart, click Save to save and Exit to leave Edit mode.

## Edit a chart

Each chart will have an Edit link at it's top right. Click it to get back into the editor. Click Save to save and Exit to get out of the editor. If a wiki page has multiple charts, only one can be edited at the same time.

## View or revert to old versions

On each save, a new version of the backing file will be added to the wiki file store. You can get there by clicking the chart while you're not editing. There you can look at and revert to old versions like with any file uploaded to the wiki.

## Options ##

Options are appended to the tag separated by `|`. For example:

```wiki
{{#drawio:ChartName|type=svg|max-width=500px}}
```

### Chart dimensions

While the defaults are good under most circumstances, it may be necessary to control how your chart is displayed. The following options can be used for that:

* _width_: Sets the chart width. Defaults to `100%`.
* _max-width_: Set the maximum chart width if _width_ is relative. Defaults to `chart`.
* _height_: Sets the chart height. Defaults to `auto`. Usually not used.

These option take any absolute CSS length value as argument, for example:

* `400px`
* `80%`
* `auto`

The keyword `chart` has a special meaning and stands for the actual size of the chart. When the chart is saved, the image dimensions are automatically adjusted. Usually it's preferable to use `chart` instead of a fixed pixel value.

The default behaviour is the let the chart scale (`width=100%` and `height=auto`) until it reaches its actual width stored in the chart (`max-width=chart`).

If you want it to scale further or less, you can adjust _max_width_ manually. Use `none` to allow infinite scaling.

If you just want a fixed width, set _width_ to `chart` or some fixed CSS value and leave _height_ on `auto`. Unless you need a fixed sized image area before the image is actually loaded or really need to scale based on height, there is usually no point in setting _height_. If you set it you probably should set _width_ to `auto`, or when setting both use `chart` so you don't need to update the values manually every time your image changes.

#### Examples

* Let the chart scale until it reaches its actual width (default):
  
  ```wiki
  {{#drawio:ChartName}}
  ```
  Same as:
  
  ```wiki
  {{#drawio:ChartName|width=100%|max-width=chart|height=auto}}
  ```
  
* Let the chart scale until it's 800 px wide:

  ```wiki
  {{#drawio:ChartName|max-width=800px}}
  ```

* Let the chart scale infinitely:

  ```wiki
  {{#drawio:ChartName|max-width=none}}
  ```

* Fixed width:
  
  ```wiki
  {{#drawio:ChartName|width=600px}}
  ```
  
* Fixed width using the actual chart width:
  
  ```wiki
  {{#drawio:ChartName|width=chart}}
  ```
  
* Fixed height and width using the actual chart dimensions:

  ```wiki
  {{#drawio:ChartName|width=chart|height=chart}}
  ```

### File type
The file type to be used can be set to either `png` or `svg`. The default is `svg` unless you set $wgDrawioEditorImageType to png in LocalSettings.php.
  
```wiki
{{#drawio:ChartName|type=png}}
```

### Use an Object Tag
MediaWiki normally embeds images inside `<img>` tags and links them to their description page. If you want to use SVG multimedia functions (e.g. links) it has to be embedded as `<object>`. This can be set using the interactive attribute or generally by enabling `$wgDrawioEditorImageInteractive = true` in LocalSettings.php.

```wiki
{{#drawio:ChartName|interactive}}
```


# Privacy
As mentioned in the Warnings Section above, **there are some privacy concerns when using this plugin (or draw.io in general)**. Carefully read the information below, especially when you're running a wiki in a private environment.

**draw.io may change it's code at any time, so the everything below could be outdated by the time you read it.**

## Referrer Leak
The draw.io editor is loaded within an iframe when you click the Edit link for a chart. At this point your browser loads all draw.io code from draw.io servers. While it is running in an iframe and has no access to your wiki page contents or any other resources, your browser may still send a referrer containing the current wiki page's URL to the draw.io servers, which may or may not be a problem in your environment. The wiki setting $wgReferrerPolicy may help you with this, but only for modern browsers.

## Chart Data Privacy
Obviously the chart data must be passed to the draw.io application. The plugin uses the draw.io embed mode and passes the  data to draw.io running in an iframe using javascript's postMessage(). This part happens locally, the data does not leave your browser. Currently, there does not seem to be any interaction with the draw.io servers while editing, which is good but of course this could change at any time without you or your wiki's users noticing. When saving, the file data is prepared (exported) by the iframe and passed back to this plugin (again) using postMessage(). The plugin then safely uploads the new version to the wiki file store. While the data passing happens locally and uploading is safe because it's done by this plugin in the main window context, the draw.io data export code running in the iframe seems to require interaction with draw.io servers in some cases.

One example is when you are using the Safari browser and save a chart which uses type png (see Options above). That process does not seem to be entirely implemented in javascript and needs the draw.io servers to generate the PNG data. This means your chart data leaves the browser. It is sent SSL encrypted and the draw.io folks probably don't care about your chart, but of course it's up to you to decide wether you can accept his or not. SVG does not seem to have that problem, at least in Chrome, Firefox and Safari, so I recommend using that. There may be other circumstances under which data leaves the browser. If this is a concern, you should check wether your use cases trigger such uploads, or not use this plugin and draw.io at all.

Again, be aware that the draw.io code running in the iframe may change its behavior at any time without you noticing. While that code has no access to your wiki, it may cause your chart data to be leaked. If this is a concern, don't use this plugin.

# Links
https://www.mediawiki.org/wiki/Extension:DrawioEditor
