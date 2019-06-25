# middleware-multipart

A _multipart/form-data_ middleware based on [busboy](https://github.com/mscdex/busboy) library.

### Installation

```bash
$ npm i @marblejs/middleware-multipart
```

Requires `@marblejs/core` to be installed.

### Importing

```typescript
import { multipart$ } from '@marblejs/middleware-multipart';
```

### Type declaration <a id="type-declaration"></a>

```text
multipart$ :: ParserOpts -> HttpMiddlewareEffect
```

### Parameters

| parameter | definition |
| :--- | :--- |
| _options_ | &lt;optional&gt; `ParserOpts` |

_**ParserOpts**_

| _**parameter**_ | definition |
| :--- | :--- |
| _files_ | &lt;optional&gt; `Array<string>` |
| _stream_ | &lt;optional&gt; `StreamHandler` |
| _maxFileSize_ | &lt;optional&gt; `number` |
| _maxFileCount_ | &lt;optional&gt; `number` |
| _maxFieldSize_ | &lt;optional&gt; `number` |
| _maxFieldCount_ | &lt;optional&gt; `number` |

### Usage

{% hint style="warning" %}
Make sure that you always handle the files that a user uploads. Never add it as a global middleware since a malicious user could upload files to a route that you didn't handle. Only use this it on routes where you are handling the uploaded files.
{% endhint %}

**In-memory storage:**

```typescript
import { multipart$ } from '@marblejs/middleware-multipart';

const effect$ = r.pipe(
  r.matchPath('/'),
  r.matchType('POST'),
  r.useEffect(req$ => req$.pipe(
    use(multipart$()),
    map(req => ({ body: {
      files: req.files,    // file data
      body: req.body,      // all incoming body fields
    }}))
  )));
```

**Out-of-memory storage:**

```typescript
import { multipart$, StreamHandler } from '@marblejs/middleware-multipart';

const stream: StreamHandler = ({ file, fieldname }) => {
  // stream here incoming file to different location...
  const destination = // ... and grab the persisted file location
  return of({ destination });
};

const effect$ = r.pipe(
  r.matchPath('/'),
  r.matchType('POST'),
  r.useEffect(req$ => req$.pipe(
    use(multipart$({
      stream,
      maxFileCount: 1,
      files: ['image_1'],
    })),
    map(req => ({ body: {
      files: req.files['image_1'],  // file data
      body: req.body,               // all incoming body fields
    }}))
  )));
```

You can intercept incoming files and stream them to the different place, eg. to OS filesystem or AWS S3 bucket. The prervious example shows how you can specify constraints for _multipart/form-data_ parsing the accepts only **one** `image_1` field.

Each file included inside `req.files` object contains the following information:

| key | description |
| :--- | :--- |
| `fieldname` | Field name specified in the form |
| `filename` | Name of the file on the user's computer |
| `encoding` | Encoding type of the file |
| `mimetype` | Mime type of the file |
| `size` | Size of the file in bytes **\(in-memory parsing\)** |
| `destination` | The place in which the file has been saved **\(if not in-memory parsing\)** |
| `buffer` | A `Buffer` of the entire file  **\(in-memory parsing\)** |

You can define the following middleware options:

| key | description |
| :--- | :--- |
| `maxFileCount` | The total count of files that can be sent |
| `maxFieldCount` | The total count of fields that can be sent |
| `maxFileSize` | The max possible file size in bytes |
| `files` | An array of acceptable field names |
| `stream` | A handler which you can use to stream incoming files to different location |

