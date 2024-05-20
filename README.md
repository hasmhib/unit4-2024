# Unit 4: A mini Reddit


# Criteria C: Development (no more than 1500 words)

## Technique Used
1. Flask Library/Routes
2. Python inside HTML
3. CSS Styling
4. For loops for showing posts
5. if statements
6. Password Hashing
7. Token-based authentication
8. Interacting with Databases
9. Arrays and Lists
10. Text Formatting

## Sources

### Login System

My client required a login system for the application so that different users could have their unique profile pages and post comments. I initially decided to use cookies as a way of storing when a user is logged in. The code below shows my first attempt and I will explain it in detail below:

```.py
def login():
    if request.method == "POST":
        uname = request.form.get('uname')
        psw = request.form.get('psw')
        db = DatabaseBridge('user.db')
        r = db.search(f"""SELECT * from user where uname == '{uname}' and psw == '{psw}'""")
        if r[1] == {uname} and r[2] == {psw}:
            response = make_response(redirect(url_for('profile')))
            response.set_cookie('user_id', r[0])
            return response
```
This code is run when the request from the client received by the server is of type POST ```request.method == "POST"```. This happens when the user clicks on the login button on the index.html page. Then, I proceed to get the variables from the login form including the name and password, this is contained in the dictionary “request”. After checking the database with the SQL query ##(“””(“””))... Then I set the cookie for the user with the code ``` response.set_cookie('user_id', r[0]) ```, and noted that the cookie is like a dictionary with keys and values in both strings.

However, based on my research about cookies and testing in the browser, I found out that the cookie is not a secure way of storing the user information since this is a variable stored in the browser on the client side, which can be easily changed causing identity thief if not hashed. So my solution to this issue was to change from client to server side by using sessions. The code below shows that I could store the current user in the dictionary session, "The data is required to be saved in the Session is stored in a temporary directory on the server". "The data in the Session is stored on the top of the cooclies and signed by the server cryptography". [^1]



# Criteria D: Functionlaity

## Demonstration video


# Criteria E: Evaluation

## FUnctionality test


[^1]: https://www.geeksforgeeks.org/how-to-use-flask-session-in-python-flask/

