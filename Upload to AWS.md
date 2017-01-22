
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

> Written with [StackEdit](https://stackedit.io/).