[![npm (scoped)](https://img.shields.io/npm/v/@gov-cy/dsf-email-templates)](https://www.npmjs.com/package/@gov-cy/dsf-email-templates)
![License](https://img.shields.io/github/license/gov-cy/dsf-email-template)
[![Unit test](https://github.com/gov-cy/dsf-email-template/actions/workflows/unit-test.yml/badge.svg)](https://github.com/gov-cy/dsf-email-template/actions/workflows/unit-test.yml)
[![tag-and-publish-on-version-change](https://github.com/gov-cy/dsf-email-template/actions/workflows/tag-and-publish-on-version-change.yml/badge.svg)](https://github.com/gov-cy/dsf-email-template/actions/workflows/tag-and-publish-on-version-change.yml)
[![tag-and-publish-on-version-change](https://github.com/gov-cy/dsf-email-template/actions/workflows/mailtrap-test.yml/badge.svg)](https://github.com/gov-cy/dsf-email-template/actions/workflows/mailtrap-test.yml)

Use this project to produce html email templates to be used by DSF. These emails should:

- Follow the HTML best practices for emails (for example max-width: 600px, use of inline styling)
- Minimize the risk of being stuck in “SPAM”
- Make sure it works well with most email clients
- Be responsive (work well on desktop and mobiles)
- Follow accessibility guidelines
- Allow multi-language content
- Include the gov.cy branding

The project uses [nunjucks](https://mozilla.github.io/nunjucks/) templates to built the email html.

The project also uses https://mailtrap.io/ for spam analysis and to verify HTML compatibility 

Compatibility with clients based on mailtrap test on 2024-04-10

| Client     | Desktop | Mobile | Web  | 
| ------     | ------- | ------ |----- |
| Apple mail | 97%     |  95%   | 100% |
| Gmail      | 100%    |  89%   | 92%  |
| Outlook    | 80%     |  93%   | 94%  |
| Yahoo Mail | 100%    |  80%   | 80%  |
| Other      | 100%    |  89%   | 89%  |

Though the project is not intended to be used for production purposes and does provide any guaranties, your welcome to try it.

> [!WARNING]  
> The project allows you to input un-escaped HTML in your emails, so use it with care.

## Features

`DSF-email-templates` can:
- Generate email HTML programmatically from input nunjucks template, using the project's base template and macros.
- Generate email HTML programmatically from input JSON data
- Generate email HTML from the command line 

## Install

First, install the DSF Email Templates package using npm:

```shell
npm install @gov-cy/dsf-email-templates
```

## Using it Programmatically

You can use the `DSFEmailRenderer` class to render email templates programmatically in your Node.js applications. 

The class allows you to render HTML email using various input sources such as nunjucks template or JSON objects. 

### Method 1: Using nunjucks template for input

You can use nunjucks template as input to render the desired HTML email using DSF base template and macro components.

Render email HTML directly from a string template using the `renderFromString` method. Here's an example:

```js 
import { DSFEmailRenderer } from '@gov-cy/dsf-email-templates';

// Create an instance of DSFEmailRenderer
const renderer = new DSFEmailRenderer();

const inputString = `
{% extends "govcyBase.njk" %}
{% from "govcyEmailElement.njk" import govcyEmailElement %}
{% set lang='en'%}
{% block lang %}{{lang}}{% endblock %}
{% block subject %}Service email{% endblock %}
{% block header -%}{{ govcyEmailElement ('header',{serviceName:'Service name', name:'First and Last name',lang:lang}) }}{%- endblock %}
{% block body -%}
    {% call govcyEmailElement('body') -%}
        {% call govcyEmailElement('bodyHeading',{headingLevel:1}) -%}Heading 1{%- endcall %}
        {% call govcyEmailElement('bodyParagraph') -%}Paragraph{%- endcall %}
        {% call govcyEmailElement('bodyParagraph',{"lang":"el"}) -%}Παράγραφος{%- endcall %}
    {% endcall %}
{% endblock %}
{% block footer -%}
    {{ govcyEmailElement ('footer',{footerText:'Service name'}) }}
{%- endblock %}
`;

//Render the email template 
const renderedTemplate = renderer.renderFromString(inputString);

console.log(renderedTemplate); // Output the rendered template
```

You may also use load a template from a file using `fs/promises` and provide a text file with the nunjucks input template as follows:

```js
import DSFEmailRenderer from '@gov-cy/dsf-email-templates';
import fs from 'fs/promises';

// Create an instance of DSFEmailRenderer
const renderer = new DSFEmailRenderer();

// Specify the path of the template file
const templatePath = 'path/to/template.njk';

// Load template file
const templateContent = await fs.readFile(templatePath, 'utf8');
// Render template
const renderedTemplate = renderer.renderFromString(templateContent);

console.log(renderedTemplate); // Output the rendered template

```

#### Nunjucks template with DSF assets

To use the DSF base template start with the following line in your template:

```njk
{% extends "govcyBase.njk" %}
```

You can then use various the base template's blocks to built the structure of the email. The available blocks are:
- **lang**: Sets the lang html property
- **subject**: Set's the html title. Usually the same as the intended email subject
- **pre**: Pre header text that is invisible in the html body 
- **header**: The header of the email with the gov.cy logo and name of the service or site
- **body**: The main body of the email. 
- **footer**: The footer of the email

Here's an example using the blocks of the base template. Note that this will only render the HTML email structure and not the UI components.

```njk
{% extends "govcyBase.njk" %}
{% set lang='el'%}
{% block lang %}{{lang}}{% endblock %}
{% block subject %}Service email{% endblock %}
{% block pre -%}The pre header text'{%- endblock %}
{% block header -%}The header{%- endblock %}
{% block body -%}The Body{% endblock %}
{% block footer -%}The footer{%- endblock %}
```

To render the UI components use the `govcyEmailElement` macro. To do that first you need to import the macro by adding the following line:

```njk
{% from "govcyEmailElement.njk" import govcyEmailElement %}
```

To use the `govcyEmailElement` macro you need to define the component name and it's parameters. The available components the following:

<details>
  <summary>preHeader</summary>
  
Returns govcy pre header for emails html. It should be used in the `pre` block. 

**Parameters**
- {string} `preText` The pre header text. Will escape text 

Usage example: 

```njk
{{ govcyEmailElement 
    (
        'preHeader',
        {
            preText:'The pre header text'
        }
    ) 
}}
```
</details>

<details>
  <summary>header</summary>
  
Returns govcy header for emails html. It should be used in the `header` block. 

**Parameters**
- {string} `serviceName` The service name. Will escape text
- {string} `name` The name used in salutation. Will escape text 
- {string} `lang` The language used. Can be 'en','el'. Optional.

Usage example: 

```njk
{{ govcyEmailElement 
    (
        'header',
        {
            serviceName:'Service name', 
            name:'First and Last name',
            lang:'en'
        }
    ) 
}}
```
</details>

<details>
  <summary>body</summary>
  
Returns govcy body for emails html. It should be used in the `body` block. Accepts no parameters and will return the content available inside the macro.

Usage example: 

```njk
{% call govcyEmailElement('body') %} 
    BODY 
{% endcall%}
```
</details>

<details>
  <summary>bodySuccess</summary>
  
Returns govcy success panel for emails html. It should be used in the `body` block. 

**Parameters**
- {string} `title` The success title text. Will escape text 
- {string} `body` The success body text. Will escape text 
- {string} `lang` The language used. Can be 'en','el'. Optional.

Usage example: 

```njk
{{ govcyEmailElement 
    (
        'bodySuccess',
        {
            title:'title part', 
            body:'body part'
        }
    ) 
}}
```
</details>

<details>
  <summary>bodyHeading</summary>
  
Returns govcy an h1, h2, h3 or h3 for emails html. It should be used in the `body` block. It returns the content available inside the macro.

**Parameters**
- {string} `headingLevel` The heading level. Optional, default is 1. Can be 1,2,3,4
- {string} `lang` The language used. Can be 'en','el'. Optional.

Usage example: 

```njk
{% call govcyEmailElement('bodyHeading',{headingLevel:2,lang:'en'}) -%}
    Heading 2
{%- endcall %}
```
</details>

<details>
  <summary>bodyParagraph</summary>
  
Returns govcy a paragraph. It should be used in the `body` block. It returns the content available inside the macro.

**Parameters**
- {string} `headingLevel` The heading level. Optional, default is 1. Can be 1,2,3,4
- {string} `lang` The language used. Can be 'en','el'. Optional.

Usage example: 

```njk
{% call govcyEmailElement('bodyParagraph',{lang:'en'}) -%}
    Lorem ipsum
{%- endcall %}
```
</details>

<details>
  <summary>bodyList</summary>
  
Returns govcy an ordered or un-ordered list. It should be used in the `body` block. 

**Parameters** 
- {string} `type` The list type. Optional, default is 'ul'. Can be 'ul', 'ol' 
- {array} `items` The array of items to turn onto list items 
- {string} `lang` The language used. Can be 'en','el'. Optional.

Usage example: 

```njk
{{ govcyEmailElement ('bodyList',{type:'ol', items: ["item 1", "item 2", "item 3"]}) }}
{{ govcyEmailElement ('bodyList',{type:'ul', items: ['<a href="#">item 1</a>', "item 2", "item 3"]}) }}
```
</details>

<details>
  <summary>bodyKeyValue</summary>
  
Returns govcy an ordered or un-ordered list with key and value. It should be used in the `body` block. 

**Parameters** 
- {string} `type` The list type. Optional, default is 'ul'. Can be 'ul', 'ol' 
- {array} `items` The array of items to turn onto list items. 
  i.e. items: 
    [
        {key:'key1', value:'value 1'}, 
        {key:'key2', value:'value 2'}
    ]
- {string} `lang` The language used. Can be 'en','el'. Optional.

Usage example: 

```njk
{{ govcyEmailElement ('bodyKeyValue',{type:'ul', items: [ {key:'key1', value:'value 1'}, {key:'key2', value:'value 2'} ]}) }}
```
</details>

---------

Here's a complete example of a nunjucks template using DSF assets:

```njk
{# extend the base html #}
{% extends "govcyBase.njk" %}
{# import components #}
{% from "govcyEmailElement.njk" import govcyEmailElement %}
{# set language #}
{% set lang='el'%}
{# set the blocks of the base template  #}
    {# language block  #}
    {% block lang %}{{lang}}{% endblock %}
    {# subject block #}
    {% block subject %}Έκδοση πιστοποιητικού γέννησης{% endblock %}
    {# header block #}
    {% block header -%}
        {# use the header component #}
        {{ govcyEmailElement ('header',{serviceName:'Έκδοση πιστοποιητικού γέννησης',lang:lang}) }}
    {%- endblock %}
    {# success block #}
    {# body block #}
    {% block body -%}
        {# use the body component   #}
        {% call govcyEmailElement('body') -%}
            {# use the email success component #}
            {{ govcyEmailElement ('bodySuccess',
            {
                title:'Το πιστοποιητικό γέννησης που εκδώσατε είναι έτοιμο', 
                body:'Αριθμός αναφοράς 123456'}
            ) }}
            {# use combinatopn of body components to complete the body #}
            {% call govcyEmailElement('bodyParagraph') -%}
                Η ημερομηνία έκδοσης είναι 1/1/2019 και ο αριθμός αναφοράς σας είναι 123456.
            {%- endcall %}
            {{ govcyEmailElement ('bodyList',
                {type:'ol', 
                    items: [
                        "<b>Ονοματεπώνυμο</b>: Όνομα Επώνυμο", 
                        "<b>Αριθμός ταυτότητας</b>: 123456", 
                        "<b>Ημερομηνία έκδοσης</b>: 1/1/2019"
                        ]
                    }) 
            }}
            {% call govcyEmailElement('bodyParagraph') -%}
                Μπορείτε να τα κατεβάσετε στη σελίδα <a href="#">Τα έγγραφά μου</a> μαζί με όσα έγγραφα έχετε εκδώσει.
            {%- endcall %}
            {% call govcyEmailElement('bodyParagraph') -%}
               Στη σελίδα <a href="#">Οι συναλλαγές μου</a> μπορείτε να βρείτε την απόδειξη πληρωμής και τις συναλλαγές σας.
            {%- endcall %}
            {% call govcyEmailElement('bodyParagraph') -%}
               Αν έχετε ερωτήσεις, επικοινωνήστε με το Τμήμα Αρχείου Πληθυσμού και Μετανάστευσης στο migration@crmd.moi.gov.cy. Συμπεριλάβετε τον αριθμό αναφοράς 123456.
            {%- endcall %}
            {% call govcyEmailElement('bodyParagraph') -%}
               Μην απαντήσετε σε αυτό το email.
            {%- endcall %}
           
        {%- endcall%}
    {%- endblock %}
    {# footer block #}
    {% block footer -%}
        {# use the footer component #}
        {{ govcyEmailElement ('footer',{footerText:'Έκδοση πιστοποιητικού γέννησης'}) }}
    {%- endblock %}
```

### Method 2: Using JSON object for input

You can use JSON object as input to render the desired HTML email using the DSF base templare and macro components, by using the the `renderFromJson` method. Here's an example:

```js
import { DSFEmailRenderer } from '@gov-cy/dsf-email-templates';

// Create an instance of DSFEmailRenderer
const renderer = new DSFEmailRenderer();

const inputJson={
    lang: "el",
    subject: "Service email",
    pre: "The pre header text",
    header: {serviceName:'Service name', name:'First and Last name'},
    success: {
        title:'title part',
        body:'body part'
    },
    body: [
        {"component": "bodySuccess", params:{title:'title part'}, body:'body part'},
        {"component": "bodyHeading",params: {"headingLevel":1},body:"Heading 1"},
        {"component": "bodyParagraph", params:{lang:"en"}, body:"Paragraph"},
        {"component": "bodyList", params:{type:"ol", items: ["item 1", "item 2", "item 3"]}}
    ],
    footer: {
        footerText: "Name of service"
    }
}

//Render the email template 
const renderedTemplate = renderer.renderFromJson(inputJson);

console.log(renderedTemplate); // Output the rendered template
```

## Using it from Command line

Use `dsf-email-templater` to built the email html file by defining the input and output as arguments. 

The basic usage syntax is:

```shell
dsf-email-templater <input.njk> <output.html>
```

Replace `<input.njk>` with the path to your Nunjucks template file and `<output.html>` with the desired path for the rendered HTML output file.

For example:

```shell
dsf-email-templater .\src\built-with-nunjucks.js .\src\njk\templates\submission-email.njk .\build\submission-email.html
```

Note for non-Windows users: you may need to replace regular slashes ( `\` ) with backslashes ( `/` ) in all the paths of the above commands.

## License

The package is released under the [MIT License](https://opensource.org/licenses/MIT).

## Contact

If you have any questions or feedback, please feel free to reach out to us at [dsf-admin@dits.dmrid.gov.cy](mailto:dsf-admin@dits.dmrid.gov.cy)

