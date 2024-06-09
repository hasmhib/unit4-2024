```.py
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="stylesheet" href="/static/my_style.css">
<style>
body {font-family: Arial, Helvetica, sans-serif;}
</style>
</head>
<body>

<div>

  <form class="modal-content animate" action="/" method="post">
    <div class="imgcontainer">
        <h2>Login to the Movie Reddit</h2>
      <span onclick="document.getElementById('id01').style.display='none'" class="close" title="Close Modal">&times;</span>
      <img src="/static/face.png" alt="Avatar" class="avatar">
    </div>

    <div class="container">
      <label for="uname"><b>Enter Username</b></label>
      <input type="text" placeholder="Enter Username" name="uname" required>

      <label for="psw"><b>Password</b></label>
      <input type="password" placeholder="Enter Password" name="psw" required>

      <button type="submit">Login</button>
      <label>
        <input type="checkbox" checked="checked" name="remember"> Remember me
      </label>
    </div>

    <div class="container" style="background-color:#f1f1f1">
      <button type="button" onclick="document.getElementById('id01').style.display='none'" class="cancelbtn">Cancel</button>
      <button type="button" onclick="window.location='/signup';" class="signupbtn">Sign Up</button>
      <span class="psw">Forgot <a href="#">password?</a></span>
    </div>
  </form>
</div>

<script>
function displayMessage(message) {
    if (message) {
        alert(message);
    }
}

// Get the message from the query parameter
const urlParams = new URLSearchParams(window.location.search);
const message = urlParams.get('message');

// Display the message
displayMessage(message);
</script>

</body>
</html>

```
