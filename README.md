# Unit 4: A mini Reddit

<img width="max" alt="Screenshot 2024-05-30 at 4 50 32 PM" src="https://github.com/hasmhib/unit4-2024/assets/142870448/60040ad3-732d-464f-a799-1287dd2e9a45">

##### _Figure.1 DALL·E 2024-05-30 16.49.04 - A stylized, artistic depiction of a diverse group of people sitting in a dimly lit movie theater, watching a movie on a large screen.webp_


# Criteria C: Development (Word count 982/1000 words)

## Technique Used

1. Python inside HTML
2. CSS Styling
3. For loops for showing posts
4. If statements
5. Password Hashing
6. Token-based authentication
7. Lists and Dictionaries
8. HTTP server
9. POST, GET Requests
10. SQL Queries
11. Sanitization of user input


## User profile picture
I decided to allow users to add a profile picture. The code below shows my attempt, and I will explain in detail:

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

The function ```save_profile_pic``` is designed to securely handle the upload and saving of user profile pictures. It starts by checking **if** the file exists and if its filename is valid using the ```allowed_file``` function, which verifies that the file has one of the permitted extensions like 'png', 'jpg', 'jpeg', or 'gif'. These extensions are defined in the application's configuration ```app.config['ALLOWED_EXTENSIONS']```. The reason I chose to use ```app.config``` is that it allows for configuration management within the Flask application, making it easier to manage and modify settings in one place. If the file passes this validation, the filename is sanitized using ```secure_filename``` from the ```werkzeug.utils``` module to prevent security problems from malicious filenames. The sanitized filename is then used to save the file to the server in the directory specified by ```app.config['UPLOAD_FOLDER']```. The path to the file is created using the ```os.path.join``` function from the ```os``` module, ensuring compatibility across different operating systems. The ```save``` method of the file object is then called to save the file. If the file is successfully saved, the function returns the filename, otherwise it returns ```None```. This ensures that only valid image files are uploaded and saved securely, enhancing the user experience by allowing them to personalize their profiles.


## User profile information retrieval

The **SQL Queries** are used to fetch various information about a user from the database, which will be displayed on the user's profile page. I will explain an example of the code in detail:

```.py
like_count = db.search(f"""SELECT COUNT(*) FROM likes l JOIN reviews r ON l.review_id = r.id WHERE r.user_id={user_id}""", multiple=False)[0]
```

This code counts the number of likes received by a user's reviews. The ```db.search``` method executes a **SQL Query** that uses the ```COUNT(*)``` method to sum the likes. The ```JOIN``` clause combines the ```likes``` and ```reviews ``` tables based on the ```review_id``` column, ensuring each like matches its corresponding review. The ```WHERE``` clause filters the reviews to include only the specified user ```(user_id)```. The ```multiple=False``` parameter means a single result is expected, and ```[0]``` extracts the count from the result. This count is stored in the ```like_count``` variable. This enhances the user experience by showing numbers that reflect user interactions. 


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

The **SQL Query** gains reviews for a specific movie, showing the number of likes each review has received and whether the current user liked each review. The query selects columns from the ```reviews``` table, including review details, not only the review ID, date, stars, comment, and movie ID but also the user details like the user ID and username. The ```COUNT()``` method counts the number of likes each review has received. The ```JOIN``` clause combines rows from the ```reviews``` table with the ```user``` table to get the reviewer's username and with the ```likes``` table to count the likes.

The query includes a **subquery** to check if the current user has liked each review, returning ```1``` if the current user liked the review and ```NULL``` otherwise. This check is performed by selecting from the ```likes``` table where the ```review_id``` matches the review ID and the ```user_id``` matches the current user's ID. The query filters reviews by the specified movie ID using the ```WHERE``` clause and groups the results by the review ID using the ```GROUP BY``` clause to ensure correct likes. The user experience will be increased by viewing movie reviews, including user engagement insights.


## Dynamic Rendering with Python inside HTML

Here is an example of Dynamic Rendering with Python inside HTML using the ```render_template``` method, which efficiently enables interaction between Python and HTML. I will explain this code in detail:

```.py
return render_template('movie_reddit.html', username=username, results=movies, user_id=user_id)
```

This line of code uses the ```render_template``` function from Flask. This is used to generate and return an HTML page. The function takes the template file name ```'movie_reddit.html'``` and passes the provided variables stored in Python: ```username``` (the logged-in user's name), ```results``` (a list of movies), and ```user_id``` (the user's ID). This allows the display of these variables in HTML files, even though these are stored inside Python, such as the user's name, movie list, and specific user actions. Using ```render_template``` separates the presentation logic (HTML) from the application logic (Python), making the code more maintainable and beneficial for future development.

Here is my first attempt at using dynamic rendering inside ```movie_reddit.html``` with the code above:

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

This code uses **for loops**, which iterates over the ```results``` list, creating a table row ```(<tr>)``` for each item. Each ```<td>``` tag displays an element from the current item ```r``` in the ```results``` list, where ```r``` is expected to be a **list** or **tuple**. The ```{{ r[0] }}```, ```{{ r[1] }}```, ```{{ r[2] }}```, ```{{ r[3] }}```, and ```{{ r[4] }}``` are placeholders that will be replaced by the elements of ```r``` stored in the Python file.
The last ```<td>``` contains a link styled as a button, which directs to a page generated by the ```see_review``` route, passing the ```movie_id``` parameter as ```r[0]```. The ```{% endfor %}``` line ends the **loop**. The movie details and a link to view the reviews for each movie are displayed in table rows that this code created for every movie in the results list.

However, I realized that the code was repetitive and inefficient, so I decided to use a **nested loop** to iterate through the items and create the table rows dynamically. Here is the modified version of the code:

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

This code uses a **nested loop** to iterate through each ```row``` in ```results```, and then for each ```row```, it iterates through each ```col``` (column) to create the table cells. The last column containing the "See" button is added separately outside the inner loop.


# Criteria D: Functionlaity

## Demonstration video
Here is the video of the functionality test as well as highlighting the ease of use for future developers to expand upon it.

https://drive.google.com/file/d/1Nr1C20sKp4H9QzbTCXGERGqoUqDqaZyG/view?usp=drive_link 


# Criteria E: Evaluation 

## Functionality test from 2 users

| Step | Action                                                                                                                                                                                     | Expected Outcome                                                                                                              | User 1 Feedback                                                                                 | User 2 Feedback                                                                      |
|------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| 1    | Open the web application                                                                                                                                                                   | The login page should be displayed.                                                                                           | "The login page loaded quickly."                                                                | "The layout of the login page is clear and easy to see what is going on."            |
| 2    | Attempt to log in with valid credentials (Example: username: user1, password: test123)                                                                                                     | Redirect to the main page, a cookie set with user_id (example: user_id=1).                                                    | "I was redirected to the main page successfully."                                               | "Login worked smoothly and I was taken to the main page."                            |
| 3    | Attempt to log in with invalid credentials (Example: username: user1, password: wrong_pass)                                                                                                | Redirect back to the login page with a pop-up message 'Login failed'.                                                         | "I received a clear error message when using the wrong password."                               | "The popup for invalid login was helpful."                                           |
| 4    | Sign up with a new username and password (Example: username: user2, password: test123, confirm password: test123)                                                                          | Redirect to the login page with a pop-up message 'Signup successful'.                                                         | "The signup process was very smooth and I got a success message."                               | "I liked the feedback after signing up, showing that I succeeded."                   |
| 5    | Sign up with a registered username (Example: username: test1, password: test123, confirm password: test123)                                                                                | Redirect back to the signup page with a pop-up message 'Username already exists'.                                             | "I got a message when trying to use an existing username so I decided to use a different one."  | "I got a clear error message for duplicate username."                                |
| 6    | Navigate to own profile page                                                                                                                                                               | Profile page with user details, number of followers and followings, number of reviews, and number of likes that the user got. | "All my profile details were correctly displayed."                                              | "Profile information and stats were accurate and informative. This is very helpful." |
| 7    | Upload a valid profile picture (Example: test.jpeg)                                                                                                                                        | Profile picture updated and stored in the 'static/profile_pics' directory.                                                    | "Profile picture updated without any issues."                                                   | "The picture upload was quick and easy to use."                                      |
| 8    | Upload an invalid profile picture format (Example: test.pdf)                                                                                                                               | Redirect back with a message 'Invalid file type'.                                                                             | "I received a proper pop-up message for invalid file type."                                     | "I got helpful feedback when trying to upload unsupported file format."              |
| 9    | Follow another user                                                                                                                                                                        | The user will be added to the follows table and redirected to the followed user's profile.                                    | "Following a user was straightforward and worked as expected."                                  | "The follow/following functionality worked well."                                    |
| 10   | Unfollow a user                                                                                                                                                                            | The user will be removed from the follows table and redirected to the unfollowed user's profile.                              | "When I try to unfollow someone, the unfollow action is smooth and redirected correctly."       | "Unfollowing a user was simple and worked perfectly."                                |
| 11   | Add a movie (Example: Title: Inception, Genre: Science Fiction, Director: Christopher Nolan, Release Year: 2010)                                                                           | A new movie will be added to the movies table and displayed on the main page.                                                 | "Adding a movie was easy and the movie appeared in the list."                                   | "The add movie feature is good for me and effective."                                |
| 12   | View all movies                                                                                                                                                                            | All movies are displayed correctly on the all_movies.html page.                                                               | "All movies were listed correctly."                                                             | "The movie list display is adequate and easy to browse."                             |
| 13   | Add a book (Example: Title: The Catcher in the Rye, Author: J.D. Salinger, Genre: Fiction, Release Year: 1951)                                                                             | A new book will be added to the books table and displayed on the all_books page.                                              | "Adding a book was as straightforward as movies and it showed up in the list correctly."        | "Easy to add a book and see it listed."                                              |
| 14   | View all books                                                                                                                                                                             | All books are displayed correctly in the all_books.html page.                                                                 | "All books were displayed accurately."                                                          | "The list of books is easy to navigate and complete."                                |
| 15   | Post a review for a movie (Example: Movie ID: 1 (assuming this is the ID for "Inception"), Comment: "A masterpiece with stunning visuals and unpredictable storyline.", Stars: 5)          | Review added to the reviews table and displayed under the corresponding movie.                                                | "Review submission was smooth and it appeared immediately."                                     | "Posting a review was simple and it showed up correctly."                            |
| 16   | Edit a review for a movie (Example: Movie ID: 1 (assuming this is the ID for "Inception"), Comment: "I did not like the ending", Stars: 2)                                                 | The review is updated in the reviews table and changes are reflected under the movie.                                         | "Editing my review was easy since I just needed to press the "EDIT" button and update quickly." | "The edit feature worked perfectly and updated my review."                           |
| 17   | Like a movie review                                                                                                                                                                        | Review like count increased and like entry added to the likes table.                                                          | "Liking a review was easy and reflected immediately."                                           | "The like functionality is smooth and responsive."                                   |
| 18   | Unlike a movie review                                                                                                                                                                      | Review like count decreased and like entry removed from the likes table.                                                      | "Unliking a review worked as expected and was as quick as liking a review."                     | "The unlike action was simple and worked perfectly."                                 |
| 19   | Delete a movie review                                                                                                                                                                      | The review will be removed from the reviews table and no longer be displayed under the movie table.                           | "Deleting my review was straightforward and reflected right away."                              | "The delete function worked easily and I was able to remove my review."              |
| 20   | Post a review for a book (Example: Book ID: 1 (assuming this is the ID for "The Catcher in the Rye"), Comment: "A complicated story that captures the essence of teenage life.", Stars: 4) | Review added to the reviews table and displayed under the corresponding book.                                                 | "Review submission was smooth and it appeared immediately."                                     | "Posting a review was simple and it showed up correctly."                            |
| 21   | Edit a review for a book (Example: Book ID: 1 (assuming this is the ID for "The Catcher in the Rye"), Comment: "A story was a bit creche", Stars: 2)                                       | The review is updated in the reviews table and changes are reflected under the book.                                          | "Editing my review was easy since I just needed to press the "EDIT" button and update quickly." | "The edit feature worked perfectly and updated my review."                           |
| 22   | Like a book review                                                                                                                                                                         | Review like count increased and like entry added to the likes table.                                                          | "Liking a review was easy and reflected immediately."                                           | "The like functionality is smooth and responsive."                                   |
| 23   | Unlike a book review                                                                                                                                                                       | Review like count decreased and like entry removed from the likes table.                                                      | "Unliking a review worked as expected and was as quick as liking a review."                     | "The unlike action was simple and worked perfectly."                                 |
| 24   | Delete a book review                                                                                                                                                                       | The review will be removed from the reviews table and no longer be displayed under the book table.                            | "Deleting my review was straightforward and reflected right away."                              | "The delete function worked easily and I was able to remove my review."              |
| 25   | Logout                                                                                                                                                                                     | Redirect to the login page and user_id cookie deleted.                                                                        | "Logout was successful and took me to the login page."                                          | "The logout function worked correctly and cleared my session."                       |


## Evaluation by Peers
My peer is very satisfied with the website, especially the login screen and the buttons such as "LIKE" and "FOLLOW" as detailed in Figure 3 and Figure 4. However, they suggested an area for improvement: adding functionality on the 'Profile' page that allows users to see lists of their followers and whom they are following. Implementing this feature could allow user engagement by making the social connections within the platform more visible and accessible.

## Evaluation of beta testing
I received a feedback from the user and they are very satisfied with the website, as detailed in Figure 5. They especially liked how the user can visualise actual insights such as how many followers does the user have, and how many likes that the user got so far. However, they suggested an area for improvement: add a confirmation dialog for when deleting posts as the user had accidentally deleted a few posts while testing. This will enables users to prevent data loss. 


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


# Work Cited

https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-iv-database 
---. “The Flask Mega-Tutorial, Part IV: Database.” Copyright (C) 2012-2024 Miguel Grinberg, blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-iv-database.

https://www.w3schools.com/sql/ 
SQL Tutorial. 

https://jinja.palletsprojects.com/en/3.1.x
Jinja — Jinja Documentation (3.1.x). jinja.palletsprojects.com/en/3.1.x.

https://pypi.org/project/Werkzeug 
“Werkzeug.” PyPI, 5 May 2024, pypi.org/project/Werkzeug.

https://docs.python.org/3/library/os.html
“Os — Miscellaneous Operating System Interfaces.” Python Documentation, docs.python.org/3/library/os.html.

