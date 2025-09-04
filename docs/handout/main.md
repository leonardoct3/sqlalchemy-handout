# Handout: SQLAlchemy com Arquitetura Limpa

???+ info inline end "Informações"

    **Objetivo**: Aprender SQLAlchemy aplicando princípios de Clean Architecture, elaborando a estrutura base de uma API 
    **Pré-requisitos**: Python básico, alguns conceitos de POO (Programação Orientada à Objetos) e SQL

## Introdução

Neste handout, você aprenderá a criar um projeto Python utilizando **SQLAlchemy** como ORM (Object-Relational Mapper) seguindo os princípios da **Arquitetura Limpa** (Clean Architecture). Ao final, você terá criado um sistema de biblioteca que gerencia autores e livros, preparando o terreno para futuramente criar uma API REST.

### O que é SQLAlchemy?

SQLAlchemy é um toolkit SQL e ORM para Python que permite:

- Mapear classes Python para tabelas de banco de dados
- Executar consultas SQL de forma mais pythônica
- Gerenciar relacionamentos entre entidades
- Abstrair detalhes específicos do banco de dados

### O que é Clean Architecture?

A Arquitetura Limpa é um padrão arquitetural que visa separar as preocupações do software em camadas bem definidas, tornando o código:

- **Testável**: Cada camada pode ser testada independentemente
- **Flexível**: Mudanças em uma camada não afetam as outras
- **Independente**: A lógica de negócio não depende de frameworks ou bancos específicos

## Estrutura do Projeto

Vamos criar um projeto com a seguinte estrutura:

```
sqlalchemy-lesson/
├── app/
│   ├── database/          # Configuração do banco de dados
│   ├── entities/          # Entidades de domínio (Pydantic)
│   ├── models/            # Modelos SQLAlchemy (ORM)
│   └── repositories/      # Padrão Repository
├── examples/              # Exemplos de uso
├── alembic/              # Migrações de banco
├── .env                  # Variáveis de ambiente
├── requirements.txt      # Dependências
└── alembic.ini          # Configuração do Alembic
```

### Explicação das Camadas

#### 🗄️ Database
Contém a configuração da conexão com o banco de dados, engine e sessões.

#### 🏗️ Entities (Entidades)
Classes Python que representam os conceitos do domínio de negócio, independentes de qualquer tecnologia de persistência.

#### 📊 Models
Classes SQLAlchemy que mapeiam as entidades para tabelas do banco de dados.

#### 🔄 Repositories
Implementam o padrão Repository, abstraindo o acesso aos dados e fornecendo uma interface limpa para operações CRUD.

---

## Parte 1: Configuração Inicial do Projeto

### 1.1 Criando o Repositório

Primeiro, vamos criar o repositório e a estrutura básica:

```bash
mkdir sqlalchemy-lesson
cd sqlalchemy-lesson
```

### 1.2 Ambiente Virtual e Dependências

```bash
# Criar ambiente virtual
python -m venv venv

# Ativar ambiente virtual (Windows)
venv\Scripts\activate

# Ativar ambiente virtual (Linux/macOS)
source venv/bin/activate
```

Agora vamos criar o arquivo `requirements.txt`:

```txt title="requirements.txt"
SQLAlchemy==2.0.30
psycopg2-binary==2.9.9
python-dotenv==1.0.1
pydantic[email]==2.7.1
alembic==1.13.1
```

```bash
# Instalar dependências
pip install -r requirements.txt
```

### 1.3 Estrutura de Pastas

Crie a estrutura de pastas do projeto:

```bash
mkdir -p app/database app/entities app/models app/repositories examples alembic
```

!!! tip "Organização"
    Uma boa organização de pastas é fundamental para manter o projeto escalável e fácil de manter.

---

## Parte 2: Configuração do Banco de Dados

### 2.1 Variáveis de Ambiente

Crie o arquivo `.env` na raiz do projeto:

```env title=".env"
DATABASE_URL=sqlite:///./biblioteca.db
```

!!! note "Bancos Suportados"
    Para PostgreSQL: `postgresql://user:password@localhost/dbname`  
    Para MySQL: `mysql://user:password@localhost/dbname`

### 2.2 Configuração da Database

Crie o arquivo `app/database/database.py`:

```python title="app/database/database.py"
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base
from dotenv import load_dotenv
import os

load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./biblioteca.db")

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()
```

**Explicação do código:**

- `create_engine()`: Cria a conexão com o banco
- `sessionmaker()`: Factory para criar sessões de banco
- `declarative_base()`: Classe base para nossos modelos ORM

### 2.3 Arquivo `__init__.py`

Crie `app/database/__init__.py` vazio (podemos removê-lo posteriormente se necessário).

---

## Parte 3: Entidades de Domínio

As entidades representam os conceitos do nosso domínio de negócio. Vamos usar **Pydantic** para validação e serialização.

### 3.1 Entidade Author

Crie `app/entities/author.py`:

```python title="app/entities/author.py"
from pydantic import BaseModel, EmailStr, Field
from typing import List, Optional

class Author(BaseModel):
    """
    Entidade Author usando Pydantic
    Representa um autor no domínio da aplicação
    """
    id: Optional[int] = None
    name: str = Field(..., min_length=1, max_length=255)
    email: EmailStr
    books: List['Book'] = Field(default_factory=list)

    class Config:
        # Permite conversão de objetos SQLAlchemy
        from_attributes = True

    def __str__(self):
        return f"Author: {self.name} ({self.email})"

# Resolve forward references
from .book import Book
Author.model_rebuild()
```

!!! question "Por que Pydantic?"
    - Validação automática de dados
    - Serialização/deserialização JSON
    - Documentação automática de APIs
    - Type hints nativos

### 3.2 **Prática 1**: Entidade Book

Agora é sua vez! Crie a entidade `Book` no arquivo `app/entities/book.py`.

**Requisitos:**
- Campo `id` opcional
- Campo `title` obrigatório (1-500 caracteres)
- Campo `isbn` opcional (10-17 caracteres)
- Lista de `authors`

??? success "Solução"
    ```python title="app/entities/book.py"
    from pydantic import BaseModel, Field
    from typing import List, Optional

    class Book(BaseModel):
        """
        Entidade Book usando Pydantic
        Representa um livro no domínio da aplicação
        """
        id: Optional[int] = None
        title: str = Field(..., min_length=1, max_length=500)
        isbn: Optional[str] = Field(None, min_length=10, max_length=17)
        authors: List['Author'] = Field(default_factory=list)

        class Config:
            # Permite conversão de objetos SQLAlchemy
            from_attributes = True

        def __str__(self):
            return f"Book: {self.title} {self.isbn}"

    # Resolve forward references
    from .author import Author
    Book.model_rebuild()
    ```

---

## Parte 4: Modelos SQLAlchemy

Os modelos SQLAlchemy fazem o mapeamento objeto-relacional (ORM) entre nossas classes Python e as tabelas do banco de dados.

### 4.1 Modelo Author

Crie `app/models/author_model.py`:

```python title="app/models/author_model.py"
from sqlalchemy import Column, Integer, String, Table, ForeignKey
from sqlalchemy.orm import relationship
from app.database.database import Base

# Tabela de associação many-to-many entre autores e livros
author_book_association = Table(
    'author_book_association',
    Base.metadata,
    Column('author_id', Integer, ForeignKey('authors.id'), primary_key=True),
    Column('book_id', Integer, ForeignKey('books.id'), primary_key=True)
)

class AuthorModel(Base):
    __tablename__ = "authors"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    email = Column(String, unique=True, index=True)

    # Relacionamento many-to-many com livros
    books = relationship("BookModel", secondary=author_book_association, back_populates="authors")

    def __repr__(self):
        return f"<AuthorModel(id={self.id}, name='{self.name}', email='{self.email}')>"
```

**Conceitos importantes:**

- `__tablename__`: Nome da tabela no banco de dados
- `Column()`: Define uma coluna da tabela
- `relationship()`: Define relacionamentos entre entidades
- `Table()`: Tabela de associação para relacionamentos many-to-many

### 4.2 **Prática 2**: Modelo Book

Crie o modelo `BookModel` no arquivo `app/models/book_model.py`.

**Dicas:**
- Herdar de `Base`
- Definir `__tablename__ = "books"`
- Campos: `id`, `title`, `isbn`
- Relacionamento many-to-many com `AuthorModel`

??? success "Solução"
    ```python title="app/models/book_model.py"
    from sqlalchemy import Column, Integer, String
    from sqlalchemy.orm import relationship
    from app.database.database import Base

    class BookModel(Base):
        __tablename__ = "books"

        id = Column(Integer, primary_key=True, index=True)
        title = Column(String, index=True)
        isbn = Column(String, unique=True, index=True)
        
        # Relacionamento many-to-many com autores
        authors = relationship("AuthorModel", secondary="author_book_association", back_populates="books")

        def __repr__(self):
            return f"<BookModel(id={self.id}, title='{self.title}', isbn='{self.isbn}')>"
    ```

### 4.3 Arquivo de Inicialização dos Models

Crie `app/models/__init__.py`:

```python title="app/models/__init__.py"
from .author_model import AuthorModel, author_book_association
from .book_model import BookModel

__all__ = ["AuthorModel", "BookModel", "author_book_association"]
```

---

## Parte 5: Padrão Repository

O padrão Repository abstrai o acesso aos dados, fornecendo uma interface limpa para operações CRUD.

### 5.1 Base Repository

Primeiro, vamos criar um repositório base com operações comuns:

```python title="app/repositories/base_repository.py"
from sqlalchemy.orm import Session
from typing import TypeVar, Type, Generic

T = TypeVar('T')

class BaseRepository(Generic[T]):
    def __init__(self, session: Session, model: Type[T]):
        self.session = session
        self.model = model

    def get_all(self) -> list[T]:
        return self.session.query(self.model).all()

    def get_by_id(self, id: int) -> T | None:
        return self.session.query(self.model).filter(self.model.id == id).first()

    def add(self, entity: T) -> T:
        self.session.add(entity)
        self.session.commit()
        self.session.refresh(entity)
        return entity

    def update(self, entity: T) -> T:
        self.session.merge(entity)
        self.session.commit()
        self.session.refresh(entity)
        return entity

    def delete(self, id: int):
        entity = self.get_by_id(id)
        if entity:
            self.session.delete(entity)
            self.session.commit()
```

!!! info "Generics em Python"
    Usamos `Generic[T]` para criar um repositório reutilizável para qualquer tipo de modelo.

### 5.2 Repository do Author

Crie `app/repositories/author_repository.py`:

```python title="app/repositories/author_repository.py"
from sqlalchemy.orm import Session
from app.models.author_model import AuthorModel
from app.repositories.base_repository import BaseRepository

class AuthorRepository(BaseRepository[AuthorModel]):
    def __init__(self, session: Session):
        super().__init__(session, AuthorModel)

    def get_by_email(self, email: str) -> AuthorModel | None:
        """Busca um autor pelo email"""
        return self.session.query(AuthorModel).filter(AuthorModel.email == email).first()

    def get_by_name(self, name: str) -> list[AuthorModel]:
        """Busca autores pelo nome (busca parcial)"""
        return self.session.query(AuthorModel).filter(AuthorModel.name.ilike(f"%{name}%")).all()

    def get_authors_with_books(self) -> list[AuthorModel]:
        """Retorna autores que têm livros"""
        return self.session.query(AuthorModel).join(AuthorModel.books).distinct().all()
```

### 5.3 **Prática 3**: Repository do Book

Crie o `BookRepository` no arquivo `app/repositories/book_repository.py`.

**Métodos adicionais necessários:**
- `get_by_title(title: str)`: Busca por título (parcial)
- `get_by_isbn(isbn: str)`: Busca por ISBN exato
- `get_books_with_authors()`: Livros com autores carregados

??? success "Solução"
    ```python title="app/repositories/book_repository.py"
    from sqlalchemy.orm import Session
    from app.models.book_model import BookModel
    from app.repositories.base_repository import BaseRepository

    class BookRepository(BaseRepository[BookModel]):
        def __init__(self, session: Session):
            super().__init__(session, BookModel)

        def get_by_title(self, title: str) -> list[BookModel]:
            """Busca livros pelo título (busca parcial)"""
            return self.session.query(BookModel).filter(BookModel.title.ilike(f"%{title}%")).all()

        def get_by_isbn(self, isbn: str) -> BookModel | None:
            """Busca livro pelo ISBN"""
            return self.session.query(BookModel).filter(BookModel.isbn == isbn).first()

        def get_books_with_authors(self) -> list[BookModel]:
            """Retorna livros com informações dos autores carregadas"""
            return self.session.query(BookModel).join(BookModel.authors).all()
    ```

---

## Parte 6: Migrações com Alembic

O Alembic é uma ferramenta de migração de banco de dados para SQLAlchemy que permite:

- Versionamento do schema do banco
- Migrações incrementais
- Rollback de mudanças

### 6.1 Inicializando o Alembic

```bash
alembic init alembic
```

### 6.2 Configurando o Alembic

Edite o arquivo `alembic/env.py` para conectar com nossos modelos:

```python title="alembic/env.py" hl_lines="24-27"
from logging.config import fileConfig

from sqlalchemy import engine_from_config
from sqlalchemy import pool

from alembic import context

# this is the Alembic Config object, which provides
# access to the values within the .ini file in use.
config = context.config

# Interpret the config file for Python logging.
# This line sets up loggers basically.
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

# add your model's MetaData object here
# for 'autogenerate' support
# from myapp import mymodel
# target_metadata = mymodel.Base.metadata
from app.database.database import Base
# Import all models to ensure they are registered with the Base metadata
from app.models import AuthorModel, BookModel, author_book_association
target_metadata = Base.metadata

# ... resto do arquivo permanece igual
```

### 6.3 Configurando a URL do Banco

Edite `alembic.ini` para usar a URL do arquivo `.env`:

```ini title="alembic.ini"
# Comentar a linha sqlalchemy.url e adicionar:
# sqlalchemy.url = driver://user:pass@localhost/dbname
sqlalchemy.url = sqlite:///./biblioteca.db
```

### 6.4 Criando a Primeira Migração

```bash
alembic revision --autogenerate -m "Create authors, books and association tables"
```

```bash
alembic upgrade head
```

!!! success "Banco Criado!"
    As tabelas `authors`, `books` e `author_book_association` foram criadas no banco de dados.

---

## Parte 7: Exemplos Práticos

Agora vamos criar exemplos práticos para testar nossa implementação.

### 7.1 Exemplo Básico de CRUD

Crie `examples/basic_example.py`:

```python title="examples/basic_example.py"
from app.database.database import SessionLocal, Base, engine
from app.models.author_model import AuthorModel
from app.models.book_model import BookModel
from app.repositories.author_repository import AuthorRepository
from app.repositories.book_repository import BookRepository

# Criar as tabelas (caso não tenham sido criadas pelo Alembic)
Base.metadata.create_all(bind=engine)

def main():
    # Criar uma sessão
    session = SessionLocal()
    
    try:
        # Instanciar repositórios
        author_repo = AuthorRepository(session)
        book_repo = BookRepository(session)
        
        # Criar um autor
        author = AuthorModel(
            name="Machado de Assis",
            email="machado@email.com"
        )
        author = author_repo.add(author)
        print(f"Autor criado: {author}")
        
        # Criar um livro
        book = BookModel(
            title="Dom Casmurro",
            isbn="978-8525406958"
        )
        book = book_repo.add(book)
        print(f"Livro criado: {book}")
        
        # Associar autor ao livro
        book.authors.append(author)
        session.commit()
        
        # Buscar autor por email
        found_author = author_repo.get_by_email("machado@email.com")
        print(f"Autor encontrado: {found_author}")
        print(f"Livros do autor: {[book.title for book in found_author.books]}")
        
        # Buscar livro por título
        found_books = book_repo.get_by_title("Dom")
        print(f"Livros encontrados: {[book.title for book in found_books]}")
        
    finally:
        session.close()

if __name__ == "__main__":
    main()
```

### 7.2 **Prática 4**: Exemplo de Uso Avançado

Crie um exemplo em `examples/advanced_example.py` que:

1. Cria múltiplos autores e livros
2. Estabelece relacionamentos many-to-many
3. Demonstra buscas complexas
4. Mostra operações de update e delete

??? success "Solução"
    ```python title="examples/advanced_example.py"
    from app.database.database import SessionLocal, Base, engine
    from app.models.author_model import AuthorModel
    from app.models.book_model import BookModel
    from app.repositories.author_repository import AuthorRepository
    from app.repositories.book_repository import BookRepository

    def main():
        session = SessionLocal()
        
        try:
            author_repo = AuthorRepository(session)
            book_repo = BookRepository(session)
            
            # Limpar dados existentes para o exemplo
            session.query(AuthorModel).delete()
            session.query(BookModel).delete()
            session.commit()
            
            # Criar autores
            authors_data = [
                {"name": "Clarice Lispector", "email": "clarice@email.com"},
                {"name": "Jorge Amado", "email": "jorge@email.com"},
                {"name": "Paulo Coelho", "email": "paulo@email.com"}
            ]
            
            authors = []
            for data in authors_data:
                author = AuthorModel(**data)
                author = author_repo.add(author)
                authors.append(author)
                print(f"Autor criado: {author.name}")
            
            # Criar livros
            books_data = [
                {"title": "A Hora da Estrela", "isbn": "978-8520925829"},
                {"title": "Gabriela, Cravo e Canela", "isbn": "978-8535902976"},
                {"title": "O Alquimista", "isbn": "978-8595081413"},
                {"title": "Água Viva", "isbn": "978-8520925836"}
            ]
            
            books = []
            for data in books_data:
                book = BookModel(**data)
                book = book_repo.add(book)
                books.append(book)
                print(f"Livro criado: {book.title}")
            
            # Estabelecer relacionamentos
            # Clarice Lispector - A Hora da Estrela e Água Viva
            books[0].authors.append(authors[0])  # A Hora da Estrela
            books[3].authors.append(authors[0])  # Água Viva
            
            # Jorge Amado - Gabriela, Cravo e Canela
            books[1].authors.append(authors[1])
            
            # Paulo Coelho - O Alquimista
            books[2].authors.append(authors[2])
            
            session.commit()
            
            # Demonstrar buscas
            print("\n=== BUSCAS ===")
            
            # Autores com livros
            authors_with_books = author_repo.get_authors_with_books()
            print(f"Autores com livros: {len(authors_with_books)}")
            
            # Busca por nome parcial
            clarice_authors = author_repo.get_by_name("Clarice")
            print(f"Autores com 'Clarice': {[a.name for a in clarice_authors]}")
            
            # Livros de um autor específico
            clarice = author_repo.get_by_email("clarice@email.com")
            print(f"Livros da Clarice: {[book.title for book in clarice.books]}")
            
            # Update - Alterar email de um autor
            paulo = author_repo.get_by_email("paulo@email.com")
            paulo.email = "paulo.coelho@email.com"
            updated_paulo = author_repo.update(paulo)
            print(f"Email atualizado: {updated_paulo.email}")
            
            # Delete - Remover um livro
            book_to_delete = book_repo.get_by_isbn("978-8520925836")
            if book_to_delete:
                book_repo.delete(book_to_delete.id)
                print(f"Livro '{book_to_delete.title}' removido")
            
        finally:
            session.close()

    if __name__ == "__main__":
        main()
    ```

### 7.3 Executando os Exemplos

```bash
# Executar exemplo básico
python examples/basic_example.py

# Executar exemplo avançado
python examples/advanced_example.py
```

---

## Parte 8: Princípios SOLID Aplicados

Vamos analisar como nosso projeto aplicou os princípios SOLID:

### 8.1 Single Responsibility Principle (SRP)

Cada classe tem uma única responsabilidade:

- **Entities**: Representam conceitos de domínio
- **Models**: Mapeamento ORM
- **Repositories**: Acesso a dados

### 8.2 Open/Closed Principle (OCP)

O `BaseRepository` está **aberto para extensão** e **fechado para modificação**:

```python
# Extensão sem modificar a classe base
class AuthorRepository(BaseRepository[AuthorModel]):
    def get_by_email(self, email: str):
        # Método específico para Author
        pass
```

### 8.3 Liskov Substitution Principle (LSP)

Qualquer `Repository` pode ser substituído por sua classe base:

```python
def process_repository(repo: BaseRepository):
    # Funciona com qualquer implementação de BaseRepository
    entities = repo.get_all()
    return entities
```

### 8.4 Interface Segregation Principle (ISP)

Cada repository expõe apenas os métodos relevantes para sua entidade.

### 8.5 Dependency Inversion Principle (DIP)

Os repositories dependem de abstrações (Session) e não de implementações concretas.

---

## Parte 9: Próximos Passos e Integração com APIs

### 9.1 Preparação para APIs

Nossa arquitetura está pronta para integração com **FastAPI**:

```python title="Exemplo de integração futura"
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session
from app.database.database import SessionLocal
from app.repositories.author_repository import AuthorRepository
from app.entities.author import Author

app = FastAPI()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.post("/authors/", response_model=Author)
def create_author(author: Author, db: Session = Depends(get_db)):
    repo = AuthorRepository(db)
    # Lógica de criação
    return created_author
```

### 9.2 Melhorias Futuras

- **Validações de negócio** nas entities
- **Use Cases** para lógica complexa
- **DTOs** para separar entradas/saídas da API
- **Logging** e monitoramento

---

## Resumo

Neste handout, os conteúdos que passados foram:

✅ **Configurar um projeto SQLAlchemy** com arquitetura limpa  
✅ **Criar entidades de domínio** com Pydantic  
✅ **Implementar modelos ORM** com relacionamentos  
✅ **Aplicar o padrão Repository** para acesso a dados  
✅ **Gerenciar migrações** com Alembic  
✅ **Aplicar princípios SOLID** na prática  
✅ **Preparar a base** para futuras APIs REST  

### Conceitos-Chave

| Conceito | Descrição |
|----------|-----------|
| **ORM** | Mapeamento objeto-relacional |
| **Entities** | Representação do domínio de negócio |
| **Models** | Mapeamento para tabelas do banco |
| **Repository** | Abstração de acesso a dados |
| **Clean Architecture** | Separação de responsabilidades |
| **Migrations** | Versionamento do schema |

---

## Limpeza e Organização do Projeto

### Arquivos Desnecessários

Durante o desenvolvimento, alguns arquivos `__init__.py` podem não ser necessários. No nosso projeto:

**🗂️ Arquivos `__init__.py` NECESSÁRIOS:**
- `app/database/__init__.py` - Exporta configurações do banco
- `app/models/__init__.py` - Exporta modelos para importação fácil

**🗂️ Arquivos `__init__.py` DESNECESSÁRIOS:**
- `app/entities/` - Não precisa, usamos imports diretos
- `app/repositories/` - Não precisa, usamos imports diretos

### Estrutura Final Limpa

```
sqlalchemy-lesson/
├── app/
│   ├── database/
│   │   ├── database.py    ✅
│   │   └── __init__.py    ✅ (útil para imports)
│   ├── entities/
│   │   ├── author.py      ✅
│   │   └── book.py        ✅
│   ├── models/
│   │   ├── author_model.py ✅
│   │   ├── book_model.py   ✅
│   │   └── __init__.py     ✅ (útil para imports)
│   └── repositories/
│       ├── base_repository.py     ✅
│       ├── author_repository.py   ✅
│       └── book_repository.py     ✅
├── examples/
│   └── complete_example.py       ✅
├── alembic/                      ✅
├── .env                          ✅
├── .env.example                  ✅
├── requirements.txt              ✅
├── alembic.ini                   ✅
└── README.md                     ✅
```

!!! tip "Dica de Organização"
    Mantenha apenas os arquivos necessários. Uma estrutura limpa facilita a manutenção e compreensão do projeto.

---

## 🎯 Desafio

**Implemente um sistema de empréstimos de livros:**

1. Crie uma entidade `Loan` (empréstimo)
2. Relacione com `Book` e adicione um campo `User`
3. Implemente `LoanRepository` com métodos específicos
4. Crie migração para a nova tabela
5. Desenvolva exemplo prático de uso

### Campos sugeridos para Loan:
- `id`, `book_id`, `user_name`, `loan_date`, `return_date`, `returned`

!!! tip "Dica"
    Use tudo que aprendeu: entidades Pydantic, modelos SQLAlchemy, repository pattern e migrações! Se baseie também no que já está feito!

---

## Referências

- [SQLAlchemy Documentation](https://docs.sqlalchemy.org/en/20/)
- [Pydantic Documentation](https://docs.pydantic.dev/)
- [Alembic Documentation](https://alembic.sqlalchemy.org/)
- [Clean Architecture by Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)