# FastAPI
## Simple CRUD application code
      from fastapi import FastAPI, Body
      app = FastAPI()
      
      BOOKS = [
          {
              "title": "book1",
              "author":"abc",
              "category": "anime"
          },
          {
              "title": "book1",
              "author":"abc",
              "category": "science"
          },
          {
              "title": "book1",
              "author": "abc2",
              "category": "science"
          }
      ]
      
      @app.get("/books/")
      async def get_books():
          return {"data": BOOKS}
      
      @app.get("/books/{author}/")
      async def filter_by_author_category(author:str, category: str):
          filtered_books = []
          for book in BOOKS:
              if(book.get("category").casefold() == category.casefold()) and \
                  (book.get("author") == author):
                  filtered_books.append(book)
          return {"data": filtered_books}
      
      @app.post("/books/create_book/")
      async def create_book(new_body=Body()):
          data = BOOKS.append(new_body)
          return {"data": data}
      
      @app.put("/books/update-book/")
      async def update_book(updated_data=Body()):
          for i in range(len(BOOKS)):
              if(BOOKS[i].get("title") == updated_data.get("title")):
                  BOOKS[i] = updated_data
          return {"data": BOOKS}
      
      @app.delete("/books/{author}/")
      async def delete_book(author:str):
          for i in range(len(BOOKS)):
              if(BOOKS[i].get("author") == author):
                  BOOKS.pop(i)
                  break
          return {"data": BOOKS}
## pydantic
      validation of fields using pydantic
      Simple post operation
            from fastapi import FastAPI
            from pydantic import BaseModel, Field
            
            app = FastAPI()
            
            
            class Book:
                id: int
                title: str
                description: str
                author: str
                rating: int
            
                def __init__(self, id, title, description, author, rating):
                    self.id = id
                    self.title = title
                    self.description = description
                    self.author = author
                    self.rating = rating
            
            class BookRequest(BaseModel):
                id: int
                title: str = Field(min_length=3, max_length=5)
                description: str
                author: str
                rating: int = Field(gt=0)
            
            BOOKS = [
                Book(1,"abc","abc desc", "abc1", 5),
                Book(2,"abcd","abc desc", "abc1", 5),
            ]
            
            @app.get("/books/")
            async def get_all_books():
                return {"data": BOOKS}
            
            @app.post("/create-book/")
            async def create_book(req_data:BookRequest):
                data = Book(**req_data.model_dump()) # Converting to dictanory here we are converting actual book request to book
                updated = BOOKS.append(data)
                return {"data": updated}


