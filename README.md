for email subscription : google sheet and appscript is used 
paste this in code.gs 
var sheetName = 'Sheet1'
var scriptProp = PropertiesService.getScriptProperties()

function intialSetup () {
  var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet()
  scriptProp.setProperty('key', activeSpreadsheet.getId())
}

function doPost (e) {
  var lock = LockService.getScriptLock()
  lock.tryLock(10000)

  try {
    var doc = SpreadsheetApp.openById(scriptProp.getProperty('key'))
    var sheet = doc.getSheetByName(sheetName)

    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0]
    var nextRow = sheet.getLastRow() + 1

    var newRow = headers.map(function(header) {
      return header === 'timestamp' ? new Date() : e.parameter[header]
    })

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow])

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
      .setMimeType(ContentService.MimeType.JSON)
  }

  catch (e) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': e }))
      .setMimeType(ContentService.MimeType.JSON)
  }

  finally {
    lock.releaseLock()
  }
}

paste this in html page
<script>
            const scriptURL = 'https://script.google.com/macros/s/AKfycbyaC6BZ-8eIy7i0wG3iSC126Fvbm7Cl-LPW4Gc1vB_cL7xyA4VrmwxRw_l6dFihq_1w/exec' (change this with the script from appscript
            const form = document.forms['submit-to-google-sheet']
            
          
            form.addEventListener('submit', e => {
              e.preventDefault()
              fetch(scriptURL, { method: 'POST', body: new FormData(form)})
                .then(response =>{
                    
                    form.reset()

                 } )
                .catch(error => console.error('Error!', error.message))
            })
          </script>
          
for contribution page:

code.gs
function getData() {
  let ws = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Sheet1');
  let date1 = ws.getRange('H1').getValue();
  let date2 = ws.getRange('J1').getValue();
  let startDate = new Date(date1);
  let endDate = new Date(date2);
  let data = ws.getRange('A2:F1000').getValues();
  let output = data.filter(function (getData) {
    return getData[0] > startDate && getData[0] < endDate
  });
  Logger.log(output)
  Logger.log(startDate);
  Logger.log(endDate)
 
}
 
 
function doGet(){
  return HtmlService.createTemplateFromFile('contribute').evaluate()
}
 
 
function include(filename){
  return HtmlService.createHtmlOutputFromFile(filename).getContent();
}
 
 
function submitFormData(formObject){
  let folder=DriveApp.getFolderById('1x3UnNVnvIUnNHh_1TUyf4_0NwSrkvi4C');
  let mainsheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Sheet1')
 if(formObject.filename.length>0){
    let blob=formObject.filename;
    let file= folder.createFile(blob);
    let url = file.getUrl();
    let fileName = file.getName();
    mainsheet.appendRow([new Date(),formObject.firstname,formObject.lastname,formObject.email,formObject.city,formObject.title,formObject.description,url,fileName])
  }
}

contribute.html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Bootstrap demo</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.0/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-gH2yIJqKdNHPEq0n4Mqa/HGKIhSkIHeL5AyhkYV8i59U5AR6csBvApHHNl/vI1Bx" crossorigin="anonymous">
    
    <?!=include("Javascript");?>
  </head>
  <body style="background-color:black">
    <div id="showTime" style="width: 120px;height: 50px;font-size: 20px; color: black;color: white"></div>
<div class="main" style="width:50%; margin:auto;padding:40px;">
    <form class="row g-3" id="formdata"  onsubmit="getFormData(this)">
  <div class="col-md-6">
    <label for="firstname" class="form-label" style="color:white">First Name</label>
    <input type="text" class="form-control" name="firstname" id="firstname">
  </div>
  <div class="col-md-6">
    <label for="lastname" class="form-label" style="color:white">Last Name</label>
    <input type="text" class="form-control" id="lastname" name="lastname">
  </div>
  <div class="col-6">
    <label for="email" class="form-label" style="color:white">Email</label>
    <input type="text" class="form-control" id="email" placeholder="xyz@gmail.com" name="email">
  </div>
  
  <div class="col-md-6">
    <label for="inputCity" class="form-label" style="color:white">City</label>
    <input type="text" class="form-control" id="inputCity" name="city">
  </div>
  <div class="col-md-6">
    <label for="title" class="form-label" style="color:white">Title</label>
    <input type="text" class="form-control" id="title" name="title">
  </div>
  <div class="col-md-6">
    <label for="description" class="form-label" style="color:white">Description</label>
    <input type="text" class="form-control" id="description" name="description">
  </div>
  <div>
  <label for="formFileLg" class="form-label" style="color:white">Upload Files </label>
  <input class="form-control form-control-lg" id="formFileLg" type="file" name="filename">
</div>
  
  <div class="col-12">
    <button type="submit" class="btn btn-primary">Submit</button>
  </div>
</form>
 
<div>
  <br>
  <div id="showAlert"></div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.0/dist/js/bootstrap.bundle.min.js" integrity="sha384-A3rJD856KowSb7dwlZdYEkO39Gagi7vIsF0jrRAoQmDKKtQBHUuLZ9AsSv4jD4Xa" crossorigin="anonymous"></script>
  </body>
</html>

javascript.html
<script>
   
   setInterval((showClock) => {
     let date = new Date();
    let time = date.toLocaleTimeString()
    document.getElementById("showTime").innerHTML=time
   }, 1000);
 
 window.addEventListener('load',preventSubmit);
 
 function preventSubmit(){
   let myForm =document.querySelectorAll('form');
   console.log(myForm)
   for(var i=0;i<myForm.length;i++){
     myForm[i].addEventListener('submit',function(e){
     e.preventDefault();
     })
   }
 }
 
function getFormData(formObject){
  google.script.run.withSuccessHandler(showSucess).submitFormData(formObject)
}
 
 function showSucess(){
   document.getElementById('showAlert').innerHTML='<div class="alert alert-success" role="alert">Data Successfully Submitted |</div>'
   document.getElementById("formdata").reset();
  setTimeout(clearAlert,4000)
 }
 
 function clearAlert(){
   document.getElementById('showAlert').innerHTML=""
 }
</script>
