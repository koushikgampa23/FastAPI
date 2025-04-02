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
## Routers
### add router basic
    Step1) Create a router folder add this file auth.py
        from fastapi import APIRouter
        router = APIRouter()
        @router.get("/auth/")
        async def get_todo():
            return {"msg":"inside some auth router"}
    Step2) In the main.py import router
        from router import auth,todos
        app.include_router(auth)
        app.include_router(todos)
### Updated main, todos app
    main.py
        from fastapi import FastAPI
        from models import Base
        from database import engine
        from routers import auth, todos
        
        app = FastAPI()
        
        Base.metadata.create_all(bind=engine) # will create everything from database.py file and models.py file to be able to create database with todo tables
        app.include_router(auth.router)
        app.include_router(todos.router)
    todos.py
        from typing import Annotated
        from fastapi import Depends, APIRouter, HTTPException, Path, status
        from pydantic import BaseModel, Field
        from models import Todos
        from database import SessionLocal
        from sqlalchemy.orm import Session
        
        router = APIRouter()
        
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
        
        @router.get("/todo/", status_code=status.HTTP_200_OK)
        async def get_home(db: db_dependency):
            return db.query(Todos).all()
        
        @router.get("/todo/{todo_id}/", status_code=status.HTTP_200_OK)
        async def get_todo_details(db: db_dependency, todo_id: int = Path(gt=0)): # Here if the todo_id <0 it will throw exception that is default validation happening no need to exceptions for everything
            todo_data = db.query(Todos).filter(Todos.id==todo_id).first()
            if todo_data is not None:
                return todo_data
            raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Todo not found")
        
        @router.post("/todo/", status_code=status.HTTP_201_CREATED)
        async def create_todo(db: db_dependency, todo_request: TodoRequest):
            todo_model = Todos(**todo_request.model_dump()) # Converting object to dictionary
        
            db.add(todo_model)
            db.commit() # flushing and Actually doing transaction to database
        
        @router.put("/todo/{todo_id}/", status_code=status.HTTP_204_NO_CONTENT)
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
        
        @router.delete("/todo/{todo_id}/", status_code=status.HTTP_204_NO_CONTENT)
        async def delete_todo(db: db_dependency, todo_id: int):
            todo_model = db.query(Todos).filter(Todos.id == todo_id).first()
            if todo_model is None:
                raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="todo id not found")
            db.query(Todos).filter(Todos.id == todo_id).delete()
            db.commit()

## User module(hash password using bcrypt)
    Step1) Create User database and iam modifying todos and user table is the foreign key of todos table
    Step1) update models.py
        from database import Base
        from sqlalchemy import Column, Integer, String, Boolean, ForeignKey
        
        class Users(Base):
            __tablename__ = "users"
            
            id = Column(Integer, primary_key=True)
            email = Column(String, unique=True)
            username = Column(String, unique=True)
            first_name = Column(String)
            last_name = Column(String)
            hashed_password = Column(String)
            is_active = Column(Boolean)
            default = Column(Boolean, default=True)
            role = Column(String)
        
        class Todos(Base):
            __tablename__ = 'todos' # Naming table
        
            id = Column(Integer, primary_key=True, index=True) # Id column
            title = Column(String)
            description = Column(String)
            priority = Column(Integer)
            complete = Column(Boolean, default=False)
            owner_id = Column(Integer, ForeignKey("users.id"))

    Step2) auth.py
        from fastapi import APIRouter, Depends
        from pydantic import BaseModel
        from database import SessionLocal
        from typing import Annotated
        from sqlalchemy.orm import Session
        from models import Users
        from passlib.context import CryptContext
        
        router = APIRouter()
        
        bcrypt_context = CryptContext(schemes=['bcrypt'], deprecated="auto")
        
        class UserRequest(BaseModel):
            email: str
            username: str
            first_name: str
            last_name: str
            password: str
            is_active: bool
            default: bool
            role: str
        
        def get_db():
            db = SessionLocal()
            try:
                yield db
            finally:
                db.close()
        
        db_dependency = Annotated[Session, Depends(get_db)]
        
        @router.post("/auth/")
        async def create_user(db: db_dependency, user_request: UserRequest):
            # we are not using **user_request since we are storing password as hashed_password in database
            user_model = Users(
                email = user_request.email,
                username = user_request.username,
                first_name = user_request.first_name,
                last_name = user_request.last_name,
                hashed_password = bcrypt_context.hash(user_request.password), # Hashing Here
                is_active = user_request.is_active,
                default = user_request.default,
                role = user_request.role
            )
            db.add(user_model)
            db.commit()
## Authentication
    if we use OAuth2PasswordRequestForm that gives the authentication behaviour to the endpoint
    if we access this endpoint it asks for username, password, client id(optional), client secret(optional)
    Code:
        from fastapi.security import OAuth2PasswordRequestForm
        # Function for authentication
        def authenticate_user(username: str, password: str, db):
            user = db.query(Users).filter(Users.username == username).first()
            if not user: # Nothing from database
                return False
            if not bcrypt_context.verify(password, user.hashed_password): # not correct password
                return False
            return True
        
        @router.post("/token/")
        async def login_for_access_token(formdata: Annotated[OAuth2PasswordRequestForm, Depends()], db: db_dependency):
            user = authenticate_user(formdata.username, formdata.password, db)
            if not user:
                return "Failed authentication"
            return "authentication success"

![tcsglobal udemy com_course_fastapi-the-complete-course_learn_lecture_29025928](https://github.com/user-attachments/assets/e19df233-cad9-4d9d-b36e-ba5cf5a7c211)

## JSON web tokens
    Q) What is JSON web tokens?
    JSON web token is a self contained way to securly transmit data and information between two parts using json object.
    Json web tokens are trusted since each token is digitally signed, which in return allows server to know if the jwt token has been changed at all.
    Jwt can be used when dealing with authorization.
    JWT is the great way to exchange information between client and server.
    JWT Structure
          Header - contains algo and type of token, header is encoded with base 64.
          payload - contains actual data of the user. The payloads data contains claims. and there are three different claims
                      Registered - Iss(Issuer), sub(subject), exp(expiry)
                      Public 
                      Private
                  It is also encoded with base 64
          signature - JWT Signature is created by using the algorithm in the header to hash out the encoded header, encoded payload with a secret.
                      The secret can be anything that has been saved some where in the server not accessable to the client.
### Generating JWT tokens
    Step1) pip install "pyjwt"
    import jwt
    # Function to generate jwt key
        def create_access_token(username: str, user_id: int, expires_delta: timedelta):
            encode = {"sub": username, "id": user_id}
            expires = datetime.now(timezone.utc) + expires_delta
            encode.update({"exp": expires})
            return jwt.encode(encode, SECRET_KEY, algorithm=ALGORITHM)
    Now our existing code changed like this
        def authenticate_user(username: str, password: str, db):
            user = db.query(Users).filter(Users.username == username).first()
            if not user: # Nothing from database
                return False
            if not bcrypt_context.verify(password, user.hashed_password): # not correct password
                return False
            return user
            
        @router.post("/token/", response_model=Token)
        async def login_for_access_token(formdata: Annotated[OAuth2PasswordRequestForm, Depends()], db: db_dependency):
            user = authenticate_user(formdata.username, formdata.password, db)
            if not user:
                return "Failed authentication"
            token = create_access_token(user.username, user.id, timedelta(minutes=10))
            return {"access_token": token, "token_type": "Bearer"}
### Verifying jwt token
    Step1) from fastapi.security import OAuth2PasswordBearer #This can be used later for depency injection
    Step2) Intialize it and point to a url
        oauth2_bearer = OAuth2PasswordBearer(tokenUrl="auth/token")
    Step3) Create a function to verify credentials and return username and user_id
        Code:
            async def get_current_user(token: Annotated[str, Depends(oauth2_bearer)]):
            try:
                payload = jwt.decode(token, SECRET_KEY, algorithms=ALGORITHM)
                username: str = payload.get("sub")
                user_id: int = payload.get("user_id")
                if username is None or user_id is None:
                    raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="couldnot validate details")
                return {"username": username, "user_id": user_id}
            except Exception as e:
                raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="couldnot validate details")
    
## Code cleanups
    if change route to this
        router = APIRouter(
            prefix="/auth",
            tags=["auth"]
        )
        this will add auth as the prefix path of all the routers that are created in this file
        tag acts as a title in the swagger

## Alembic
    What is Alembic and why do we need this?
        Lightweight database migration tool for when using SQLAlchemy
        Migration tools allows us to plan, transfer and upgrade within our database.
        Alembic allows us to change SQLAlchemy database table after it has been created.
        Currently SQLAlchemy will create table for us, but we cannot enhance it.
    Some more details:
    Alembic provides the create and invocation change management scripts
    this allows you to be able to create migration environment and able to change data how you like.
    It is a powerful migration tool that allows us to modify database schema.
    As the application evolves our database will need to evolve as well
    Helps us to modifying database to keep up with the rapid development growth.
    Step1) install alembic
        pip install alembic
    Step2) Alembic Commands
        alembic init<folder name> - initializes new, generic environment
        alembic revision -m <message> - Creates a new revision of the environment
        alembic upgrade<revision #> - Run upgrade migration to database
        alembic downgrade -1 - Run downgrade migration to databse
### How does alembic work?
    After we initilize our application with alembic, two new things will be appered in our directory.
    alembic.ini
    alembic directory
    These are automatically created so we can upgrade, downgrade and keep data integrity of the application
    alembic.ini
        file that alembic looks when invoked
        contains a bunch of configuration information for alembic that we are able to change to match our project.
    alembic Directory
        has all environment properties for alembic.
        Holds all the revisions of the application.
        where we can call the migrations for upgrading
        where we can call the migrations for downgrading.
    Run the init command
        alembic init alembic(foldername)
### Alembic Revision ?
    Alembic revision is how we create a new alembic file where we can add some type of database upgrade.
    when we run:
        alembic revision -m "create phone_number col on user table"
        create a new file where we can write the upgrade code.
        each new revision will have revised id.
### Alembic upgrade and downgrade?
    Alembic upgrade
        Alembic upgrade is how we actually run the migration
        def upgrade() -> None:
            op.add_column('users', sa.Column('phone_number', sa.String(), nullable=True))
        Enhances our database to now have a new column within our user tables called phone_number
        Previous data within our database doesnt change.
        Run upgrade command:
            alembic upgrade <revision id>
        This will successfully implement the change within the upgrade functionality.
    Alembic downgrade
        In the same file we can also write downgrade functionality.
        def downgrade() -> None:
            op.drop_column('users', 'phone_number')
        Reverts our database to remove last migration change.
        previous data within database will not change unless it was on the colum 'phone_number' because we deleted it.
        Run downgrade migration command:
            alembic downgrade -1
### Excute downgrade and upgrade in the code
    step1) alembic revision -m "Create phone_number column on user table"
    Step2) open the generated file under alembic/versions/<random_id>create_phone_number_colum_on_user_table.pu
    Step3) There we can find the generated code for upgrade and downgrade functions with pass statements.
    Step4) In the upgrade function add this
        op.add_column('users', sa.Column('phone_number', sa.String(), nullable=True))
    Step5) Copy the revision id from the above file
    Step6) command to upgrade
        alembic upgrade 6c14b2d5f700
        After execution it will show like this
            Running upgrade  -> 6c14b2d5f700, create phonenumber col on user table
    
    

    
    
    
    
        
    

                      
          
    



    
        

    
            

