```.py
from datetime import datetime
from flask import Flask, render_template, request, make_response, redirect, url_for
from my_library import DatabaseBridge, get_hash, check_hash
import os
from werkzeug.utils import secure_filename

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = 'static/profile_pics'
app.config['ALLOWED_EXTENSIONS'] = {'png', 'jpg', 'jpeg', 'gif'}


def allowed_file(filename):
    return '.' in filename and filename.rsplit('.', 1)[1].lower() in app.config['ALLOWED_EXTENSIONS']


def save_profile_pic(file):
    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
        return filename
    return None


@app.route('/profile/<int:user_id>', methods=['GET', 'POST'])
def profile(user_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')

    if request.method == 'POST':
        if 'profile_picture' in request.files:
            file = request.files['profile_picture']
            if file and allowed_file(file.filename):
                filename = secure_filename(file.filename)
                file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
                db.run_query(f"UPDATE user SET profile_picture='{filename}' WHERE id={user_id}")

    user = db.search(f"SELECT * FROM user WHERE id={user_id}", multiple=False)
    following = db.search(f"SELECT * FROM follows WHERE follower_id={current_user_id} AND followed_id={user_id}",
                          multiple=False) is not None
    review_count = db.search(f"SELECT COUNT(*) FROM reviews WHERE user_id={user_id}", multiple=False)[0]

    like_count = db.search(f"""
        SELECT COUNT(*)
        FROM likes l
        JOIN reviews r ON l.review_id = r.id
        WHERE r.user_id={user_id}
    """, multiple=False)[0]

    follower_count = db.search(f"SELECT COUNT(*) FROM follows WHERE followed_id={user_id}", multiple=False)[0]
    following_count = db.search(f"SELECT COUNT(*) FROM follows WHERE follower_id={user_id}", multiple=False)[0]

    db.close()
    return render_template('profile.html', user=user, following=following, review_count=review_count,
                           like_count=like_count, follower_count=follower_count, following_count=following_count)


@app.route('/profile')
def profile_self():
    user_id = request.cookies.get('user_id')
    if user_id:
        return redirect(url_for('profile', user_id=user_id))
    return redirect(url_for('login', message='Please log in to view your profile'))


@app.route('/follow/<int:user_id>', methods=['POST'])
def follow_user(user_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')
    db.run_query(f"INSERT INTO follows (follower_id, followed_id) VALUES ({current_user_id}, {user_id})")
    db.close()
    return redirect(url_for('profile', user_id=user_id))


@app.route('/unfollow/<int:user_id>', methods=['POST'])
def unfollow_user(user_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')
    db.run_query(f"DELETE FROM follows WHERE follower_id={current_user_id} AND followed_id={user_id}")
    db.close()
    return redirect(url_for('profile', user_id=user_id))


@app.route('/', methods=['GET', 'POST'])
def login():
    if request.method == "POST":
        uname = request.form.get('uname')
        psw = request.form.get('psw')
        db = DatabaseBridge('user.db')
        user = db.search(f"SELECT * from user where uname == '{uname}'", multiple=False)
        db.close()
        if user and check_hash(user[2], psw):  # user[2] is the hashed password
            response = make_response(redirect(url_for('main')))
            response.set_cookie('user_id', str(user[0]))  # Cookie expires in 30 days
            return response
        else:
            return redirect(url_for('login', message='Login failed'))

    message = request.args.get('message')
    title = "Login to the Movie Reddit"
    return render_template("login.html", title=title, message=message)


@app.route('/signup', methods=['GET', 'POST'])
def signup():
    if request.method == "POST":
        uname = request.form.get('uname')
        psw = request.form.get('psw')
        confirm_psw = request.form.get('confirm_psw')
        if psw != confirm_psw:
            return redirect(url_for('signup', message='Password does not match'))

        db = DatabaseBridge('user.db')
        existing_user = db.search(f"SELECT * from user where uname == '{uname}'", multiple=False)
        if existing_user:
            db.close()
            return redirect(url_for('signup', message='Username already exists'))

        hashed_psw = get_hash(psw)

        if not os.path.exists(app.config['UPLOAD_FOLDER']):
            os.makedirs(app.config['UPLOAD_FOLDER'])

        profile_picture = 'default.png'
        if 'profile_picture' in request.files:
            file = request.files['profile_picture']
            if file and allowed_file(file.filename):
                filename = secure_filename(file.filename)
                file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
                profile_picture = filename

        db.run_query(
            f"INSERT INTO user(uname, psw, profile_picture) VALUES ('{uname}', '{hashed_psw}', '{profile_picture}')")
        db.close()

        return redirect(url_for('login', message='Signup successful'))

    message = request.args.get('message')
    title = "Signup to the WebApp"
    return render_template("signup.html", title=title, message=message)


@app.route('/edit_profile/<int:user_id>', methods=['POST'])
def edit_profile(user_id):
    if 'profile_picture' not in request.files:
        return redirect(url_for('profile', user_id=user_id, message='No file part'))

    file = request.files['profile_picture']
    if file.filename == '':
        return redirect(url_for('profile', user_id=user_id, message='No selected file'))

    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
        profile_picture = filename

        db = DatabaseBridge('user.db')
        db.run_query(f"UPDATE user SET profile_picture='{profile_picture}' WHERE id={user_id}")
        db.close()

        return redirect(url_for('profile', user_id=user_id))

    return redirect(url_for('profile', user_id=user_id, message='Invalid file type'))


@app.route('/main')
def main():
    user_id = request.cookies.get('user_id')
    username = None

    if user_id:
        db = DatabaseBridge('user.db')
        user = db.search(f"SELECT * FROM user WHERE id == {user_id}", multiple=False)
        db.close()
        if user:
            username = user[1]  # Assuming user[1] is the username

    db = DatabaseBridge('user.db')
    movies = db.search(query='SELECT * from movies', multiple=True)
    books = db.search(query='SELECT * from books', multiple=True)
    db.close()

    return render_template('main.html', username=username, movies=movies, books=books, user_id=user_id)


@app.route('/movie_reddit')
def movie_reddit():
    user_id = request.cookies.get('user_id')
    username = None

    if user_id:
        db = DatabaseBridge('user.db')
        user = db.search(f"SELECT * FROM user WHERE id == {user_id}", multiple=False)
        db.close()
        if user:
            username = user[1]  # Assuming user[1] is the username

    db = DatabaseBridge('user.db')
    movies = db.search(query='SELECT * from movies', multiple=True)
    db.close()

    return render_template('movie_reddit.html', username=username, results=movies, user_id=user_id)


@app.route('/book_reddit')
def book_reddit():
    user_id = request.cookies.get('user_id')
    username = None

    if user_id:
        db = DatabaseBridge('user.db')
        user = db.search(f"SELECT * FROM user WHERE id == {user_id}", multiple=False)
        db.close()
        if user:
            username = user[1]  # Assuming user[1] is the username

    db = DatabaseBridge('user.db')
    books = db.search(query='SELECT * from books', multiple=True)
    db.close()

    return render_template('book_reddit.html', username=username, results=books, user_id=user_id)


@app.route('/movie/<int:movie_id>', methods=['GET', 'POST'])
def see_review(movie_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')
    res = db.search(query=f"SELECT * from movies where id={movie_id}", multiple=False)

    revs = db.search(query=f"""
        SELECT r.id, r.date, r.stars, r.comment, r.movie_id, COUNT(l.id) as likes, r.user_id, u.uname,
               (SELECT 1 FROM likes WHERE review_id = r.id AND user_id = {current_user_id}) as liked
        FROM reviews r
        JOIN user u ON r.user_id = u.id
        LEFT JOIN likes l ON r.id = l.review_id
        WHERE r.movie_id = {movie_id}
        GROUP BY r.id
    """, multiple=True)

    edit_review_id = None
    comment = ''
    stars = 1

    if request.method == "POST":
        edit_review_id = request.form.get('edit_review_id')
        comment = request.form.get('comment')
        stars = request.form.get('stars')
        user_id = request.cookies.get('user_id')

        if edit_review_id:
            db.run_query(query=f"UPDATE reviews SET comment='{comment}', stars={stars} WHERE id={edit_review_id}")
        else:
            review = (f"INSERT INTO reviews (date, stars, comment, movie_id, user_id) "
                      f"VALUES ('{datetime.now()}', {stars}, '{comment}', {movie_id}, {user_id})")
            db.run_query(query=review)

        db.close()
        return redirect(url_for('see_review', movie_id=movie_id))

    db.close()
    return render_template('movie.html', movie=res, reviews=revs, edit_review_id=edit_review_id, comment=comment,
                           stars=stars, current_user_id=int(current_user_id))


@app.route('/movie/<int:movie_id>/edit/<int:review_id>', methods=['GET', 'POST'])
def edit_review(movie_id, review_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')

    # Verify that the current user is the owner of the review
    review = db.search(f"SELECT user_id FROM reviews WHERE id={review_id}", multiple=False)

    if review and review[0] != int(current_user_id):
        db.close()
        return redirect(url_for('see_review', movie_id=movie_id))

    if request.method == 'POST':
        comment = request.form.get('comment')
        stars = request.form.get('stars')
        edit_review_id = request.form.get('edit_review_id')
        db.run_query(query=f"UPDATE reviews SET comment='{comment}', stars={stars} WHERE id={edit_review_id}")
        db.close()
        return redirect(url_for('see_review', movie_id=movie_id))

    movie = db.search(query=f"SELECT * from movies where id={movie_id}", multiple=False)
    edit_review = db.search(query=f"SELECT * from reviews where id={review_id}", multiple=False)
    review = db.search(query=f"""
        SELECT r.id, r.date, r.stars, r.comment, r.movie_id, COUNT(l.id) as likes, r.user_id, u.uname,
               (SELECT 1 FROM likes WHERE review_id = r.id AND user_id = {current_user_id}) as liked
        FROM reviews r
        JOIN user u ON r.user_id = u.id
        LEFT JOIN likes l ON r.id = l.review_id
        WHERE r.movie_id = {movie_id}
        GROUP BY r.id
    """, multiple=True)
    comment = edit_review[3]
    stars = edit_review[2]
    edit_review_id = edit_review[0]
    db.close()
    return render_template('movie.html', movie=movie, reviews=review, stars=stars, comment=comment,
                           edit_review_id=edit_review_id, current_user_id=int(current_user_id))


@app.route('/add_movie', methods=['GET', 'POST'])
def add_movie():
    if request.method == 'POST':
        title = request.form.get('title')
        genre = request.form.get('genre')
        director = request.form.get('director')
        release_year = request.form.get('release_year')

        db = DatabaseBridge('user.db')
        db.run_query(f"INSERT INTO movies (title, genre, director, release_year) VALUES ('{title}', '{genre}',"
                     f" '{director}', {release_year})")
        db.close()

        return redirect(url_for('main'))

    return render_template('add_movie.html')


@app.route('/see_all')
def get_all_movies():
    db = DatabaseBridge('user.db')
    res = db.search(query='SELECT * from movies', multiple=True)
    db.close()
    return render_template('all_movies.html', results=res)


@app.route('/logout')
def logout():
    response = make_response(redirect(url_for('login')))
    response.delete_cookie('user_id')
    return response


@app.route('/movie/<int:movie_id>/like/<int:review_id>', methods=['POST'])
def like_review(movie_id, review_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')
    db.run_query(f"INSERT INTO likes (user_id, review_id) VALUES ({current_user_id}, {review_id})")
    db.close()
    return redirect(url_for('see_review', movie_id=movie_id))


@app.route('/movie/<int:movie_id>/unlike/<int:review_id>', methods=['POST'])
def unlike_review(movie_id, review_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')
    db.run_query(f"DELETE FROM likes WHERE user_id={current_user_id} AND review_id={review_id}")
    db.close()
    return redirect(url_for('see_review', movie_id=movie_id))


@app.route('/movie/<int:movie_id>/remove/<int:review_id>', methods=['POST'])
def delete_review(movie_id, review_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')

    # Verify that the current user is the owner of the review
    review = db.search(f"SELECT user_id FROM reviews WHERE id={review_id}", multiple=False)

    if review and review[0] == int(current_user_id):
        query = f"DELETE from reviews where id={review_id}"
        db.run_query(query)

    db.close()
    return redirect(url_for('see_review', movie_id=movie_id))


@app.route('/books')
def get_all_books():
    db = DatabaseBridge('user.db')
    books = db.search(query='SELECT * from books', multiple=True)
    db.close()
    return render_template('all_books.html', results=books)


@app.route('/add_book', methods=['GET', 'POST'])
def add_book():
    if request.method == 'POST':
        title = request.form.get('title')
        author = request.form.get('author')
        genre = request.form.get('genre')
        release_year = request.form.get('release_year')

        db = DatabaseBridge('user.db')
        db.run_query(f"INSERT INTO books (title, author, genre, release_year) VALUES ('{title}', '{author}', '{genre}', {release_year})")
        db.close()

        return redirect(url_for('get_all_books'))

    return render_template('add_book.html')


@app.route('/book/<int:book_id>', methods=['GET', 'POST'])
def see_book_review(book_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')
    res = db.search(query=f"SELECT * from books where id={book_id}", multiple=False)

    revs = db.search(query=f"""
        SELECT r.id, r.date, r.stars, r.comment, r.book_id, COUNT(l.id) as likes, r.user_id, u.uname,
               (SELECT 1 FROM likes WHERE review_id = r.id AND user_id = {current_user_id}) as liked
        FROM book_reviews r
        JOIN user u ON r.user_id = u.id
        LEFT JOIN likes l ON r.id = l.review_id
        WHERE r.book_id = {book_id}
        GROUP BY r.id
    """, multiple=True)

    edit_review_id = None
    comment = ''
    stars = 1

    if request.method == "POST":
        edit_review_id = request.form.get('edit_review_id')
        comment = request.form.get('comment')
        stars = request.form.get('stars')
        user_id = request.cookies.get('user_id')

        if edit_review_id:
            db.run_query(query=f"UPDATE book_reviews SET comment='{comment}', stars={stars} WHERE id={edit_review_id}")
        else:
            review = (f"INSERT INTO book_reviews (date, stars, comment, book_id, user_id) "
                      f"VALUES ('{datetime.now()}', {stars}, '{comment}', {book_id}, {user_id})")
            db.run_query(query=review)

        db.close()
        return redirect(url_for('see_book_review', book_id=book_id))

    db.close()
    return render_template('book.html', book=res, reviews=revs, edit_review_id=edit_review_id, comment=comment,
                           stars=stars, current_user_id=int(current_user_id))


@app.route('/book/<int:book_id>/edit/<int:review_id>', methods=['GET', 'POST'])
def edit_book_review(book_id, review_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')

    # Verify that the current user is the owner of the review
    review = db.search(f"SELECT user_id FROM book_reviews WHERE id={review_id}", multiple=False)

    if review and review[0] != int(current_user_id):
        db.close()
        return redirect(url_for('see_book_review', book_id=book_id))

    if request.method == 'POST':
        comment = request.form.get('comment')
        stars = request.form.get('stars')
        edit_review_id = request.form.get('edit_review_id')
        db.run_query(query=f"UPDATE book_reviews SET comment='{comment}', stars={stars} WHERE id={edit_review_id}")
        db.close()
        return redirect(url_for('see_book_review', book_id=book_id))

    book = db.search(query=f"SELECT * from books where id={book_id}", multiple=False)
    edit_review = db.search(query=f"SELECT * from book_reviews where id={review_id}", multiple=False)
    review = db.search(query=f"""
        SELECT r.id, r.date, r.stars, r.comment, r.book_id, COUNT(l.id) as likes, r.user_id, u.uname,
               (SELECT 1 FROM likes WHERE review_id = r.id AND user_id = {current_user_id}) as liked
        FROM book_reviews r
        JOIN user u ON r.user_id = u.id
        LEFT JOIN likes l ON r.id = l.review_id
        WHERE r.book_id = {book_id}
        GROUP BY r.id
    """, multiple=True)
    comment = edit_review[3]
    stars = edit_review[2]
    edit_review_id = edit_review[0]
    db.close()
    return render_template('book.html', book=book, reviews=review, stars=stars, comment=comment,
                           edit_review_id=edit_review_id, current_user_id=int(current_user_id))


@app.route('/book/<int:book_id>/like/<int:review_id>', methods=['POST'])
def like_book_review(book_id, review_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')
    db.run_query(f"INSERT INTO likes (user_id, review_id) VALUES ({current_user_id}, {review_id})")
    db.close()
    return redirect(url_for('see_book_review', book_id=book_id))


@app.route('/book/<int:book_id>/unlike/<int:review_id>', methods=['POST'])
def unlike_book_review(book_id, review_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')
    db.run_query(f"DELETE FROM likes WHERE user_id={current_user_id} AND review_id={review_id}")
    db.close()
    return redirect(url_for('see_book_review', book_id=book_id))


@app.route('/book/<int:book_id>/remove/<int:review_id>', methods=['POST'])
def delete_book_review(book_id, review_id):
    current_user_id = request.cookies.get('user_id')
    db = DatabaseBridge('user.db')

    # Verify that the current user is the owner of the review
    review = db.search(f"SELECT user_id FROM book_reviews WHERE id={review_id}", multiple=False)

    if review and review[0] == int(current_user_id):
        query = f"DELETE from book_reviews where id={review_id}"
        db.run_query(query)

    db.close()
    return redirect(url_for('see_book_review', book_id=book_id))

```
