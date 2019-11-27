## Create thumbnails
1. Use [index5.html](https://raw.githubusercontent.com/ilkkamtk/wop-starters/week2-1/week2_public_html/index5.html), [main5.js](https://raw.githubusercontent.com/ilkkamtk/wop-starters/week2-1/week2_public_html/js/main5.js), [mapbox.js](https://raw.githubusercontent.com/ilkkamtk/wop-starters/week2-1/week2_public_html/js/mapbox.js) and [style5.css](https://raw.githubusercontent.com/ilkkamtk/wop-starters/week2-1/week2_public_html/css/style5.css) as front-end for testing
   * ask mapbox key from the teacher or create your own
1. Add new folder `thumbnails`
1. Install [sharp](https://github.com/lovell/sharp): `npm i sharp`
1. Add new file `utils/resize.js`:
   ```javascript
   'use strict';
   const sharp = require('sharp');
   
   const makeThumbnail = async (file, thumbname) => { // file = full path to image (req.file.path), thumbname = filename (req.file.filename)
     // TODO: use sharp to create a png thumbnail of 160x160px, use async await
   };
   
   module.exports = {
     makeThumbnail,
   };
   ```
1. Complete TODO in the code above. Call `makeThumbnail` function in `catController` before you add the cat to database.
1. Upload a new cat in index5.html

## Metadata from image
1. Study [Creating a Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise#Creating_a_Promise)
1. Take a couple of photos with your phone. Make sure that location services are enabled so that the gps data is stored in the images.
1. Email the images to yourself
1. Add new field `coords` of type text to `wop_cat` table
1. Add this as the value for `coords` to existing cats in the table: [24.74,60.24]
1. Modify `addCat` function in `models/catModel.js`:
   ```javascript
   const addCat = async (params) => {
     try {
       const [rows] = await promisePool.execute(
           'INSERT INTO wop_cat (name, age, weight, owner, filename, coords) VALUES (?, ?, ?, ?, ?, ?);',
           params);
       return rows;
     }
     catch (e) {
       console.log('error', e.message);
     }
   };
   ``` 
1. Modify `cat create post` in `catController.js`:
   ```javascript
   const cat_create_post = async (req, res) => {
     const errors = validationResult(req);
   
     if (!errors.isEmpty()) {
       res.send(errors.array());
     } else {
       try {
         // make thumbnail
         resize.makeThumbnail(req.file.path, req.file.filename);
         // get coordinates
         const coords = await imageMeta.getCoordinates(req.file.path);
         console.log('coords', coords);
         // add to db
         const params = [
           req.body.name,
           req.body.age,
           req.body.weight,
           req.body.owner,
           req.file.filename,
           coords,
         ];
         const cat = await catModel.addCat(params);
         await res.json({message: 'upload ok'});
       }
       catch (e) {
         console.log('exif error', e);
         res.status(400).json({message: 'error'});
       }
     }
   };
   ```
1. Install node-exif: `npm i exif`
1. Add new file `utils/imageMeta.js`:
   ```javascript
   'use strict';
   const ExifImage = require('exif').ExifImage;
   
   const getCoordinates = (imgFile) => { // imgFile = full path to uploaded image
     return new Promise((resolve, reject) => {
       try {
         // TODO: Use node-exif to get longitude and latitude from imgFile
         // coordinates below should be an array [longitude, latitude]
         resolve(coordinates);
       }
       catch (error) {
         reject(error);
       }
     });
   };
   
   // convert GPS coordinates to decimal format
   // for longitude, send exifData.gps.GPSLongitude, exifData.gps.GPSLongitudeRef
   // for latitude, send exifData.gps.GPSLatitude, exifData.gps.GPSLatitudeRef
   const gpsToDecimal = (gpsData, hem) => {
     let d = parseFloat(gpsData[0]) + parseFloat(gpsData[1] / 60) +
         parseFloat(gpsData[2] / 3600);
     return (hem === 'S' || hem === 'W') ? d *= -1 : d;
   };
   
   module.exports = {
     getCoordinates,
   };
   ```   
1. Require `utils/resize.js` as `resize` in `catController.js`
1. Complete TODO in `utils/resize.js`
1. Upload cat in index5.html to test

### Upload to virtual computer and run
1. Upload final app to your virtual computer
   * don't upload node_modules
   * after uploading, run `npm install`
   * edit .env if neccessary and run `node app.js` or `nodemon app.js`
   * modify app.js so that app runs via https
