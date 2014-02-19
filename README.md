#ESNA COLLABORATION TAGS

###How It Works

Collaboration tags are provided by iLink extensions. They are rendered with help of extension’s content script that is loaded to each browser page (including file urls if enabled). Each page where collaboration tags should be rendered should contain a basic declarative markup to help content script in decision of what should be detected as an address and might also have a set of CSS styles to adopt rendered content to host page look and feel.
Whenever content script finds a tag representing a person or address inside of host page, it will make a request to extension to get person status and set of available actions. Once that information is obtained HTML element containing person information is wrapped into special construct containing:
 - Spot. Could be used for default click action or for show card on hover. Also could be used to show person presence information or other type of image
 - Card. Optional, if card is enabled in page manifest, then it will contain a set of person labels, person image and list of actions. Depending on page manifest the card could be shown as a reaction on Spot click or as a reaction on spot hover
 - List of actions
 
Once person element is recognized and bound to extension, it will track person presence and possible updates to available actions.
Logging out of extension or disabling extension will not lead to removal of HTML wrappers, but default CSS rules will make additional elements invisible on the page. Visibility of collaboration tags will be restored automatically upon successful login in extension.


###Page manifest

In order to minimize changes necessary to be done to existing pages to enable collaboration tags, each page have to declare a set of collaboration tag options. Each option is represented by META tag in page HEAD. 

```html
	<meta name="x-ilink-lookups" content="1"/>
```
Enables collaboration tags on the page. If this tag is missing, then collaboration tags will not be presented on the page at all
```html
	<meta name="x-ilink-classes" content="person"/>
```
Space separated list of CSS classes of elements which should be treated as person information. It is preferred that elements containing person information are inline or inline-block. Example:
```html  
	<span class="person" person-id="test@test.com">First Last</span>
```
Note. HTML A (anchor) elements are always included as a candidate for possible person information. Example:
```html  
	<a href="mailto:a@b.c">First Last</a>
```
```html
	<meta name="x-ilink-attrs" content="person-id ."/>
```
Space separated list of attributes containing person address in page markup. Used in conjunction with x-ilink-classes in order to identify person address. Example:
```html
  <span class="person" person-id="test@test.com">First Last</span>
```
x-ilink-classes="person" and x-ilink-attrs="person-id", the element above will be found and detected as person info. Person address will be taken from person-id attribute.
A special value of . (dot) in list of attributes is used to define person address lookup inside of person element inner text. Example:
```html  
	<span class="person">test@test.com</span>
```
Note. For HTML A (anchor) elements lookup of address is always done in content of href attribute. Only mailto: addresses are detected as person information and corresponding email address is used as person address.
```html
	<meta name="x-ilink-mode" content="hoverCard" />
```	
Collaboration tags mode. Space separated list of modes:
- spotAction - Click on the spot will trigger default person action(depends on extension)
- spotCard - Click on the spot will trigger display of person card on the page
- hoverCard - Hover on original element will trigger display of person card on the page
- static - all original person elements will be replaced by person cards on the page


###Wrapper layout

Once element is identified as person element, content script will wrap it as shown below.

Original HTML:
```html
	<span class="person" person-id="test@test.com">First Last</span>
```
Wrapped HTML:
```html
	<span class="jsc-wrap" jsc-status="offline">
	  <span class="jsc-spot"></span>
	  <span class="jsc-card">
	    <img src="...">
	    <div>... labels ...</div>
	    <hr>
	    <a href="ws://" jsc-action="open" title="Open contact">Open</a>
	    <a href="ws://" jsc-action="mail" title="Send email">Mail</a>
	  </span>
	  <span class="person jsc-data" person-id="test2@test.com">First Last</span>
	</span>
```
Extension has exclusive control on content of the wrapper - it could change image url, set of labels, set of available actions etc. In order to provide look and feel and some visual effects host page might include additional CSS rules.
Examples:
- Show presence image, use:
```css
  .jsc-wrap[jsc-status='offline'] .jsc-spot
  {
    background-image: url('offline.png');
  }
  .jsc-wrap[jsc-status='dnd'] .jsc-spot
  {
    background-image: url('dnd.png');
  }
  .jsc-wrap[jsc-status='away'] .jsc-spot
  {
    background-image: url('away.png');
  }
  .jsc-wrap[jsc-status='online'] .jsc-spot
  {
    background-image: url('online.png');
  }
```	  
- Modify card background, use:
```css
  .jsc-wrap > .jsc-card
  {
    background-color: green;
  }
```	  
- Modify fonts, use
```css
  .jsc-wrap > .jsc-card
  {
    font-family: arial;
  }
```	  
- Hide person image, use:
```css
  .jsc-wrap > .jsc-card > img
  {
    display: none;
  }
```
- Set action icons and remove action labels, use:
```css
  .jsc-card > [jsc-action]
  {
    color: transparent;
    background-size: 1em 1em;
    background-position: center center;
    background-repeat: no-repeat;
    border: none;
    width: 1em;
    min-width: 1em;
  }
  .jsc-card > [jsc-action]:hover
  {
    background-color: orange;
  }
  [jsc-action='open']
  {
    background-image: url('open.png');
  }
  [jsc-action='call']
  {
    background-image: url('call.png');
  }
  [jsc-action='chat']
  {
    background-image: url('chat.png');
  }
  [jsc-action='mail']
  {
    background-image: url('mail.png');
  }
```

Notes:

1. All images used in spot or card should be provided by host page in CSS if necessary. Collaboration tags will provide only person image when available.
2. While you cannot exclude certain info like label or certain action from rendering in manifest, you could use CSS to hide a certain area of the card.
