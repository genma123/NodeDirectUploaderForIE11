
```html
    <h2>Your avatar</h2>

    <input type="file" id="file-input">
    <p id="status">Please select a file</p>
    <img style="border:1px solid gray;width:300px;"  id="preview" src="/images/default.png">

```
```javascript
    /*
      Function to carry out the actual PUT request to S3 using the signed request from the app.
    */
    function uploadFile(file, signedRequest, url){
	// console.log("uploading file to " + url);
	// console.log(JSON.stringify(signedRequest, null, 2));
      var xhr = new XMLHttpRequest();
      xhr.open('PUT', signedRequest);
 	  // xhr.setRequestHeader('content-type', file.type);
 	  xhr.onreadystatechange = function() {
        if(xhr.readyState === 4){
          if(xhr.status === 200){
            document.getElementById('preview').src = url;
            document.getElementById('avatar-url').value = url;
          }
          else{
            alert('Could not upload file.');
          }
        }
      };
	  console.log("about to send file");
      xhr.send(file);
	  console.log("file was sent");
    }

    /*
      Function to get the temporary signed request from the app.
      If request successful, continue to upload the file using this signed
      request.
    */
    function getSignedRequest(file){
      var xhr = new XMLHttpRequest();
	  var signedRequestURL = "/sign-s3?file-name=" + file.name + "&file-type=" + file.type;
      xhr.open('GET', signedRequestURL);
      xhr.onreadystatechange = function() {
        if(xhr.readyState === 4){
          if(xhr.status === 200){
            var response = JSON.parse(xhr.responseText);
            uploadFile(file, response.signedRequest, response.url);
          }
          else{
            alert('Could not get signed URL.');
          }
        }
      };
      xhr.send();
    }

    /*
     Function called when file input updated. If there is a file selected, then
     start upload procedure by asking for a signed request from the app.
    */
    function initUpload(){
      var files = document.getElementById('file-input').files;
      var file = files[0];
      if(file == null){
        return alert('No file selected.');
      }
      getSignedRequest(file);
    }

    /*
     Bind listeners when the page loads.
    */
    (function() {
        document.getElementById('file-input').onchange = initUpload;
    })();
```

```javascript
/*
 Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

 http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
*/


/*
 * Import required packages.
 * Packages should be installed with "npm install".
 */
const express = require('express');
const aws = require('aws-sdk');

/*
 * Set-up and run the Express app.
 */
const app = express();
app.set('views', './views');
app.use(express.static('./public'));
app.engine('html', require('ejs').renderFile);
app.listen(process.env.PORT || 3000);

/*
 * Load the S3 information from the environment variables.
 */
const S3_BUCKET = process.env.S3_BUCKET;

/*
 * Respond to GET requests to /account.
 * Upon request, render the 'account.html' web page in views/ directory.
 */
app.get('/account', (req, res) => res.render('account.html'));

/*
 * Respond to GET requests to /sign-s3.
 * Upon request, return JSON containing the temporarily-signed S3 request and
 * the anticipated URL of the image.
 */
app.get('/sign-s3', (req, res) => {
  const s3 = new aws.S3();
  const fileName = req.query['file-name'];
  const fileType = req.query['file-type'];
  const s3Params = /* {
    Bucket: S3_BUCKET,
    Key: fileName,
    Expires: 60,
    ContentType: fileType,
    ACL: 'public-read'
  } */
  {
    Bucket: S3_BUCKET,
    Key: fileName,
    Expires: 60,
    ACL: 'public-read'
  }
  ;

  s3.getSignedUrl('putObject', s3Params, (err, data) => {
    if(err){
      console.log(err);
      return res.end();
    }
    const returnData = {
      signedRequest: data,
      url: `https://${S3_BUCKET}.s3.amazonaws.com/${fileName}`
    };
    res.write(JSON.stringify(returnData));
    res.end();
  });
});

/*
 * Respond to POST requests to /submit_form.
 * This function needs to be completed to handle the information in
 * a way that suits your application.
 */
app.post('/save-details', (req, res) => {
  // TODO: Read POSTed form data and do something useful
  console.log("in save-details stub");
});
```

> Written with [StackEdit](https://stackedit.io/).