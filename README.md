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

