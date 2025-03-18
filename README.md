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
            from typing import Optional
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
                id: Optional[int] = Field(description='This field optional yes', default=None) #id: Optional[int] = None
                title: str = Field(min_length=3, max_length=5)
                description: str
                author: str
                rating: int = Field(gt=0)
            
                # Customize Example in swagger
                # To create more descriptive request with swagger documentation
                model_config = {
                    "json_schema_extra":{
                        "example": {
                            "title": "new",
                            "description": "book desc",
                            "author": "abc",
                            "rating": 13
                        }
                    }
                }
            
            BOOKS = [
                Book(1,"abc","abc desc", "abc1", 5),
                Book(2,"abcd","abc desc", "abc1", 5),
            ]
            
            @app.get("/books/")
            async def get_all_books():
                return {"data": BOOKS}
            
            @app.get("/")
            async def home():
                return {"data": "welcome"}
            
            @app.post("/create-book/")
            async def create_book(req_data:BookRequest):
                data = Book(**req_data.model_dump()) # Converting to dictanory #Here we are converting actual book request to book
                updated = BOOKS.append(data)
                return {"data": updated}

### validate path param
    from fastapi import FastAPI, Path
    @app.get("/test/{test_id}/")
    async def validate_id(test_id:int=Path(gt=2)):
        return {"data": "hello"}
    url : http://localhost:8000/test/1/ failure
### validate query param
    @app.get("/test_param/")
    async def validate_query_param(test_id:int = Query(gt=0, lt=10)):
        return {"data": "hello"}
        
## Customize example in swagger
    Add this to class
      model_config = {
        "json_schema_extra":{
            "example": {
                "title": "new",
                "description": "book desc",
                "author": "abc",
                "rating": 13
            }
        }
    }
    if array of values needed use "examples":[{},{}]

### Status Codes
    An http status code helps the client to understand what happened on the server side application
    Status codes are international standards on how a client/server should handle the result of the request
    It allows everyone who sends a request to know if there is submission is sucess/ failure
    @app.get("/books/{book_id}/")
    async def get_book_id(book_id: int):
        for book in BOOKS:
            print(book)
            if book.id == book_id:
                return {"data": book}
        raise HTTPException(status_code=404, detail="book not found")

    
            

