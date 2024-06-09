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

  <form class="modal-content animate" action="/signup" method="post" enctype="multipart/form-data">
    <div class="imgcontainer">
        <h2>Signup to the WebApp</h2>
      <span onclick="document.getElementById('id01').style.display='none'" class="close" title="Close Modal">&times;</span>
      <img src="/static/face.png" alt="Avatar" class="avatar">
    </div>

    <div class="container">
      <label for="uname"><b>Enter Username</b></label>
      <input type="text" placeholder="Enter Username" name="uname" required>

      <label for="psw"><b>Password</b></label>
      <input type="password" placeholder="Enter Password" name="psw" required>

      <label for="confirm_psw"><b>Confirm Password</b></label>
      <input type="password" placeholder="Confirm Password" name="confirm_psw" required>

      <label for="profile_picture"><b>Profile Picture</b></label>
      <input type="file" name="profile_picture" accept="image/*">

      <button type="submit">Sign Up</button>
      <label>
        <input type="checkbox" checked="checked" name="remember"> Remember me
      </label>
    </div>

    <div class="container" style="background-color:#f1f1f1">
      <button type="button" onclick="document.getElementById('id01').style.display='none'" class="cancelbtn">Cancel</button>
      <button type="button" onclick="window.location='/';" class="loginbtn">Login</button>
      <span class="psw">Forgot <a href="#">password?</a></span>
    </div>
  </form>
</div>

<script>
    // Function to display alert messages
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
