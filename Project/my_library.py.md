```.py
import sqlite3
from passlib.hash import sha256_crypt

hasher = sha256_crypt.using(rounds = 30000)


def get_hash(text:str):
    return hasher.hash(text)


def check_hash(input_hash, text):
    return hasher.verify(text, input_hash)


class DatabaseBridge:
    def __init__(self, name:str):
        self.name_db = name
        #Step 1: create a connection to the file
        self.connection = sqlite3.connect(self.name_db)
        self.cursor = self.connection.cursor()

    def run_query(self, query:str):
        self.cursor.execute(query) #run query
        self.connection.commit() #save query

    def insert(self,query:str):
        self.run_query(query)

    def search(self,query:str, multiple=False):
        results = self.cursor.execute(query)
        if multiple:
            return results.fetchall() #return multiple rows
        return results.fetchone() #return a single value

    def create(self):
        query = """CREATE TABLE if not exists users(
        id INTEGER PRIMARY KEY,
        email text NOT NULL unique,
        password VARCHAR(256) not null,
        username text not null
        )"""
        self.run_query(query)

    def update(self, query, params):
        self.connection.cursor().execute(query, params)
        self.connection.commit()

    def close(self):
        self.connection.close()

    # def insert(self, query, params=None):
    #     cursor = self.connection.cursor()
    #     if params:
    #         cursor.execute(query, params)
    #     else:
    #         cursor.execute(query)
    #     self.connection.commit()
    #
    #
    #
    # def search(self, query, params=None, multiple=False):
    #     cursor = self.connection.cursor()
    #     if params:
    #         cursor.execute(query, params)
    #     else:
    #         cursor.execute(query)
    #
    #     if multiple:
    #         return cursor.fetchall()
    #     else:
    #         return cursor.fetchone()



```
