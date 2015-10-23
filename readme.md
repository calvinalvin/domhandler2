#DOMHandler

The DOM handler (formally known as DefaultHandler) creates a tree containing all nodes of a page. The tree may be manipulated using the DOMUtils library.

###Why domhandler2?
I needed an easy way to transform and modify the attributes of html tags. See `attribTransforms` option below. If you
want the original, it's here [original domhandler](https://github.com/fb55/domhandler)

##Usage
```javascript
var handler = new DomHandler([ <func> callback(err, dom), ] [ <obj> options ]);
// var parser = new Parser(handler[, options]);
```

##Example
```javascript
var htmlparser = require("htmlparser2");
var rawHtml = "Xyz <script language= javascript>var foo = '<<bar>>';< /  script><!--<!-- Waah! -- -->";
var handler = new htmlparser.DomHandler(function (error, dom) {
    if (error)
    	[...do something for errors...]
    else
    	[...parsing done, do something...]
        console.log(dom);
});
var parser = new htmlparser.Parser(handler);
parser.write(rawHtml);
parser.done();
```

Output:

```javascript
[{
    data: 'Xyz ',
    type: 'text'
}, {
    type: 'script',
    name: 'script',
    attribs: {
    	language: 'javascript'
    },
    children: [{
    	data: 'var foo = \'<bar>\';<',
    	type: 'text'
    }]
}, {
    data: '<!-- Waah! -- ',
    type: 'comment'
}]
```

##Option: normalizeWhitespace
Indicates whether the whitespace in text nodes should be normalized (= all whitespace should be replaced with single spaces). The default value is "false".

The following HTML will be used:

```html
<font>
	<br>this is the text
<font>
```

###Example: true

```javascript
[{
    type: 'tag',
    name: 'font',
    children: [{
    	data: ' ',
    	type: 'text'
    }, {
    	type: 'tag',
    	name: 'br'
    }, {
    	data: 'this is the text ',
    	type: 'text'
    }, {
    	type: 'tag',
    	name: 'font'
    }]
}]
```

###Example: false

```javascript
[{
    type: 'tag',
    name: 'font',
    children: [{
    	data: '\n\t',
    	type: 'text'
    }, {
    	type: 'tag',
    	name: 'br'
    }, {
    	data: 'this is the text\n',
    	type: 'text'
    }, {
    	type: 'tag',
    	name: 'font'
    }]
}]
```

##Option: withStartIndices
Indicates whether a `startIndex` property will be added to nodes. When the parser is used in a non-streaming fashion, `startIndex` is an integer indicating the position of the start of the node in the document. The default value is "false".

##Option: attribTransforms
An object that allows you to transform and modify attributes for tags. The property keys for this object should map to valid html tags. Each property should map to a function that receives an `attribs` object. You can do whatever you
want to the attribs, but once you're done, make sure you return them or they'll be null! The example below
transforms relative urls paths of `img` tags into absolute paths.

```javascript
var htmlparser = require('htmlparser2');
var DomHandler = require('domhandler2');
var tohtml = require('htmlparser-to-html');
var url = require('url');

var handler = function(domErr, dom) {
  if (domErr) {
    throw domErr;
  } else {
    html = tohtml(dom);
  }
};

var attribTransforms = {
  'img': function(attribs) {
    if (attribs.src) {
      // this allows you to transform relative paths into absolute paths
      attribs.src = url.resolve('http://mydomain.com/', attribs.src);
    }

    // remember to return the attribs or they will be null!
    return attribs;
  }
};

var domHandler = new DomHandler(handler, {attribTransforms: attribTransforms});

var parser = new htmlparser.Parser(domHandler);
parser.write(html);
parser.done();
return html;
}
```
