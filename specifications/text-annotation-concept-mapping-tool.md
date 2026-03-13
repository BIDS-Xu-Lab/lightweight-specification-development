I want to create a text annotation tool called Blu Lite, which aims to support entity annotation, relation annotation, and concept normalization. Detailed specifications are as follows:

# Technical Requirements

- **Tech Stack**: Use `JavaScript`, `Bun`, `Vite`, `Vue.js` as the core stack. Use `PrimeVue` and `TailwindCSS` for frontend UI components. Use `Pinia` for state management.
- use free `font-awesome` for icons, register for vue app

```js
/* import the fontawesome core */
import { library } from '@fortawesome/fontawesome-svg-core'

/* import font awesome icon component */
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'

import { fas } from '@fortawesome/free-solid-svg-icons'
import { far } from '@fortawesome/free-regular-svg-icons'
import { fab } from '@fortawesome/free-brands-svg-icons'

/* add icons to the library */
library.add(fas, far, fab)
app.component('font-awesome-icon', FontAwesomeIcon)
```
and use icon like:

```html
<font-awesome-icon :icon="['fas', 'expand']" class="mr-1" />
```

- Blu Lite is a pure frontend application
- All user data is saved on user's local disk
- Blu Lite is a single page application with multiple views, and each view may focus on specific task
- Use `File System Access API` to enable read/save local files
- use vue router. for the router, please use `createWebHashHistory` to avoid server side config.
- Use `minisearch` for concept mapping, and user a seperate worker to perform index and search.
- Use `idb` for caching vocabulary data


# Layout design and general guideline

Blu Lite has several views, including:

- Landing view: Just a static homepage for showing features in one screen, make sure
- Annotation view: the is the main view of this app, providing file management, file and annotation workbench, schema editor, and other features for annotation. More details will be explained later.
- Schema editor view: this is for creating and editing a schema for annotation project.

## Basic layout
Except for landing view, all other views share a same UI layout:

- On the left is a full-height narrow bar including upper and lower part. the upper part contains buttons for navigation to different views. the lower part contains system setting and user profile
- On the right is the whole view. including three major parts: header, main section, and footer. The header is a full width, 3rem height menu bar. detailed functions should be defined in each view. in the middle the main section of a view. at the bottom is a 1rem height footer bar for view information. 

## Color Schema

- this app default to light theme only, not support dark theme.
- the main theme color is Yale blue

## Setting Panel

When click the Setting button, show a drawer panel from left with 400px width. Setting panel is multi tab (use PrimeVue TabList and TabPanel), contains:

- General
  - a switch for Annotation file auto-save feature, by default to false
- Agent
  - OpenAI type: OpenAI.com or Azure 
  - OpenAI base_url
  - OpenAI api_key
  - OpenAI model, default to gpt-5.1-codex-mini
- Vocabulary
  - statistics on the number of collections, number of total cached records.
  - a table of all pre-defined vocabularies from what passed from environment variables. in each row, on the left show the name of vocabulary and the number of records in the vocabulary, on the right show a status of "Included" or "Not Included". If included, a button of clear cache. If not included, a button of "Included". 
  - a table of user-uploaded cached vocabularies. in each row, on the left show the name of vocabulary and the number of records in the vocabulary, on the right just show "Remove".

## Schema editor view

This is for creating or modifying a annotation schema. The schema editor is always viewing the schema loaded in the Annotation view. A schema MUST be loaded from the annotation view and we need to save the `FileHandle` for updating the schema.

### Functions

- Create a new schema
- Edit the schema in use in the annotation view

### Menubar

- New. When clicking "New" schema, if this is already a schema loaded, please alert user that this will overwrite the current schema in use. If there is no in-use schema, just create a new one.
- Save. When saving the schema, must save back to the schema file and apply to what is using in the annotation view. if it's a new schema, ask user to save it somewhere and also save the FileHandle.

### Main section

- Two-column design, left column is all types, all Entities, all Relations. right column is a editor for the selected entity or relation.
- In Left column, for each row, left part is the entity/relation name, right part has two buttons, edit and delete

### footer

show the number of entity types and relation types.



## Annotation view

This is the main function for this app. 

### Functions

In this view, users: 

- MUST load a json format schema file
- MUST load a folder containing `.txt` or `.json` files using `window.showDirectoryPicker`, once selected, read all txt and json file in the folder, but not in the children folders. The files from the folder or user-specific are appended to the editing list.
- for `.txt` files, convert to the annotation json file using the same file name, and delete the `.txt` file
- for `.json` files, just read the content and ensure it match the format requirement (must be a object having `filename`, `content`, and `indexes`), if not match, skip
- once files are loaded, display the files in the file list panel
- when user click a file row, show the content and render the annotations in the editor panel

Users can annotate entities in the editor panel:

- In the editor panel, users can highlight a text and select what entity it is to annotate the highlighted text
- Once annotated a text, show the entity name below the annotated text and change the background color the annotated text
- If the annotated entity contains attributes, show the attribute name and value below the entity name
- Click on the entity name below the annotated text, a context menu shows on the right. The menu contains "Edit Attribute", "Concept Mapping", and "Delete"
- When user click "Edit Attribute", popup a panel showing all attributes of this entity, users can add/remove attribute key + value pair in this panel. In addition to the pre-defined attributes, user can also add other customize attributes.
- In the editor panel of the annotation view, the current entity name is display on the text right, please move the entity name to the text bottom, you can use a flex `<div>` to wrap annotated texts and the entity names and attributes.
- a text can be annotated multiple times and annotations can overlap with each other, so display the entities just all below the annotated text
- when hover a entity name below the annotated text, make the corresponding annotated text blinking background to make it easy to see which text is related, especailly when multiple entities are assigned to overlapped text.


Users can map entities and relations to a concept, i.e., concept normalization:

- Click on the entity label or the relation label, there is a "Concept Mapping" menu item. If this entity or relation is alraedy mapped, show the concept and vocabulary name, e.g., "Edit C1234560(SNOMED)" for this concept
- After clicking the "Concept Mapping" item, a floating concept mapping panel will show. this panel is not a modal, just a panel shows. this panel is about 300px width and min-height 400px.
- In the concept maping panel, the header shows the entity or relation information and current mapped concept, next row show a search input box and search button. below the search input box, the concept search results are listed, then the panel footer shows pagination and the number of matched results. each page of concept search results has 20 items, one item per row. In each row, show the concept id, concept name, concept description on the left, then a button "Select" on the right. If a concept has been selected, the button shows "De-select" and able to remove the mapping.
- The concept mapping will create two attributes for entity or relation if not available: `concept` and `vocabulary`. concept is the concept id for the selected concept and the vocabulary is where this concept from.

User can accept entity candidates based on an automatic annotate same word function:

* when annotating entities, create a dictionary of unique annotations of all loaded files. e.g., when loaded 10 json files and has 100 entities annotated, use the entities of all 10 files to create a dictionary of unique 80 entities.
* use this dictionary to find all exact same words in the current document. e.g., in a document, find 6 matches of 3 types of entities and they are not annotated as entity yet by any part. we call those newly identified entities are "Entity Candidate(s)" and we have a candidate list for the current document.
* for those matched Entity Candidate(s), if they are not annotated as entity yet, make them as a colored and dotted line. when clicking on those dotted text labels,  create a new entity annotation from this candidate and also remove it from the the candidate list.
* In the menu of annotation view, when user click "Automate", popup a small PrimeVue's Popover component to show how many candidates we found and a button to accept all of them. Once user click "Accept All", just make all candidates are annotated entities.

### menu bar

In the menu bar, provide the following function buttons. For every button, add a buttom tooltip (PrimeVue v-tooltip.button) show a simple help information.

- Schema. a box with a button to load an annotation schema, once loaded, show the annotation schema name
  - when loaded a schema, and if this schema has the guideline in json, show a switch button "Guideline"
  - when turn on the switch, show a 400px width guideline panel on the right of the editor panel and render the guideline content, which is a html string.
- File. a toggle menu that can open a TieredMenu, including Load File(s), Dump Files, Export
- Save all. a button to save all files in the file list panel. disable this button if no files.
- Search input for highlighting keywords. For the search function in the annotation, when typing keywords, after 1s, highlight texts yellow in the editor panel. also put a time icon at the end of the input box in the search, click this icon to clear keywords
- Magic Wand. Automatic annotating the current document. 
- a group of toggle buttons for status, such as show attributes, show annotation id
- a toggle menu "Samples" that loads annotation sample data for demonstration.
- a toggle menu "Help", contains "Quick Tour", "Tutorial" and "Wiki"

### main section

In the main section, a two-column layout. left column is the file list panel showing annotation files, right column is the editor panel for add/modify/delete entity and relation annotations.

- the file list panel supports paged list of files, each page contains 20 file rows. each row shows a file, the file name on the left and a group of buttons on the right. In the right button group, shows a save and delete button. the save button can save the annotations to local disk. the delete button remove this file from the list. In the file list panel header, show the number of total files and a dropdown for sorting by name
- the editor panel renders the text and the annotations

### footer

Display the following information in a line:

- Number of files, e.g., display as `Files: 24` 
- Number of chars of the opened file, e.g., `Chars: 123,231`
- Number of tokens
- Number of annotated entities


# Concept mapping Function

To support concept mapping, vocabulary data is needed, they are saved at the `public/data/vocab` folder and named as `VOCABULARY_NAME.tsv.zip`. Since the vocabulary file is large, if loaded, cache the data into indexeddb using `idb`. e.g., LNC vocabulary is saved as `LNC.tsv.zip` and cached in a `vocab-sys-LNC` collection in `idb`, for user-uploaded vocabulary, create a `vocab-user-XXX` collection `idb`. next time when app starts, load the specific cached vocabularies to create index. 

- All vocabulary tsv file follow the same schema: `concept_id`, `terms`, and `description`. `terms` contains concept name and synonums for a concept, joined with `|`. When indexing, just index `terms`. 
- When start the system, check if ICD10CM, LNC, and SNOMEDCT_US are cached. if not, for those are not cached, process one by one. load the data zip file, upzip it, save into cache by creating a collection using the the vocabulary name. If those are cached or loaded to cache, start indexing into minisearch in a seperate worker.
- for minisearch indexing, index the terms of all vocabularies together. On the annotation view, show a status of indexing at the footer right. e.g.,, "[icon] Indexing Vocabulary", "[icon] Vocabulary Index Ready".
- User can configure which vocabulary to be used for indexing, if not specified, just use the default ICD10CM and LNC. user's specification for indexing should be saved into localStorage and able to change in Settings panel.
- User can check statistics of vocabulary cache in the settings panel's Vocabulary tab. showing all cached vocabulary collection list and how many records in each collection. user can also clear the clear to load new one.
- User can upload a customized tsv file as vocabulary, e.g., USER_VOCAB.tsv, it has the same columns `concept_id`, `terms`, and `description`. Once uploaded, if there are no existing same name vocabulary, cache the uploaded vocabulary, and user can
- Due to the complexity of the indexing and size, the app should display how many records are included in the selected/cached vocabulary
- We will add more default vocabularies in the `public/data/vocab` folder, so when start/build the app, read the files end with `.tsv.zip` in the folder and pass to store through environment variable, so when we add more vocabulary tsv zip files, the app will know what are added automatically.


## Prepare sample data

Read `cui_sab_terms.tsv` use pandas, create a tsv file callded SNOMEDCT_US.tsv:

- contains columns concept_id, terms, description
- concept_id is the CUI column
- terms is the Terms column
- description is empty
- match all records by the SAB column

save the SNOMEDCT_US.tsv into `public/data/vocab`, similar, create another file called LNC.tsv, and ICD10CM.tsv



# Annotation Data Storage

Blu Lite mainly has two data types for annotation: annotation schema file, and annotation data file.

## Annotation Schema File

The schema is a JSON file that defines the annotation project. For example the following.

```json
{
  "name": "Project Name",
  "guideline": "<h1>Project Guideline</h1><p>Details ...</p>",
  "entity": [
  {
    "name": "disease",
    "attrs": [
      { "name": "concept", "value_type": "text", "values": [], default_value: "" }, 
      { "name": "comment", "value_type": "text", "values": [], default_value: "" }, 
      { "name": "certainty", "value_type": "enum", "values": ["positive", "negated", "possible", "NA"], default_value: "NA" }
    ]
  },
  {
    "name": "drug",
    "attrs": [
      { "name": "concept", "value_type": "text", "values": [], default_value: "" }, 
      { "name": "comment", "value_type": "text", "values": [], default_value: "" }, 
      { "name": "certainty", "value_type": "enum", "values": ["positive", "negated", "possible", "NA"], default_value: "NA" }
    ]
  }
  ],
  "relation": [{
    "name": "treatment",
    "from_entity": "drug",
    "to_entity": "disease",
    "attrs": []
  }]
}
```

## Annotation data file

Each document is saved as a json file with the following format.

- filename: the file path to the json file
- content: the full text to be annotated, usually imported through a txt file. all `\t` and `\n` are converted to ensure it can be saved in a string.
- indexes: all the annotation. this is a dictionary, the key is the text offset in the content, then the value is what is annotated starts from that offset, e.g., "Entity" means users annotate entities starts from this offset.
- Each annotated entity contains several values, such as the begin and end offset, attributes. 

```json
{
  "filename": "/home/user/sample.json", 
  "content": "the full content of the text to be annotated",
  "indexes": {
    "16": {
        "Entity": [
        {
          "semantic": "drug",
          "umlsConcepts": [],
          "begin": 16,
          "end": 26,
          "type": "Entity",
          "attrs": {
              "concept": {
                  "id": "1255296002433355776",
                  "attrKey": "concept",
                  "attrValue": "",
                  "attrType": "2",
                  "annotationValue": "C0853245",
                  "attrIcon": ""
              }
          }
        }
        ],
        "Token": [
        {
            "begin": 16,
            "end": 20,
            "type": "Token",
            "attrs": {}
        }
        ],
        "Sentence": [
        {
            "begin": 16,
            "end": 128,
            "type": "Sentence",
            "attrs": {}
        }
        ],
        "Relation": [
        {
          "semantic": "treatment",
          "type": "Relation",
          "umlsConcepts": [],
          "begin": 16,
          "end": 26,
          "fromEnt": {
            "begin": 16,
            "end": 26,
            "semantic": "drug",
            "type": "Entity",
            "umlsConcepts": []
          },
          "toEnt": {
            "begin": 72,
            "end": 81,
            "semantic": "disease",
            "type": "Entity",
            "umlsConcepts": []
          },
          "attrs": {
              "concept": {
                  "id": "1211296002433355445",
                  "attrKey": "concept",
                  "attrValue": "",
                  "attrType": "2",
                  "annotationValue": "C1233312",
                  "attrIcon": ""
              }
          }
        }
        ],
    },
    "72": {
        "Entity": [
        {
          "semantic": "disease",
          "umlsConcepts": [],
          "begin": 72,
          "end": 81,
          "type": "Entity",
          "attrs": {
              "concept": {
                  "id": "1244296001233355112",
                  "attrKey": "concept",
                  "attrValue": "",
                  "attrType": "2",
                  "annotationValue": "E71",
                  "attrIcon": ""
              }
          }
        }
        ],
        "Token": [
        {
            "begin": 72,
            "end": 78,
            "type": "Token",
            "attrs": {}
        }
        ],
        ]
    },
  }
}
```
