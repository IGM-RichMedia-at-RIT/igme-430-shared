# File upload: 12B code

Code for Wednesday's demo. The numbers match the steps in class.

Starter: [Express-File-Upload-Start](https://github.com/IGM-RichMedia-at-RIT/Express-File-Upload-Start)

## 1 · terminal

```
git clone https://github.com/IGM-RichMedia-at-RIT/Express-File-Upload-Start
cd Express-File-Upload-Start
npm install
```

## 6 · second terminal

```
npm install express-fileupload
```

## 7a · `src/app.js` (under the `path` line)

```js
const fileUpload = require('express-fileupload');
```

## 7b · `src/app.js` (under `app.use(compression());`)

```js
app.use(fileUpload());
```

## 8 · `src/controllers/files.js` (replaces the empty `uploadFile`)

```js
const uploadFile = (req, res) => {
  if (!req.files || !req.files.sampleFile) {
    return res.status(400).json({ error: 'No files were uploaded' });
  }
};
```

## 9 · `src/controllers/files.js`

```js
  console.log(req.files);
  const { sampleFile } = req.files;
```

## 10 · `src/models/filestore.js`

```js
const mongoose = require('mongoose');

const FileSchema = new mongoose.Schema({
});

const FileModel = mongoose.model('FileModel', FileSchema);

module.exports = FileModel;
```

## 11 · `src/models/filestore.js`

```js
  name: { type: String },
  data: { type: Buffer },
  size: { type: Number },
  mimetype: { type: String },
```

## 12a · `src/controllers/files.js`

```js
const uploadFile = async (req, res) => {
```

## 12b · `src/controllers/files.js`

```js
  try {
    const newFile = new File(sampleFile);
    const doc = await newFile.save();
    return res.status(201).json({ message: 'File stored successfully!', fileId: doc._id });
  } catch (err) {
    console.log(err);
    return res.status(400).json({ error: 'Something went wrong uploading file!' });
  }
```

## 13 · `src/controllers/files.js` (replaces the empty `retrieveFile`)

```js
const retrieveFile = async (req, res) => {
  if (!req.query._id) {
    return res.status(400).json({ error: 'Missing file id!' });
  }
};
```

## 14 · `src/controllers/files.js`

```js
  let doc;
  try {
    doc = await File.findOne({ _id: req.query._id }).exec();
  } catch (err) {
    console.log(err);
    return res.status(400).json({ error: 'Something went wrong retrieving file!' });
  }
```

## 15 · `src/controllers/files.js`

```js
  if (!doc) {
    return res.status(404).json({ error: 'File not found!' });
  }
```

## 16 · `src/controllers/files.js`

```js
  res.set({
    'Content-Type': doc.mimetype,
    'Content-Length': doc.size,
    'Content-Disposition': `inline; filename="${doc.name}"`,
  });
  return res.send(doc.data);
```

## 17 · `src/controllers/files.js`

```js
    'Content-Disposition': `attachment; filename="${doc.name}"`,
```

## 18 · `hosted/client.js`

```js
const uploadFile = async (e) => {
  e.preventDefault();
  const response = await fetch('/upload', {
    method: 'POST',
    body: new FormData(e.target),
  });
};
```

## 19 · `hosted/client.js`

```js
  const text = await response.text();
  document.getElementById('messages').innerText = text;
```

## 20 · `hosted/client.js`

```js
const init = () => {
  const uploadForm = document.getElementById('uploadForm');
  uploadForm.addEventListener('submit', uploadFile);
};

window.onload = init;
```

## 21 · `views/upload.handlebars`

```html
    <img src="/retrieve?_id=PASTE_AN_ID_HERE" style="max-width: 400px" />
```

## 23 · `src/controllers/files.js` (for your project: images only)

```js
  if (!sampleFile.mimetype.startsWith('image/')) {
    return res.status(400).json({ error: 'Images only, please' });
  }
```

## 24 · React (for your project: a sketch, not working code)

```jsx
const response = await fetch('/upload', { method: 'POST', body: new FormData(e.target) });
const { fileId } = await response.json();

// save fileId on your own record, then:
<img src={`/retrieve?_id=${post.imageId}`} alt={post.title} />
```

Finished version: [Express-File-Upload-Done](https://github.com/IGM-RichMedia-at-RIT/Express-File-Upload-Done)
