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
## sqlalchemy
    Sqlalchemy is a ORM which is what our fastapi application is going to use, to create database and be able to create a connection with database.
    Being able to use all the database records in the application.
    install sqlalchemy:
        pip install sqlalchemy
### Create database and connect postgres db
    Install sqlalchemy
    Step1) Create a database.py file inside a TodoApp folder
        Code:
            from sqlalchemy import create_engine
            from sqlalchemy.orm import sessionmaker
            from sqlalchemy.ext.declarative import declarative_base
            
            # SQLALCHEMY_DATABASE_URL = "sqlite:///./users.db"
            SQLALCHEMY_DATABASE_URL = 'postgresql://postgres:koushik@localhost:5488/new_db'
            
            engine = create_engine(SQLALCHEMY_DATABASE_URL)
            
            SessionLocal = sessionmaker(autoflush=False, autocommit=False, bind=engine)
            
            Base = declarative_base()
            
            def get_db():
                db = SessionLocal()
                try:
                    yield db
                finally:
                    db.close()
                    
    Step2) Create a models.py file to add table in it
        Code:
            from database import Base
            from sqlalchemy import Column, Integer, String, Boolean
            
            class Todos(Base):
                __tablename__ = 'todos' # Naming table
            
                id = Column(Integer, primary_key=True, index=True) # Id column
                title = Column(String)
                description = Column(String)
                priority = Column(Integer)
                complete = Column(Boolean, default=False)
                
    Step3) Create a main.py that is the entry of the project
        Code:
            from fastapi import FastAPI
            from models import Base
            from database import engine
            
            app = FastAPI()
            
            Base.metadata.create_all(bind=engine) # will create everything from database.py file and models.py file to be able to create database with todo tables
    Step4) Run the application
        uvicorn main:app --reload
    Step5) See the results in pgadmin
## Crud operations with database(main crud)
    Code:
        from typing import Annotated
        from fastapi import Depends, FastAPI, HTTPException, Path, status
        from pydantic import BaseModel, Field
        from models import Base, Todos
        from database import SessionLocal, engine
        from sqlalchemy.orm import Session
        
        app = FastAPI()
        
        Base.metadata.create_all(bind=engine) # will create everything from database.py file and models.py file to be able to create database with todo tables
        
        def get_db():
            db = SessionLocal()
            try:
                yield db
            finally:
                db.close()
        
        db_dependency = Annotated[Session, Depends(get_db)]
        # Depends is Dependency injection means we want to do something before it executes 
        # That allows us to do something behind the scenes and then inject the dependencies that our function relies on
        
        class TodoRequest(BaseModel):
            title: str = Field(min_length=3, max_length=100)
            description: str
            priority: int
            complete: bool
        
        @app.get("/", status_code=status.HTTP_200_OK)
        async def get_home(db: db_dependency):
            return db.query(Todos).all()
        
        @app.get("/todo/{todo_id}/", status_code=status.HTTP_200_OK)
        async def get_todo_details(db: db_dependency, todo_id: int = Path(gt=0)): # Here if the todo_id <0 it will throw exception that is default validation happening no need to exceptions for everything
            todo_data = db.query(Todos).filter(Todos.id==todo_id).first()
            if todo_data is not None:
                return todo_data
            raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Todo not found")
        
        @app.post("/todo/", status_code=status.HTTP_201_CREATED)
        async def create_todo(db: db_dependency, todo_request: TodoRequest):
            todo_model = Todos(**todo_request.model_dump()) # Converting object to dictionary
        
            db.add(todo_model)
            db.commit() # flushing and Actually doing transaction to database
        
        @app.put("/todo/{todo_id}/", status_code=status.HTTP_204_NO_CONTENT)
        async def update_todo(db: db_dependency, todo_request: TodoRequest, todo_id: int = Path(gt=0)):
            todo_model = db.query(Todos).filter(Todos.id == todo_id).first()
            if todo_model is None:
                raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="todo id not found")
            todo_model.title = todo_request.title
            todo_model.description = todo_request.description
            todo_model.priority = todo_request.priority
            todo_model.complete = todo_request.complete
        
            db.add(todo_model)
            db.commit()
        
        @app.delete("/todo/{todo_id}/", status_code=status.HTTP_204_NO_CONTENT)
        async def delete_todo(db: db_dependency, todo_id: int):
            todo_model = db.query(Todos).filter(Todos.id == todo_id).first()
            if todo_model is None:
                raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="todo id not found")
            db.query(Todos).filter(Todos.id == todo_id).delete()
            db.commit()


        



    
        

    
            

