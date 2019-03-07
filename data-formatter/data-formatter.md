We will need a "standardizer" library - responsible for taking data in a given format and repurposing it into our desired standards. This will likely be some representation of JSON-LD and/or IPLD. This is a constant, open conversation between people on the web and within protocol labs.

It seems that JSON-LD is compatible with IPLD, but IPLD is not completely backwards compatible with JSON-LD (taken from here)[https://github.com/ipfs/ipfs/issues/36#issuecomment-470617316]. I'm not sure what implications this has for us yet. WIP.

Relevant issues and repositories:

https://github.com/underlay/jsonld-dweb-loader
https://github.com/underlay/js-ipld-jsonld
https://github.com/ipfs/notes/issues/152
https://github.com/ipld/specs/issues/1
https://github.com/ipfs/notes/issues/155
https://github.com/ipfs/ipfs/issues/36#issuecomment-470617316
https://github.com/ipfs/ipfs/issues/36#issuecomment-135612692 - interesting note by jua nhere saying that they reserve @type and hash properties on JSON. But freedom to use @context
https://github.com/ipfs/ipfs/issues/36#issuecomment-135665957 - great link on RDF
https://github.com/jonnycrunch/ipid
