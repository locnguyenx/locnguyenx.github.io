---
layout: post
title: Moqui Screen and Form
permalink: /moqui-screen-form/
author:    locnx
tags: moqui xmlscreen xmlform
category: Moqui
---

# TOC
<!-- MarkdownTOC depth="2" -->

- [Overview](#overview)
- [Working with XMLScreen](#working-with-xmlscreen)
    - [Form validation](#form-validation)
    - [Form extends](#form-extends)
    - [Manipulate Error Message](#manipulate-error-message)
    - [Build a complex screen](#build-a-complex-screen)
    - [Reuse screen components](#reuse-screen-components)
- [Brief of Screen components](#brief-of-screen-components)
    - [screen \(root\)](#screen-root)
    - [transition](#transition)
    - [subscreens](#subscreens)
    - [subscreens-menu](#subscreens-menu)
    - [widgets](#widgets)
- [Brief of Form elements](#brief-of-form-elements)
    - [Common elements](#common-elements)
    - [form-single](#form-single)
    - [form-list](#form-list)

<!-- /MarkdownTOC -->


# Overview

This document will focus to design Moqui XMLScreen including Form, manipulation Screen flow & prensentation logic.

__Related Schemas__

    - common-types-2.0.xsd: http://moqui.org/xsd/common-types-2.0.xsd
    - xml-actions-2.0.xsd: http://moqui.org/xsd/xml-actions-2.0.xsd
    - xml-form-2.0.xsd: http://moqui.org/xsd/xml-screen-2.0.xsd

# Working with XMLScreen

## Form validation

### Processing Rules

__By Service:__

System has 2 options to validate the form fields:

1. If __transition__ of the form has a `singleServiceName` (has <service-call> tag, instead of multiple service call in <actions> tag)
    - If `singleServiceName` is used-defined type, system will validate form field against respected in-paramater of `singleServiceName` 
    - Otherwise (`singleServiceName` is entity-auto type), will validate against Entity fields
    
2. Otherwise, use <auto-fields-service service-name='name'> definition of the form
    Add more attribute into field definition:
    - validate-service: service-name
    - validate-parameter: name of respected service parameter

Notes: if service-name is a Entity-Auto service, then the rule of `auto-fields-entity` will be used

__By Entity:__

Using following tag inside the form definition:
< auto-fields-entity entity-name='Name of Entity to generate fields automatically for form'/>

Underlying, system will addd more attribute into field definition:
    - validate-entity: entity-name
    - validate-parameter: fullname of respected Entity field

__How it works:__

Processing is done in `ScreenForm.groovy`

1. While rebuild __form definition__, system will check if either following tag is existing: __auto-fields-service__/__auto-fields-entity__, then add attribute `validate-service` or `validate-entiy` respectly into each __field definition__

2. Checking form __transition__: populate __validate-service__ and __validate-entity__ attributes if the target transition calls a single service, override attributes from previous steps
    1. if the field matches an in-parameter name, then set:
```
fieldNode.attributes().put("validate-service", singleServiceName)
```
    2. if transition service is AutoEntityService, if the field matches an entity field name then set it:
```
fieldNode.attributes().put("validate-entity", entityName)
```


3. Build field validate Node 
Get Node information for validation rule from corresponding field in service/ entity and copy into current form field.
Method: MNode getFieldValidateNode(String fieldName) 
for Service: 
    `Node parameterNode = sd.getInParameter((String) fieldNode.attribute('validate-parameter') ?: fieldName)`
for Entity: 
    `Node efNode = ed.getFieldNode((String) fieldNode.attribute('validate-field') ?: fieldName)`


### Example

__Sample form definition__

```xml
<container-dialog id="CreateTutorialDialog" button-text="Create Tutorial">
    <form-single name="CreateTutorial" transition="createTutorial">
        <auto-fields-entity entity-name="tutorial.Tutorial" field-type="edit"/>
        <field name="submitButton">
<default-field title="Create"><submit/></default-field>
        </field>
    </form-single>
</container-dialog>
```

__Case 1:__

use auto entity-auto service call: will check against ENTITY definition rules

```xml
<transition name="createTutorial">
    <service-call name="create#tutorial.Tutorial"/>
    <!--<service-call name="tutorial.TutorialServices.create#Tutorial"/>-->
    <default-response url="."/>
</transition>
```

Result: PK ID requried

__Case 2:__

use user defined service call: Field validation will depend on service call. Hence, the user-defined services must implement parameters validation.

```xml
<transition name="createTutorial">
    <!--<service-call name="create#tutorial.Tutorial"/>-->
<service-call name="tutorial.TutorialServices.create#Tutorial"/>
    <default-response url="."/>
</transition>
```

Service definition:

```xml
<service verb="create" noun="Tutorial" type="inline">
    <in-parameters>
        <auto-parameters include="all"/> <!-- PK (tutorialId) is not required in parameters-->
    </in-parameters>
    <out-parameters>
        <auto-parameters include="pk" required="true"/>
    </out-parameters>
    <actions>
        <entity-make-value entity-name="tutorial.Tutorial" value-field="tutorial"/>
        <entity-set value-field="tutorial" include="all"/>
        <if condition="!tutorial.tutorialId">
<entity-sequenced-id-primary value-field="tutorial"/>
        </if>
        <entity-create value-field="tutorial"/>
    </actions>
</service>
```

Result: PK ID not requried

## Form extends

Enxtends an existing form

__Syntax:__

- From external screen: `extends="<screen path>#<form name>”`
- From same screen: `extends="<form name>”`

__Example:__

Existing form: defined in file component/SimpleScreens/template/party/PartyForms.xml

```xml
<form-single name="ContactInfo" transition="storeContactInfo" map="contactInfoFlat">
    <field name="partyId"><default-field><hidden/></default-field></field>

    <field name="postalContactMechId">
        <conditional-field condition="postalContactMechPurposeId"><hidden/></conditional-field>
        <default-field><ignored/></default-field>
    </field>
…
</form-single>
```

New form:

```xml
<form-single name="BillingInfo" transition="storeContactInfo" map="billingInfoFlat"
        extends="component://SimpleScreens/template/party/PartyForms.xml#ContactInfo"/>
(defind new fields, or overrided fields)
</form-single>
```

## Manipulate Error Message

Return Message/Error message to Screen In service implementation (<actions>), or screen ‘s transition actions:

```xml
<actions>
...
<message error="true"> Message to be displayed </message>
</actions>
```
`error`: If `true` will be considered caused by an error, meaning transaction will be rolled back, etc.

__Usage:__ 
Should use error="true" at middle of processing to rollback the transaction. If the check/validation is at the beginning of screen transition, error="false" is enough

__Implementation in service:__
    eci.message.addValidationError

__Display message in Screen:__

in apps.xml

```xml
<section name="MessagesSection">
    <widgets>
        <section-iterate name="headerMessages" list="ec.message.messages" entry="message">
            ...
        </section-iterate>
        <section-iterate name="headerErrors" list="ec.message.errors" entry="errorMessage">
            ...
        </section-iterate>
        <section-iterate name="headerValidationErrors" list="ec.message.validationErrors" entry="validationError">
            <!-- TODO: put these by fields instead of in header -->
        </section-iterate>
...
```


## Build a complex screen

### Problem 1 - A menu with many independent screens

#### Problem Description
We needs to build a Menu "Tools" with has following sub menu, each will be a completed screen:

- Wiki
- Entity View

#### Solution
File structure (file without extension is folder)

```
- Tools
---- Wiki.xml
---- EntityView.xml
- Tools.xml
```

__Main Screen design:__ 

> __Important:__
> 
> - Atrribute `default-menu-include="true"` (default value) will display main screen _Tools_ in master menu 
> - Widget element `<subscreens-panel type="popup"/>` is required to render the subscreens as popup menu

Tools.xml

```xml
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-2.0.xsd"
        default-menu-title="Tools" default-menu-index="1">
    <subscreens default-item="Wiki"/>
    <widgets>
        <subscreens-panel id="tools-panel" type="popup"/>
    </widgets>
</screen>
```

__Sub Screen design:__

> __Important:__
> 
> - Atrribute `default-menu-include="true"` (default value) will display sub screen, i.e _Wiki_, inside the menu _Tools

Wiki.xml

```xml
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-2.0.xsd" default-menu-title="Wiki" default-menu-include="true" default-menu-index="1">
<!-- define screen components if required here-->
</screen>
```

EntityView.xml

```xml
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-2.0.xsd" default-menu-title="Entity View" default-menu-include="true" default-menu-index="2">
<!-- define screen components if required here-->
</screen>
```

### Problem 2 - A search screen and multi-tabs screen for each record

#### Problem Description
We needs to build a screen which Search/List accounts and a screen "Account" with has 3 tabs:

- Account main info
- All contacts of this account
- List of Account's Addresses

#### Solution
File structure (file without extension is folder)

```
- Account
---- FindAccount.xml
---- EditAccount.xml
---- Contact.xml
---- Address.xml
- Account.xml
```

__Main Screen design:__ 

> __Important:__
> 
> - Atrribute `default-menu-include="true"` (default value) will display main screen _Account_ in master menu 
> - Widget element `<subscreens-panel type="tab"/>` is required to render the subscreens, which are defined to be displayed in menu, as tabs of one screen

Account.xml

```xml
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-2.0.xsd" default-menu-title="Account" default-menu-index="1">
    <subscreens default-item="FindAccount"/>
    <!-- show Search Screen first -->
    <widgets>
        <subscreens-panel id="Account-panel" type="tab"/>
        <!--required to render the subscreens, which are defined to be displayed in menu, as tabs of one screen-->
    </widgets>
</screen>
```

__Sub Screen design:__

> __Important:__
> 
> - the default screen (FindAccount.xml) isn't needed to be displayed in the Menu or the Tabbed Screen. Hence, its definition has atrribute `default-menu-include="false"`
> - Atrribute `default-menu-include="true"` (default value) will display all sub screens as a Tabbed Screen, i.e _Account/ Contacts/ Addresses_, inside the menu _Account_

FindAccount.xml

```xml
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-2.0.xsd" default-menu-title="Find Accounts" default-menu-include="false">
<!-- define screen components if required here-->
</screen>
```

EditAccount.xml

```xml
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-2.0.xsd" default-menu-title="Main Account Info" default-menu-include="true" default-menu-index="1">
<!-- define screen components if required here-->
</screen>
```

Contact.xml

```xml
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-2.0.xsd" default-menu-title="Contacts" default-menu-include="true" default-menu-index="2">
<!-- define screen components if required here-->
</screen>
```

Address.xml

```xml
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-2.0.xsd" default-menu-title="Addresses" default-menu-include="true" default-menu-index="3">
<!-- define screen components if required here-->
</screen>
```


## Reuse screen components

### Overview
- subscreens-item
- include-screen
- form extends: see [Form extends](#form-extends)
- transition-include
- section-include
- Include a widget-template
- Include a html in widget

### subscreens-item

We can reuse a existing Screen without explicit redeisng it (or copy/paste). 

After declaration the subscreen, we can use it in transition's url or render it to display as normal:

```xml
<subscreens-item name="ScreenName" location="resource location"/>
```
Example: `<subscreens-item name="UpdateContactInfo" location="component://SimpleScreens/screen/SimpleScreens/Party/EditParty/UpdateContactInfo.xml"/>`

### section-include

### transition-include

### Include a widget-template

Template Definition

```xml
<widget-template name="enumDropDown">
        <drop-down allow-empty="${allowEmpty ?: 'false'}" no-current-selected-key="${noCurrentSelectedKey?:''}"
    style="${style?:''}" allow-multiple="${allowMultiple ?: 'false'}">
<entity-options key="${enumId}" text="${description}">
    <entity-find entity-name="moqui.basic.Enumeration">
        <econdition field-name="enumTypeId"/>
        <order-by field-name="description"/>
    </entity-find>
</entity-options>
        </drop-down>
</widget-template>
```

Using template

```xml
<widget-template-include location="component://webroot/template/screen/BasicWidgetTemplates.xml#enumDropDown">
    <!-- assign variables used in template if required -->
    <set field="enumTypeId" value="ProductPriceType"/>
    <set field="allowEmpty" value="true"/><set field="style" value=" "/>
</widget-template-include>
```

__Include a html in widget__

```xml
<widgets><render-mode><text type="html" location="component://SimpleScreens/template/party/ContactInfo.html.gstring"/></render-mode></widgets>
```

# Brief of Screen components

## screen (root)

### Attributes
- __require-authentication__: type=true/false/anonymous-all/anonymous-view default="true"
- __include-child-content__: _will check_
- __standalone__: If set to true this screen will be rendered without rendering any parent screens. It can still be referred to as a subscreen of its parent, but when rendered the parent will not run, the rendering will start at this screen. Any non-standalone children will still be treated as normal subscreens.
- __default-menu-title__
- __default-menu-index__
- __default-menu-include__: type="boolean" default="true"
    Set this to false to not automatically appear in the parent's subscreens menu based on the directory it is in. If true this screen will automatically be included in the parent's subscreens menu.
- __menu-image__
- __menu-image-type__

### Children elements

#### parameter

- minOccurs="0" maxOccurs="unbounded"
- documentation: 
    These are the parameters the screen expects or allows to be passed in, and where the calling screen can get them from by default (usually just default to the same from, but can be a static value for default or whatever).
    Individual transition, transition.*-response and other elements can override where the parameter comes from with their own parameter sub-elements.

#### always-actions

- minOccurs="0"
- documentation:
    These actions always run when this screen appears in a screen path, including both screen rendering and transition running. One difference between this and the pre-actions element is that this runs before transitions are processed while pre-actions do not. The always-actions also run for all screens in a path while the pre-actions only run for screens that will be rendered.

#### pre-actions

- minOccurs="0"
- documentation:
    These actions run before any of the screens (this screen or any parent screens) are rendered, allowing you to set parameters used by parent screens or other general reasons.

#### actions

- minOccurs="0"

#### subscreens

#### transition

#### transition-include

## transition

### Attributes
- __name__ 
    Transition names should be camel-cased and start with an lower-case letter (whereas screen filenames and subscreens-item names start with a upper-case letter).

    The transition name is used in link and other elements in place of URLs when going to another screen within this application. The transition name will appear briefly as the URL before the redirect is done for the transition response.

- __method__ any/put/get/post/delete
- __begin-transaction__ type="boolean" default="true"
- __read-only__ type="boolean" default="false"
    Declare that this transition does only read operations to skip the check for insecure parameters.

- __require-session-token__ default="true" type="boolean"
    If not false (default true) moquiSessionToken (from ec.web.sessionToken) must be passed to this transition for all requests in a session after the first.

### Children elements
#### parameter
These are the parameters the transition expects or allows to be passed in, and where the calling screen can get them from by default (usually just default to the same from, but can be a static value for default or whatever).

These are in addition to the screen.parameter values. Individual transition.*-response and other elements can override where the parameter comes from with their own parameter sub-elements.

#### path-parameter
These are additional path elements after the transition's path element. The values will be added to the web parameters based on the order of these path-elements.

#### condition
This condition is run wherever this transition is referenced in the screen to see if the transition is available (otherwise the button/link/etc is disabled).

#### service-call
NOTE: service-call & actions are mutual exclusive
In most cases the best way to handle input for a transition is with a single service. To do that use this element instead of an actions element.

This will automatically have an in-map=true. To get the same effect inside the actions element just use in-map=true.

#### actions
NOTE: service-call & actions are mutual exclusive

When this transition is followed these actions are run.

After the actions are run it goes to the url that this transition goes to (through client-side redirect, dynamic update of a screen area, etc).

#### conditional-response
If there are multiple transition-response sub-elements the first one whose condition evaluates to true will be the one used. If no conditional responses match, the default-response will be used.

#### default-response
This response must always be defined and is the response that will be used if there is no error in the actions, and if none of the conditions in conditional responses evaluate to true.

#### error-response
If there is an error in evaluating the actions on this transition then the error-response will be used and the transition-response element(s) will be ignored.

If there are actions and there is no error-response defined then the default error response will be used.

## subscreens

Declare subscreens for this screen. One subscreen at a time is active, based on the "screen path" used to access this screen. The parent screen (this screen) will be the current element in the screen path and the next screen path element will be the name of the subscreen of this screen to use.

If there is no additional element in the screen path or the next element is not a valid subscreen-item.name
then the default-item will be the active subscreen. 

There are three ways to add subscreens to a screen:

1. for screens within a single application:
   by directory structure: create a directory in the directory where the parent screen is named the same as
   the parent screen's filename and put XML Screen files in that directory (name=filename up to .xml,
   title=screen.default-title, location=parent screen minus filename plus directory and filename for
   subscreen)
2. for including screens that are part of another application, or shared and not in any application:
   subscreens-item elements below the screen -> subscreens element (this element)
3. for adding screens, removing screens, or changing order and title of screens to an existing application:
   a record in the moqui.screen.SubscreensItem entity

There are two visual elements (widgets) that come from the subscreens, a menu and the active subscreen.
Those are included with the widgets using the "subscreens-menu" and "subscreens-active" elements, or the
"subscreens-panel" element.

### Attributes
- __default-item__ 
    The name of the default subscreen-item. Used when then screen-path ends on this screen so we know which subscreen-item to activate.

    If empty the first subscreen-item will be the default.

### Children elements

#### subscreens-item
     One way to add a subscreen. This is most commonly used to refer to a subscreen that is located in another application, another part of this application, that is not in any application and is meant to be shared, or is in a different type of location than the parent screen.

    One subscreens-item is active at a time, meaning that screen is shown and the tab/etc for that screen is highlighted.

#### conditional-default
    use a `condition`, which is a Groovy condition expression (evaluates to a boolean) used to determine if the specified subscreens item is the one to use by default instead of the on specified in the subscreens.@default-item attribute.

## subscreens-menu

### Attributes
- __type__  tab/ popup/ popup-tree, default="tab"
- __id__ use="required"
- __title__
- __width__
- __header-menus-id__ type="xs:string" default="header-menus"

## widgets

### Attributes
- __name__ 

### subscreens-active

### subscreens-panel

### include-screen
- __location__ required
- __share-scope__ default="false" type="boolean"

### render-mode
  
- Include a text & render screen. Currently, there's only 1 child element: text.

__Attributes of text__

- __type__ default="any"
    Can be anything. Default supported values include: text, cwiki, html, xsl-fo, xml, and csv.A value of "any" will cause it to be used if no other element matches the current output type.
- __location__ This is the template or text file location and can be any location supported by the Resource Facade including file, http, component, content, etc.
- __template__ type="boolean" default="true"
    Interpret the text at the location as an FTL or other template? Supports any template type supported by the Resource Facade.
    Defaults to true, set to false if you want the text included literally.

- __encode__ default="false" type="boolean"
    If true the text will be encoded so that it does not interfere with markup of the target output. Templates ignore this setting and are never encoded.
    For example, if output is HTML then data presented will be HTML encoded so that all HTML-specific characters are escaped.

### section
- __name__ >A name for the section, used for reference within the screen. Must be specified and must be unique within the screen
- __condition__ A condition expression, just like the section.condition.expression element but more concise.

### section-iterate
- __name__ >A name for the section, used for reference within the screen. 
- __list__ The name of the field that contains the list to iterate over.
- __entry__ The name of the field that will contain each entry as we iterate through the list.
- __key__ If list points to a Map or List of MapEntry the key will be put where this refers to, the value where the entry attribute refers to.
- __condition__ A condition expression, just like the section-iterate.condition.expression element but more concise.

### section-include
- __name__ >A name for the section, used for reference within the screen. Must be specified and must be unique within the screen
- __location__ Location of the screen containing the section to include. Example: `location="component path to ScreenName.xml#SectionNameToBeIncluded"`

#### container

#### container-box

#### container-row
    A responsive 12-column grid row. For the concept and one possible implementation see http://getbootstrap.com/css/#grid

### container-panel
This panel can have up to five areas: header, left, center, right, footer. Only the center area is required. This can be re-used within the different areas as well, usually just the center area but could be used to split up even the header and footer.

If there is an id for the outer container, and each area will have an automatic id as well (with a suffix of: _header, _left, _center, _right, _footer).

___Attributes:___ 
- __panel-center__ required
- __dynamic__ default="false" type="boolean". When true uses a dynamic layout, by default with jQuery Layout (see http://layout.jquery-dev.net/). When false (default) uses a static HTML/CSS layout.

### container-dialog
The contents start out hidden with only a button with the button-text on it. When the button is clicked on a dialog opens to show the contents.

### tree

# Brief of Form elements

## Common elements
These elements are used in both `form-single` and `form-list`

### auto-fields-entity

Automatically generate list of fields based on entity definition.

>When a form has _dynamic=true_ and a _${}_ string expansion in the _auto-field-sentity.entity-name_ attribute then it will be expanded on the fly as the screen is rendered, meaning a single form can be used to generate tabular HTML or CSV output for any entity given an entity name as a screen parameter.

__Attributes__
- __entity-name__ required
- __field-type__ default="find-display", value in [edit, find, display, find-display, hidden]
- __include__ default="all", value in [pk, nonpk, all]
    type of entity fields to be included in the form

### auto-fields-service

> _Notes_: 
> One important note about forms based on a service (using the _auto-fields-service_
> element) is that various client-side validations will be added automatically based on the
> validations defined for the service the form field corresponds to.

This is only true if __service-name__ is a user defined serivce, otherwise (if this is entity-auto service) the rule will be applied as if it's auto-fields-entity: includes all fields of ENTITY, but in `edit` mode

__Attributes__
- __service-name__ required
- __field-type__ default="edit", value in [edit, find, display, find-display, hidden]
- __include__ default="in", value in [in, out, all]
    type of service parameters to be included in the form

### field


## form-single
A single form is used to view or edit fields of a single map/hash/record/etc

### Attributes
- __name__ 
    The name of the form. Used to reference the form along with the XML Screen file location. For HTML output this is the form name and id, and for other output may also be used to identify the part of the output corresponding to the form.
    Form name must be unique in the screen. System can't check this rule but the rendering will have problem if there's duplication in form name 
- __extends__ 
    The location and name separated by a hash/pound sign (#) of the form to extend. If there is no location it is treated as a form name in the current screen.
- __transition__
    The transition in the current screen to submit the form to.
- __map__
    The Map to get field values from. Is often a EntityValue object or a Map with data pulled from various places to populate in the form. Map keys are matched against field names. This is ignored if the field.entry-name attribute is used, that is evaluated against the context in place at the time each field is rendered. Defaults to "fieldValues".
- __focus-field__
    The name of the field to focus on when the form is rendered.
- __skip-start__ type="boolean" default="false"
    Skip the starting rendered elements of the form. When used after a form with skip-end=true this will effectively combine the forms into one
- __skip-end__ type="boolean" default="false"
    Skip the ending rendered elements of the form. Use this to leave a form open so that additional forms can be combined with it.
- __dynamic__ type="boolean" default="false"
    If true this form will be considered dynamic and the internal definition will be built up each time it is used instead of only when first referred to. This is necessary when auto-fields-* elements have ${} string expansion for service or entity names.
- __background-submit__ type="boolean" default="false"
    Submit the form in the background without reloading the screen.
- __background-reload-id__ 
    After the form is submitted in the background reload the dynamic-container with this id.
- __background-hide-id__
    After the form is submitted in the background hide the element (usually a dialog) with the specified id.
- __background-message__ 
    After the form is submitted in the background show this message in a dialog.

### Children elements

#### field-layout

## form-list
A list form is a list of individual forms in a table (could be called a tabular form), it has a list of sets of values and creates one form for each list element.
A variation on the list form is the multi form (set the attribute multi=true). In the multi mode all list elements will be put into a single large form with suffixes on each field for each row, with a single submit button at the bottom instead of a submit button on each row.

### Attributes
- __name__ 
- __extends__ 
- __transition__
all three attributes above are same as in `form-single`

- __row-actions__
- __multi__ default="false" type="boolean"
    Make the form a multi-submit form where all rows on a page are submitted together in a single request with a "_${rowNumber}" suffix on each field. Also passes a _isMulti=true parameter so the Service Facade knows to run the service (a single service-call in a transition) for each row. Defaults to true, so set to false to disable this behavior and have a separate form (submitted separately) for each row.

- __list__ 
    An expression that evaluates to a list to iterate over.

- __list-entry__ 
    If specified each list entry will be put in the context with this name, otherwise the list entry must be a Map and the entries in the Map will be put into the context for each row
    _Notes:_ 
    - if obmit, data binding in each field will automatically done using field name and the same key in context
    - if this attribute is specified, must define `entry-name` explicitly for each field
- __paginate__ type="xs:string" default="true"
    Indicate if this form should paginate or not. Defaults to true.
- __paginate-always-show__ type="xs:string" default="true"
    Always show the pagination control with count of rows, even when there is only one page? Defaults to true.
- __skip-start, skip-end__ same as in `form-single`
- __skip-form__ type="boolean" default="false"
    Make the output a plain table, not submittable (in HTML don't generate 'form' elements). Useful for view-only list forms to minimize output.
- __skip-header__ type="boolean" default="false"
    Skip the table header element.
- __header-dialog__ type="boolean" default="false"
    Put header-field widgets in a dialog instead of the table header. Includes all fields with header widgets, not just those displayed.
- __select-columns__ type="boolean" default="false"
    Enable per-user selection of which columns to display.
- __saved-finds__ type="boolean" default="false"
    Enable saved finds (query parameters, order by).
- __show-csv-button__ type="boolean" default="false"
    Show a button to export as CSV (renderMode=csv), if the pagination header is displayed
- __show-text-button__ type="boolean" default="false"
    Show a button to export as plain text (renderMode=text), if the pagination header is displayed
- __show-all-button__ type="boolean" default="false"
    Show a button to display all results (pageNoLimit=true), if the pagination header is displayed
- __dynamic__ type="boolean" default="false"
    If true this form will be considered dynamic and the internal definition will be built up each time it is used instead of only when first referred to. This is necessary when auto-fields-* elements have ${} string expansion for service or entity names.
- __background-submit__ type="boolean" default="false"
    Submit the form in the background without reloading the screen.
- __background-reload-id__ 
    After the form is submitted in the background reload the dynamic-container with this id.
- __background-hide-id__
    After the form is submitted in the background hide the element (usually a dialog) with the specified id.
- __background-message__ 
    After the form is submitted in the background show this message in a dialog.

### Children elements

#### form-list-column

