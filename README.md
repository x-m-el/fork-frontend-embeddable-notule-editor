# frontend-embeddable-notule-editor

This application allows you to embed the RDFa editor in other applications without integrating with EmberJS directly. It will behave like any other HTML editor.

## Target usage

The idea is that you can have multiple HTML tags in which you can initialize an editor. We'll explain how it works and the process can be repeated if multiple editors are required. We need some HTML structure to start with, and then set-up the editor when everything has finished loading and rendering. We can also initialize the editor with some content or set it later via an event. Use something like the following HTML snippet as a base. *Note that the order of the JavaScript files matters.*

```html
<!DOCTYPE html>
<html>
  <head>
    <title>I have an editor in my document</title>

    <!-- Requirements for the style -->
    <link rel="stylesheet" href="assets/frontend-embeddable-notule-editor.css">
    <link rel="stylesheet" href="assets/vendor.css">

    <!-- Sources of the editor, THE ORDER MATTERS -->
    <script src="assets/vendor.js"></script>
    <script src="assets/frontend-embeddable-notule-editor-app.js"></script>
    <script src="assets/frontend-embeddable-notule-editor.js"></script>
  </head>
  <body>
    ...
  </body>
</html>
```

Next, put some tags in the body of the page. We'll place the editor in those tags.

```html
<body>
  <div id="my-editor"></div>
</body>
```

Lastly, we'll instantiate the editor. We wait until the DOM has loaded and then initialize the embedded EmberJS app with the editor inside. Put this script in the head of the HTML page with `<script>...</script>`, or use another method if desired (e.g. at the bottom of the HTML).

```javascript
window.addEventListener('load', function () {
  let App = require('frontend-embeddable-notule-editor/app').default.create({
    autoboot: false,
    name: 'frontend-embeddable-notule-editor'
  });
  App.visit('/', { rootElement: '#my-editor' }).then(() => {
    const editorContainer = document.getElementById('my-editor');
    const editorElement =
    editorContainer.getElementsByClassName('notule-editor')[0];
    console.log(editorElement);
    const arrayOfPluginNames = ['citation', 'rdfa-date'];
    const userConfigObject = {}
    editorElement.initEditor(arrayOfPluginNames, userConfigObject);
  });
```

Once the editor is initialized, you can get the relevant document node and set its content. You can play with this by opening the developer console and executing the following, or use the following code in another script:

```javascript
const editorContainer = document.getElementById('my-editor');
const editorElement = editorContainer.getElementsByClassName('notule-editor')[0]
editorElement.setHtmlContent('<h1>Hello World</h1>');  // note: the content in the page changes
console.log(editorElement.getHtmlContent());           // note: there may be a difference in returned content
```

The contents may be slightly different between the two modes. As the editor evolves, the exporting functionality will be able to better filter out the relevant HTML and remove temporary styling.


You can enable/disable an environment banner using the following methods:
```javascript
editorElement.enableEnvironmentBanner('Testing');
editorElement.enableEnvironmentBanner(); // the default environment name is 'Test'
editorElement.disableEnvironmentBanner();
```

For a complete version of this example, checkout this file: [public/test.html](public/test.html). It also includes another button that inserts a template in the editor to showcase the plugins.

## Prosemirror

The rdfa editor uses prosemirror as a base, after the `editorElement.initEditor()` function is called you will have access to the editor controller with `editorElement.controller` this is an instance of the `ProseController` class fo the (ember-rdfa-editor)[https://github.com/lblod/ember-rdfa-editor]
It provides the following methods:
- `toggleMark(name: string, includeEmbeddedView = false)`: method which allows to enable/disable a specific mark on the current selection. Expects the name of the mark to toggle and whether the command should be applied on an embedded editor view if such a view is active.
- `focus(includeEmbeddedView = false)`: method which allows one to focus the main editor view (or an embedded view, if such a view is active).
- `setHtmlContent(content: string)`: sets the content of the main editor.
- `doCommand(command: Command, includeEmbeddedView = false)`: executes a Prosemirror command (https://prosemirror.net/docs/guide/#commands) on the main editor, or when active an embedded editor instance.
- `checkCommand(command: Command, includeEmbeddedView = false)`: checks whether a Prosemirror command may be executed.
- `isMarkActive(mark: MarkType, includeEmbeddedView = false)`: checks whether a mark is currently active.
- `withTransaction(callback: (tr: Transaction) => Transaction | null, includeEmbeddedView = false)`: method which allows you to apply a transaction on the main view (or currently active embedded view). When you want to apply the transaction, the callback should return a transaction object.
- `getState(includeEmbeddedView = false)`: used to request the current state of the main editor view (or an embedded view if active).
- `getView`: used to request the main editor view (or an embedded view if active).
- `setEmbeddedView(view: RdfaEditorView)`: activate a specific embedded view.
- `clearEmbeddedView`: deactive the current embedded view.

Additionally, a controller provides the following attributes:
- `externalContextStore`: provides an instance of `ProseStore` describing the RDFa around the editor element.
- `datastore`: provides an instance of `ProseStore` describing the RDFa inside the editor element.
- `schema`: provides the schema of the main editor.
- `state`: provides the current state (see https://prosemirror.net/docs/guide/#state) of the main editor.
- `view`: provides the main editor view (see https://prosemirror.net/docs/guide/#view).


## Building the sources

In order to build the JavaScript and CSS sources of this repository you will need `ember-cli` installed (more info at section *Development of frontend-embeddable-notule-editor* below), then execute the following:

```bash
git clone https://github.com/lblod/frontend-embeddable-notule-editor.git
cd frontend-embeddable-notule-editor
npm install
ember build -prod
```

In the 'dist' folder structure, two CSS files and three JavaScript files will have been generated. These are the files to use, as demonstrated in the example above. Note that the fingerprints of your files may vary.

```bash
dist
└── assets
    ├── frontend-embeddable-notule-editor-app.js
    ├── frontend-embeddable-notule-editor.css
    ├── frontend-embeddable-notule-editor.js
    ├── vendor.css
    └── vendor.js
```

## Configuring the editor

The editor can be customized to best fit your application. In order to use the editor with these options, be sure to rebuild the sources.

### Adding/removing plugins
Embeddable ships with the following plugins available, for more info on each of them and posible configurations, check the documentation of [lblod/ember-rdfa-editor-lblod-plugins](https://github.com/lblod/ember-rdfa-editor-lblod-plugins):
* `besluit`: mostly provides the correct nodes for constructing a besluit, it's mostly useful for validation in prosemirror internals
* `citation`: recognizes citations and allows inserting an annotation manually, see more at the [plugin docs](https://github.com/lblod/ember-rdfa-editor-lblod-plugins#citaten-plugin)
* `rdfa-date`: allow inserting and modifying annoted date and times, see more at the [plugin docs](https://github.com/lblod/ember-rdfa-editor-lblod-plugins#rdfa-date-plugin)
* `roadsign-regulation`: allow inserting roadsign regulation, based on the registry managed and provided by MOW, see more at the [plugin docs](https://github.com/lblod/ember-rdfa-editor-lblod-plugins#roadsign-regulation-plugin)
* `template-variable`: Related to the roadsign-regulation plugin, allows filling in variables in the road sign regulation templates, see more at the [plugin docs](https://github.com/lblod/ember-rdfa-editor-lblod-plugins#template-variable-plugin)
* `variable`: Allows insertion of custom variables to be later filled by the template-variable plugin, see more at the [plugin docs](https://github.com/lblod/ember-rdfa-editor-lblod-plugins#insert-variable-plugin)
* `article-structure`: Provides several structures to better manage official documents, like titles, chapters, articles and paragraphs. Allows you to insert, move and delete them in an easy way, it has 2 modes that can be set in the configuration 'besluit' for only being able to add besluit_articles and 'regulatoryStatement' for all the other structures.
* `table-of-contents`: Provides a table of contents that allow you to click on it to go to the different sections specified with the article-structure plugin, see more at the [plugin docs](https://github.com/lblod/ember-rdfa-editor-lblod-plugins#table-of-contents-plugin)
* `formatting-toggle`: Allows to toggle on and off the formatting marks
* `rdfa-blocks-toggle`: Allows to toggle on and off the visual indications of the rdfa blocks

See above for how these plugins can be enabled.
ATTENTION: Currently the besluit plugin is incompatible with the regulatoryStatement mode in the article-structure plugin, so if you want to activate that mode you will need to disable the besluit plugin

### Default configuration
We provide the following defaults in case you enable a plugin and don't provide any configuration to it, you can take it as a base for your desired configuration. Take into account that if you provide any configuration to a plugin all of the default will be overrided, so make sure you include all the relevant attributes.

```
{
  docContent: 'block+',
  date: {
    placeholder: {
      insertDate: this.intl.t('date-plugin.insert.date'),
      insertDateTime: this.intl.t('date-plugin.insert.datetime'),
    },
    formats: [
      {
        label: 'Short Date',
        key: 'short',
        dateFormat: 'dd/MM/yy',
        dateTimeFormat: 'dd/MM/yy HH:mm',
      },
      {
        label: 'Long Date',
        key: 'long',
        dateFormat: 'EEEE dd MMMM yyyy',
        dateTimeFormat: 'PPPPp',
      },
    ],
    allowCustomFormat: true,
  },
  citation: {
    type: 'ranges',
    activeInRanges: (state) => [[0, state.doc.content.size]],
  },
  variable: {
    type: 'ranges',
    activeInRanges: (state) => [[0, state.doc.content.size]],
  },
  tableOfContents: [
    {
      nodeHierarchy: [
        'title|chapter|section|subsection|article',
        'structure_header|article_header',
      ],
    },
  ],
  articleStructure: {
    mode: 'besluit',
  }
}
```

Let's break down this configuration block by block:

`docContent: 'block+'`
The property docContent specifies which nodes do you want to allow in your document, in this case we allow one or more nodes that are of the supertype block, for more info about this check the [prosemirror docs](https://prosemirror.net/docs/guide/#schema.content_expressions)

```
date: {
  placeholder: {
    insertDate: this.intl.t('date-plugin.insert.date'),
    insertDateTime: this.intl.t('date-plugin.insert.datetime'),
  },
  formats: [
    {
      label: 'Short Date',
      key: 'short',
      dateFormat: 'dd/MM/yy',
      dateTimeFormat: 'dd/MM/yy HH:mm',
    },
    {
      label: 'Long Date',
      key: 'long',
      dateFormat: 'EEEE dd MMMM yyyy',
      dateTimeFormat: 'PPPPp',
    },
  ],
  allowCustomFormat: true,
},
```
This block configures the date plugin, first the placeholder block specifies 2 attributes `insertDate` and `insertDateTime` in the default config we use `intl` to correctly set the string to the language of the user, for example the corresponding strings in english are `Insert date` and `Insert date and time`.
Then we define the formats offered to the user, each format has 4 attributes:
- label: This is the label to be shown to the user on the card, is an optional property, if no label is specified the dateFormat and dateTimeFormat will be used as labels
- key: A unique key to identify the format, if the key is not unique it might cause problems
- dateFormat: The format to use when the user is inserting a date
- dateTimeFormat: The format to use when the user is inserting a date with time information
The final property is `allowCustomFormat` if this is set to true the user will be able to specify it's own format when inserting the date
For more information about date formats check the documentation of the underlying library used [date-fns](https://date-fns.org/v2.29.3/docs/format)

```
citation: {
  type: 'ranges',
  activeInRanges: (state) => [[0, state.doc.content.size]],
},
variable: {
  type: 'ranges',
  activeInRanges: (state) => [[0, state.doc.content.size]],
},
```
This block configures both the citation and the variable plugin in the same way, it basically set the configuration type as `ranges` and set the active range of the plugin to trigger in the entire document, this is a very basic configuration, to see how to specify a different trigger zone or hopw to define custom variables check the docs of the [citation](https://github.com/lblod/ember-rdfa-editor-lblod-plugins#citaten-plugin) and the [variable](https://github.com/lblod/ember-rdfa-editor-lblod-plugins#insert-variable-plugin) plugins

```
tableOfContents: [
  {
    nodeHierarchy: [
      'title|chapter|section|subsection|article',
      'structure_header|article_header',
    ],
  },
],
```
This block configures the table of contents plugin, it just specifies the nodeHierarchy that the node has to follow. This can be configured with your custom nodes or changing the order of the default ones if you want.

```
articleStructure: {
  mode: 'besluit',
}
```
As said in the plugin description the articleStructure we have simplified this plugin to be basically a toggle between 2 modes, you can select if you want the `mode: 'besluit'` that basically includes the besluit_article structure, or the `mode: 'regulatoryStatement'` that includes all the other structures like title, chapter, section, subsection, article ...

### How to use the plugins

#### Besluit
As said above this plugin is mostly useful for validation, if you are working with decisions we recommend to enable this plugin. The only way to interact with this plugin is if you delete the title of a besluit, in this case a card will appear saying that the decision is missing a title and will provide a button to insert it.
![besluit plugin](https://imgur.com/iwCudJy.png)

#### Citation
You will have 2 ways of inserting a citation, first if you open the insert menu on the right sidebar you will see a button that says `Insert reference`, if you click on that a modal will appear where you can search and insert the citation you want.
The other way of triggering the plugin is to write one of the trigger phrases described in the plugin docs:
* [specification]**decreet** [words to search for] *(e.g. "gemeentedecreet wijziging")*
* **omzendbrief** [words to search for]
* **verdrag** [words to search for]
* **grondwetswijziging** [words to search for]
* **samenwerkingsakkoord** [words to search for]
* [specification]**wetboek** [words to search for]
* **protocol** [words to search for]
* **besluit van de vlaamse regering** [words to search for]
* **gecoordineerde wetten** [words to search for]
* [specification]**wet** [words to search for] *(e.g. "kieswet wijziging", or "grondwet")*
* **koninklijk besluit** [words to search for]
* **ministerieel besluit** [words to search for]
* **genummerd besluit** [words to search for]

After writting this a little card will appear on the right that you can expand to the big modal.

#### Roadsign Regulation
For this plugin you will need to be in a besluit with one of these types:
* `https://data.vlaanderen.be/id/concept/BesluitType/4d8f678a-6fa4-4d5f-a2a1-80974e43bf34`
* `https://data.vlaanderen.be/id/concept/BesluitType/7d95fd2e-3cc9-4a4c-a58e-0fbc408c2f9b`
* `https://data.vlaanderen.be/id/concept/BesluitType/3bba9f10-faff-49a6-acaa-85af7f2199a3`
* `https://data.vlaanderen.be/id/concept/BesluitType/0d1278af-b69e-4152-a418-ec5cfd1c7d0b`
* `https://data.vlaanderen.be/id/concept/BesluitType/e8afe7c5-9640-4db8-8f74-3f023bec3241`
* `https://data.vlaanderen.be/id/concept/BesluitType/256bd04a-b74b-4f2a-8f5d-14dda4765af9`
* `https://data.vlaanderen.be/id/concept/BesluitType/67378dd0-5413-474b-8996-d992ef81637a`

If this condition is met you will find a button called `Voeg mobiliteitsmaatregel in` in the insert menu. When this button is clicked a modal will apear where you can filter and select which roadsign regulation you want to insert.
![citation plugin](https://imgur.com/oerd9rV.png)

#### Variable
This plugin allows you to insert variables in the document, a variable is a value to be filled by another user, ideally you will have 2 separate instances of the editor, the first one will enable this plugin and the other will enable the template-variable plugin.
In order to insert a variable you will need to use the insert variable card
![insert variable card](https://imgur.com/9kSqgXc.png)
In this dropdown you can select one of the 4 default types of variables (or more if you added your custom variables to the config): text, number, date or codelist. If codelist is selected you will need to specify which codelist you want to use

#### Template variable
This plugin allows to fill the codelist variables generated by the variable plugin, if you are in one of these variables the following card will appear:
![template variable card](https://imgur.com/b5Kmmqj.png)
Using this card you will be able to select one of the codelist values.

#### Article Structure
This plugin is in charge of inserting and manipulating structures, for inserting the structures you will find the buttons on the insert menu of the right sidebar, for example if you are in besluit mode you will only have a button that says `Insert article`, if you are in regulatory statement mode you will be able to insert titles, chapters, sections...
After inserting a structure you will be presented with a card, where you can move up and down the structure or delete it. For deleting you have 2 options, deleting just the structure if possible, trying to upwrap the content or deleting the structure with its content. Note that these buttons can be disabled if the action is not possible.
![article structure card](https://imgur.com/2zkbNw3.png)

#### Table of Contents
This plugin provides a toggle in the top bar that says `Show Table of Contents` if you click this toggle the table of contents will appear, take into account that the table of contents works with the regulatory statement mode of the article structure by default but this can be modified.

#### Formatting Toggle
This plugin provides a toggle in the top bar that says `Show formatting marks`, after this toggle is active all the formatting marks of the document such as break lines, paragraphs... will be visible.
![document with formatting annotations](https://imgur.com/KTNxuBW.png)

#### Rdfa Blocks Toggle
This plugin provides a toggle in the top bar that says `Show annotations`, when you click on this toggle the styling will change to show all the rdfa information contained in the document, this plugin is useful if you want to check for errors in your rdfa structure or just look to hidden information
![document with rdfa blocks visible](https://imgur.com/Asdu2aN.png)

### Enabling/disabling the environment banner
The environment banner is a visual indication of the environment you are currently using and which versions of embeddable, the editor and editor-plugins are in use.

You can enable/disable the banner using the following methods: `enableEnvironmentBanner` and `disableEnvironmentBanner`.

### Localization

Localization of the editor is an ongoing effort, the main target usage of embeddable is currently Dutch speaking users. Some plugins, like the [citation plugin](https://github.com/lblod/ember-rdfa-editor-citaten-plugin/), use date pickers. The display format of these dates can be configured in the localization initializer.

### Styling

Styling the editor is covered in the [README](https://github.com/lblod/ember-rdfa-editor#customisation) of ember-rdfa-editor. This frontend supports SASS, customizations can be added to [app.scss](app/styles/app.scss)

**The following section is automatically generated. It can provide usefull information on how to get started on development on this editor and with EmberJS.**

# Development of frontend-embeddable-notule-editor

This README outlines the details of collaborating on this Ember application.
A short introduction of this app could easily go here.

## Prerequisites

You will need the following things properly installed on your computer.

* [Git](https://git-scm.com/)
* [Node.js](https://nodejs.org/) (with npm)
* [Ember CLI](https://ember-cli.com/)
* [Mozilla Firefox](https://www.mozilla.org/en-US/firefox/) or [Google Chrome](https://google.com/chrome/)

## Installation

* `git clone <repository-url>` this repository
* `cd frontend-embeddable-notule-editor`
* `npm install`

## Running / Development

* `ember serve`
* Visit your app at [http://localhost:4200/test.html](http://localhost:4200/test.html).
* Visit your tests at [http://localhost:4200/tests](http://localhost:4200/tests).

### Code Generators

Make use of the many generators for code, try `ember help generate` for more details

### Running Tests

* `ember test`
* `ember test --server`

### Linting

* `npm run lint:hbs`
* `npm run lint:js`
* `npm run lint:js -- --fix`

### Building

* `ember build` (development)
* `ember build --environment production` (production)

### Deploying

Specify what it takes to deploy your app.

## Further Reading / Useful Links

* [ember.js](https://emberjs.com/)
* [ember-cli](https://ember-cli.com/)
* Development Browser Extensions
  * [ember inspector for chrome](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi)
  * [ember inspector for firefox](https://addons.mozilla.org/en-US/firefox/addon/ember-inspector/)

