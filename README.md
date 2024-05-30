# Unit 4: A mini Reddit

<img width="max" alt="Screenshot 2024-05-30 at 4 50 32 PM" src="https://github.com/hasmhib/unit4-2024/assets/142870448/60040ad3-732d-464f-a799-1287dd2e9a45">
DALL·E 2024-05-30 16.49.04 - A stylized, artistic depiction of a diverse group of people sitting in a dimly lit movie theater, watching a movie on a large screen.webp

# Criteria C: Development (no more than 1000 words)

## Technique Used
1. Python inside HTML
2. CSS Styling
3. For loops for showing posts
4. if statements
5. Password Hashing
6. Token-based authentication
7. Lists and Dictionaries
8. HTTP server
9. POST, GET Request
10. SQL Query
11. Sanitations of user input


## User profile picture
I decided to allow users to be able to add a profile picture. The code below shows my attempt and I will explain in detail below:

```.py
app.config['UPLOAD_FOLDER'] = 'static/profile_pics'
app.config['ALLOWED_EXTENSIONS'] = {'png', 'jpg', 'jpeg', 'gif'}

def save_profile_pic(file):
    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
        return filename
    return None

```

The function ```save_profile_pic``` is designed to handle the upload and saving of user profile pictures securely. This function starts by checking if the file exists and if its filename is valid using the allowed_file function. The ```allowed_file``` function verifies that the file has one of the permitted extensions like 'png', 'jpg', 'jpeg', or 'gif', which are defined in the application's configuration ```app.config['ALLOWED_EXTENSIONS']```. This validation is done by checking if there is a period in the filename and if the part after the last period matches one of the allowed extensions. The reason why I choose to use ```app.config``` is that it allows for centralized configuration management within the Flask application. By storing the allowed file extensions and the upload folder path in ```app.config```, it becomes easier to manage and modify these settings in one place throughout the application. If the file passes this validation, the filename is sanitized using ```secure_filename``` from the ```werkzeug.utils``` module. This step is crucial to prevent security issues that could arise from malicious filenames, such as those containing special characters. The sanitized filename is then used to save the file to the server in the directory specified by ```app.config['UPLOAD_FOLDER']```. The path to the file is constructed using the ```os.path.join``` function from the ```os``` module, ensuring compatibility across different operating systems. The ```save``` method of the file object is then called to save the file at this location. If the file is successfully saved, the function returns the filename. If the file is not valid or the saving process fails, the function returns None. This mechanism ensures that only valid image files are uploaded and saved securely, enhancing user experience by allowing them to personalize their profiles. 


## User profile information

These SQL Queries are used to fetch various information about a user from the database ```follows```, ```reviews```, and ```likes```, which will be displayed on the user's profile page. I will explain in detail. 

```.py
user = db.search(f"SELECT * FROM user WHERE id={user_id}", multiple=False)

following = db.search(f"SELECT * FROM follows WHERE follower_id={current_user_id} AND followed_id={user_id}", multiple=False) is not None
review_count = db.search(f"SELECT COUNT(*) FROM reviews WHERE user_id={user_id}", multiple=False)[0]

like_count = db.search(f"""SELECT COUNT(*) FROM likes l JOIN reviews r ON l.review_id = r.id WHERE r.user_id={user_id}""", multiple=False)[0]

follower_count = db.search(f"SELECT COUNT(*) FROM follows WHERE followed_id={user_id}", multiple=False)[0]
following_count = db.search(f"SELECT COUNT(*) FROM follows WHERE follower_id={user_id}", multiple=False)[0]
```

The first query retrieves all information about the user from the ```user``` table based on their ```user_id```, including details like the username, email, and profile picture. 
The second query checks if the current user (the one logged in) is following the user whose profile is being viewed. This is done by looking for a record in the ```follows``` table where the ```follower_id``` is the current user's ID and the ```followed_id``` is the profile user's ID. If such a record exists, it means the current user is following the profile user.
The third query counts the number of reviews posted by the user by selecting the count of rows in the ```reviews``` table where the ```user_id``` matches the profile user's ID. This is achieved using the ```COUNT(*)``` function, an built-in function in SQL that returns the number of rows that match the specified criteria. In this case, it returns the number of reviews that the user has posted. 
The fourth query counts the number of likes received on the user's reviews by joining the ```likes``` and ```reviews``` tables. It selects the count of rows in the likes table where the ```review_id``` matches a review ID from the ```reviews``` table, which in turn matches the profile user's ID. The ```JOIN``` method is used to combine rows from two or more tables based on a related column between them.
The fifth query counts how many users are following the profile user by selecting the count of rows in the ```follows``` table where the ```followed_id``` matches the profile user's ID. Similarly, the final query counts how many users the profile user is following by selecting the count of rows in the ```follows``` table where the ```follower_id``` matches the profile user's ID. 

These queries together provide a detailed profile page with all relevant user information, enhancing the user experience by showing their activities, interactions, and social metrics on the platform.


## SQL Query for Review likes


```.py
revs = db.search(query=f"""
    SELECT r.id, r.date, r.stars, r.comment, r.movie_id, COUNT(l.id) as likes, r.user_id, u.uname,
           (SELECT 1 FROM likes WHERE review_id = r.id AND user_id = {current_user_id}) as liked
    FROM reviews r
    JOIN user u ON r.user_id = u.id
    LEFT JOIN likes l ON r.id = l.review_id
    WHERE r.movie_id = {movie_id}
    GROUP BY r.id
""", multiple=True)
```

The SQL Query used to retrieve reviews for a specific movie, corresponding with the number of likes each review has received and whether the current user liked each review, is designed to display detailed information about both movie and book reviews and user interactions. The query selects columns from the ```reviews``` table, including review details such as the review ID, date, stars, comment, and movie ID, as well as user details like the user ID and username. The ```COUNT()``` function is used to count the number of likes each review has received. The ```JOIN``` clause combines rows from the ```reviews``` table with the ```user``` table to get the username of the reviewer and with the ```likes``` table to count the number of likes for each review.

The query includes a subquery to check if the current user has liked each review. This subquery returns ```1``` if the current user liked the review; otherwise, it returns ```NULL```. This check is performed by selecting from the ```likes``` table where the ```review_id``` matches the review ID and the ```user_id``` matches the current user's ID. The query filters reviews by the specified movie ID using the ```WHERE``` clause and groups the results by the review ID using the ```GROUP BY``` clause to ensure correct aggregation of likes. This comprehensive view of movie reviews, including user engagement metrics, enhances the user experience by showing interactive and relevant content.


## Dynamic Rendering with Python inside HTML

Here is the example of the Dynamic Rendering with python inside HTML by using ```render_template()``` method. This enables me to conbine python file and HTML file, which is more efficient. I will explain this code in detail. 

```.py
return render_template('movie_reddit.html', username=username, results=movies, user_id=user_id)
```

This line of code return ```render_template('movie_reddit.html', username=username, results=movies, user_id=user_id)``` uses the ```render_templat```e function from Flask to generate and return an HTML page. The ```render_template``` function takes the template file name, in this case ```'movie_reddit.html'```, and passes the provided variables to it: ```username``` (the currently logged-in user's name), ```results``` (a list of movies from the database), and ```user_id``` (the currently logged-in user's ID). This allows the template to dynamically display these variables stored in python file, such as the user's name, the list of movies, and user-specific actions. Using ```render_template``` helps separate the presentation logic (HTML) from the application logic (Python), making the code more maintainable and is benefitable for future develops. 

Here is my first attempt of an use of Dynamic Rendering inside ```movie_reddit.html``` using the code above：
To display the value of the ```r``` variable in the HTML signup file, I use template tags provided by the Jinja2 templating engine used by Flask. The most common template tag is {{ }}, which allows me to output the value of a variable. Here's an example of how I use template tags in my HTML files: 

```.py
{% for r in results %}
<tr>
    <td>{{ r[0] }}</td>
    <td>{{ r[1] }}</td>
    <td>{{ r[2] }}</td>
    <td>{{ r[3] }}</td>
    <td>{{ r[4] }}</td>
    <td><a class="btn warning" href="{{ url_for('see_review', movie_id=r[0]) }}">See</a></td>
</tr>
{% endfor %}
```

This code iterates over the ```results``` list, creating a table row ```(<tr>)``` for each item. Each ```<td>``` tag displays an element from the current item ```r``` in the ```results``` list, where ```r``` is expected to be a list or tuple.

The ```{{ r[0] }}```, ```{{ r[1] }}```, ```{{ r[2] }}```, ```{{ r[3] }}```, and ```{{ r[4] }}``` are placeholders that will be replaced by the elements of r which is stored in python file.

The last ```<td>``` contains a link styled as a button, which navigates to a page generated by the ```see_review``` route, passing the ```movie_id``` parameter as ```r[0]```. The ```{% endfor %}``` line ends the loop. This code dynamically generates table rows for each movie in the ```results``` list, displaying movie details and providing a link to view each movie's review.

However, I realized that the code is repeated and not efficient, so I decided to use **nested loop** to iterate through the items and create the table rows dynamically. Here is the modified version of the code:

```.py
{% for row in results %}
<tr>
    {% for col in row %}
    <td>{{ col }}</td>
    {% endfor %}
    <td><a class="btn warning" href="{{ url_for('see_review', movie_id=row[0]) }}">See</a></td>
</tr>
{% endfor %}
```
This code uses **nested loop**, will iterate through each row in ```results```, and then for each row, it will iterate through each ```col``` (column) to create the table cells. The last column, which contains the "See" button, is added separately outside the inner loop.


# Criteria D: Functionlaity

## Demonstration video
how easy someone else can expand further

# Criteria E: Evaluation

## Functionality test (2 users)


## Evaluation by Peers
My peer is very satisfied with the website, especially... as detailed in Figure 3 and Figure 4. However, they suggested an area for improvement: adding functionality on the 'Profile' page that allows users to see lists of their followers and whom they are following. Implementing this feature could allow user engagement by making the social connections within the platform more visible and accessible.

## Evaluation of beta testing
I received a feedback from the user and they are very satisfied with the website, as detailed in Figure 5. However, they suggested an area for improvement: add a confirmation dialog for when deleting posts as the user had accidentally deleted a few posts while testing. This will enables users to prevent data loss. 


## Extensibility
After some testing and feedbacks from my peers, I concluded that the following future extensions could be added:

1. Adding functionality on the 'Profile' page - Adding functionality on the 'Profile' page that allows users to see lists of their followers and whom they are following. Implementing this feature could allow user engagement by making the social connections within the platform more visible and accessible.

2. Sorting option "By most likes" - An option can be added to allow for filtering by most likes so the user can find most popular comments more easily.


# Appendix

<img width="max" alt="Screenshot 2024-05-28 at 1 55 53 PM" src="https://github.com/hasmhib/unit4-2024/assets/142870448/be290c05-58db-47e2-b748-f28108ca5ae1">

##### _Figure.3 Contact between developer and client for evaluation of website_

<img width="max" alt="Screenshot 2024-05-28 at 1 56 27 PM" src="https://github.com/hasmhib/unit4-2024/assets/142870448/5c23a7f1-cdcb-4d60-a2f6-1fd4e1a00813">

##### _Figure.4 Contact between developer and client for evaluation of website_

<img width="max" alt="Screenshot 2024-05-28 at 1 56 13 PM" src="https://github.com/hasmhib/unit4-2024/assets/142870448/d5995505-fa6a-4ff4-8bdd-d5c01fabcaf3">

##### _Figure.5 Contact between developer and client regarding beta testing and feedback_



[^1]: https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-iv-database ---. “The Flask Mega-Tutorial, Part IV: Database.” Copyright (C) 2012-2024 Miguel Grinberg, blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-iv-database.
[^2]: https://www.w3schools.com/sql/ SQL Tutorial. www.w3schools.com/sql.
[^3]: https://jinja.palletsprojects.com/en/3.1.x Jinja — Jinja Documentation (3.1.x). jinja.palletsprojects.com/en/3.1.x.
[^4]: https://pypi.org/project/Werkzeug “Werkzeug.” PyPI, 5 May 2024, pypi.org/project/Werkzeug.
[^5]: https://docs.python.org/3/library/os.html “Os — Miscellaneous Operating System Interfaces.” Python Documentation, docs.python.org/3/library/os.html.

