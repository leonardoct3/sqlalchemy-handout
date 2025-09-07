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

```powershell
# PowerShell/CMD
mkdir app\database, app\entities, app\models, app\repositories, examples, alembic
```

ou

```bash
# Linux/macOS
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

### O que são Entidades?

As **entidades** representam os conceitos centrais do nosso domínio de negócio - são as "coisas" importantes que nosso sistema gerencia. No caso de uma biblioteca, nossos conceitos principais são **Autores** e **Livros**.

#### **Objetivo das Entidades:**
- Representar os **conceitos de negócio** de forma pura
- Conter as **regras de validação** dos dados
- Ser **independentes** de tecnologias (banco de dados, frameworks)
- Facilitar a **comunicação** entre diferentes camadas da aplicação

### Por que Pydantic?

**Pydantic** é uma biblioteca Python para validação de dados usando type hints. Escolhemos ela porque:

- **Validação automática**: Garante que os dados estão corretos
- **Serialização JSON**: Converte objetos Python ↔ JSON facilmente
- **Type hints nativos**: Usa anotações de tipo do Python
- **Integração com APIs**: Base perfeita para FastAPI
- **Documentação automática**: Gera documentação dos campos

### 3.1 Entidade Author (Primeira Versão)

Vamos começar com uma versão **simplificada** da entidade Author:

```python title="app/entities/author.py"
from pydantic import BaseModel, EmailStr, Field
from typing import Optional

class Author(BaseModel):
    """
    Entidade Author - Representa um autor de livros
    
    Esta é nossa representação 'pura' de um autor,
    independente de como ele é armazenado no banco
    """
    id: Optional[int] = None  # Opcional pois é gerado pelo banco
    name: str = Field(..., min_length=1, max_length=255)
    email: EmailStr

    class Config:
        # Permite converter objetos SQLAlchemy para Pydantic
        from_attributes = True

    def __str__(self):
        return f"Author: {self.name} ({self.email})"
```

??? info "📖 Explicação linha por linha:"


    ### Explicação Detalhada do Código Author

    ```python
    # Importações necessárias
    from pydantic import BaseModel, EmailStr, Field
    from typing import Optional
    
    class Author(BaseModel):
        id: Optional[int] = None
        name: str = Field(..., min_length=1, max_length=255)
        email: EmailStr
        class Config:
            from_attributes = True
        def __str__(self):
            return f"Author: {self.name} ({self.email})"
    ```

    - **Importações**:
        - `BaseModel`: Classe base do Pydantic, traz validação automática e serialização.
        - `EmailStr`: Tipo especial para validar e garantir formato de email.
        - `Field`: Permite definir regras extras para os campos (obrigatoriedade, tamanho, etc).
        - `Optional`: Indica que o campo pode ser `None` (opcional).

    - **Classe Author(BaseModel)**:
        - Herda de `BaseModel`, tornando a classe uma entidade Pydantic.

    - **Atributos**:
        - `id: Optional[int] = None`: Identificador do autor, opcional pois geralmente é gerado pelo banco.
        - `name: str = Field(..., min_length=1, max_length=255)`: Nome obrigatório, entre 1 e 255 caracteres.
        - `email: EmailStr`: Email obrigatório, validado automaticamente.

    - **Config interna**:
        - `from_attributes = True`: Permite converter objetos SQLAlchemy para Pydantic facilmente.

    - **Método especial**:
        - `__str__`: Facilita o print e debug, mostrando nome e email do autor.

    > **Resumo:**
    > Essa estrutura garante que todo autor criado seja válido, com nome preenchido e email correto, além de facilitar integração com banco e APIs.

---

### Facilitando a Prática: Entidade Book

Para criar a entidade Book, siga o mesmo padrão do Author. Veja o passo a passo:

1. **Crie o arquivo** `app/entities/book.py`
2. **Implemente o código:**

```python
from pydantic import BaseModel, Field
from typing import Optional

class Book(BaseModel):
    id: Optional[int] = None  # Opcional, gerado pelo banco
    title: str = Field(..., min_length=1, max_length=500)  # Título obrigatório
    isbn: Optional[str] = Field(None, min_length=10, max_length=17)  # ISBN opcional

    class Config:
        from_attributes = True

    def __str__(self):
        return f"Book: {self.title}"
```

**Dicas para a prática:**
- Siga o padrão de validação do Author
- Use `Field` para definir limites de tamanho
- O campo `isbn` pode ser omitido ao criar um livro
- O método `__str__` facilita prints e testes

> **Teste rápido:**
> ```python
> book = Book(title="Dom Casmurro", isbn="978-8525406958")
> print(book)
> ```

### 3.2 Testando a Entidade Author

Vamos criar um exemplo simples para entender como funciona. Crie o seguinte arquivo na pasta */app* do seu projeto e execute-o:

```python title="app/author_example.py"
from entities.author import Author

# Criando um autor válido
author = Author(
    name="Machado de Assis",
    email="machado@email.com"
)

print("============= Autor Válido =============")
print(author)  # Author: Machado de Assis (machado@email.com)
print()
print(author.model_dump())  # {'id': None, 'name': 'Machado de Assis', 'email': 'machado@email.com'}
print()
# Tentando criar autor inválido
try:
    print("============= Tentando criar Autor Inválido =============")
    invalid_author = Author(
        name="",  # ← Nome vazio! Vai dar erro
        email="email-inválido"  # ← Email inválido! Vai dar erro
    )
except ValueError as e:
    print(f"Erro de validação: {e}")
```

### 3.3 **Prática 1**: Entidade Book (Simples)

Agora é sua vez! Crie a entidade `Book` no arquivo `app/entities/book.py`.

**Requisitos:**
- Campo `id` opcional
- Campo `title` obrigatório (1-500 caracteres)
- Campo `isbn` opcional (10-17 caracteres quando fornecido)

!!! tip "Dica"
    Por enquanto, **não** inclua a lista de autores. Vamos manter simples e adicionar relacionamentos depois!

??? success "Solução"
    ```python title="app/entities/book.py"
    from pydantic import BaseModel, Field
    from typing import Optional

    class Book(BaseModel):
        """
        Entidade Book - Representa um livro na biblioteca
        
        Mantida simples por enquanto, sem relacionamentos
        """
        id: Optional[int] = None
        title: str = Field(..., min_length=1, max_length=500)
        isbn: Optional[str] = Field(None, min_length=10, max_length=17)

        class Config:
            from_attributes = True

        def __str__(self):
            return f"Book: {self.title}"
    ```

### 3.4 Versão Completa com Relacionamentos

Após entender o conceito básico, podemos criar versões mais complexas com relacionamentos:

??? info "Versão Avançada com Relacionamentos"
    ```python title="app/entities/author.py - Versão Completa"
    from pydantic import BaseModel, EmailStr, Field
    from typing import List, Optional

    class Author(BaseModel):
        """
        Entidade Author - Versão completa com relacionamentos
        """
        id: Optional[int] = None
        name: str = Field(..., min_length=1, max_length=255)
        email: EmailStr
        books: List['Book'] = Field(default_factory=list)

        class Config:
            from_attributes = True

        def __str__(self):
            return f"Author: {self.name} ({self.email})"

    # Resolve forward references (necessário para relacionamentos)
    from .book import Book
    Author.model_rebuild()
    ```

    ```python title="app/entities/book.py - Versão Completa"
    from pydantic import BaseModel, Field
    from typing import List, Optional

    class Book(BaseModel):
        """
        Entidade Book - Versão completa com relacionamentos
        """
        id: Optional[int] = None
        title: str = Field(..., min_length=1, max_length=500)
        isbn: Optional[str] = Field(None, min_length=10, max_length=17)
        authors: List['Author'] = Field(default_factory=list)

        class Config:
            from_attributes = True

        def __str__(self):
            return f"Book: {self.title}"

    # Resolve forward references
    from .author import Author
    Book.model_rebuild()
    ```

### 3.5 Conceitos Importantes

#### Forward References
```python
books: List['Book'] = Field(default_factory=list)
#           ↑ String quote porque Book ainda não foi definido
```

#### model_rebuild()
```python
Author.model_rebuild()
# ↑ Necessário para resolver as referências circulares
```

#### default_factory
```python
books: List['Book'] = Field(default_factory=list)
#                           ↑ Cria uma nova lista para cada instância
```

### 3.6 Entidades vs Models: Qual a Diferença?

É comum haver confusão entre **Entidades** e **Models**. Vamos esclarecer:

| Aspecto | **Entidades** (Pydantic) | **Models** (SQLAlchemy) |
|---------|---------------------------|--------------------------|
| **Propósito** | Regras de negócio | Persistência no banco |
| **Dependência** | Independente de tecnologia | Dependente do SQLAlchemy |
| **Foco** | Validação e comportamento | Mapeamento de tabelas |
| **Localização** | Camada de domínio | Camada de infraestrutura |

#### Fluxo na Arquitetura Limpa:

```
API Request → Entidade (validação) → Model (persistência) → Database
               ↑                      ↑
         Regras de negócio      Mapeamento ORM
```


### 3.7 Testando Nossas Entidades

Vamos criar um exemplo prático para testar as entidades:

```python title="app/test_entities.py - Exemplo de teste"
from entities.author import Author
from entities.book import Book

def test_entities():
    # Testando validações
    print("=== Testando Entidades ===")
    
    # Autor válido
    author = Author(
        name="Clarice Lispector",
        email="clarice@email.com"
    )
    print(f"✅ Autor criado: {author}")
    
    # Livro válido
    book = Book(
        title="A Hora da Estrela",
        isbn="978-8520925829"
    )
    print(f"✅ Livro criado: {book}")
    
    # Testando validações que devem falhar
    try:
        # Nome vazio
        invalid_author = Author(name="", email="test@email.com")
    except ValueError as e:
        print(f"❌ Validação funcionou - Nome vazio: {e}")
    
    try:
        # Email inválido
        invalid_author = Author(name="Test", email="email-inválido")
    except ValueError as e:
        print(f"❌ Validação funcionou - Email inválido: {e}")
    
    try:
        # ISBN muito curto
        invalid_book = Book(title="Test", isbn="123")
    except ValueError as e:
        print(f"❌ Validação funcionou - ISBN inválido: {e}")

if __name__ == "__main__":
    test_entities()
```

!!! tip "Execute o Teste"
    Crie esse arquivo na raiz do projeto e execute com `python test_entities.py` para ver as validações funcionando!

---

## Parte 4: Modelos SQLAlchemy

### O que são Models?

Os **models** são as classes que representam as tabelas do banco de dados. Eles fazem o "mapeamento objeto-relacional" (ORM), ou seja, conectam o mundo Python ao mundo SQL.

#### **Objetivo dos Models:**
- Definir como os dados serão armazenados no banco
- Mapear atributos Python para colunas SQL
- Gerenciar relacionamentos entre tabelas
- Permitir operações CRUD usando objetos Python

### Por que SQLAlchemy?

#### **SQLAlchemy** é a principal biblioteca ORM do Python. Ela permite:
- Criar tabelas e relacionamentos usando classes Python
- Executar queries SQL de forma pythônica
- Suporte a vários bancos (SQLite, PostgreSQL, MySQL...)
- Gerenciar migrações e versionamento do schema

### 4.1 Model Author: Mapeando a Tabela de Autores

Vamos criar o model que representa a tabela de autores. Ele será responsável por:
- Definir os campos da tabela
- Gerenciar o relacionamento com livros

```python title="app/models/author_model.py"
from sqlalchemy import Column, Integer, String, Table, ForeignKey
from sqlalchemy.orm import relationship
from database.database import Base

# Tabela de associação para o relacionamento many-to-many
author_book_association = Table(
    'author_book_association',
    Base.metadata,
    Column('author_id', Integer, ForeignKey('authors.id'), primary_key=True),
    Column('book_id', Integer, ForeignKey('books.id'), primary_key=True)
)

class AuthorModel(Base):
    __tablename__ = "authors"  # Nome da tabela no banco

    id = Column(Integer, primary_key=True, index=True)  # Chave primária
    name = Column(String, index=True)                   # Nome do autor
    email = Column(String, unique=True, index=True)     # Email único

    # Relacionamento many-to-many com livros
    books = relationship(
        "BookModel",                      # Classe relacionada
        secondary=author_book_association, # Tabela de associação
        back_populates="authors"          # Relacionamento reverso
    )

    def __repr__(self):
        return f"<AuthorModel(id={self.id}, name='{self.name}', email='{self.email}')>"
```

#### Explicação do Código

- `Base`: Classe base do SQLAlchemy para todos os models
- `__tablename__`: Nome da tabela no banco
- `Column`: Define cada coluna da tabela
- `relationship`: Cria o relacionamento entre tabelas
- `author_book_association`: Tabela extra para o relacionamento many-to-many

**Fluxo do relacionamento:**
Autor pode ter vários livros, e livro pode ter vários autores. O SQLAlchemy gerencia isso usando uma tabela de associação.

??? info "Equivalência em SQL"
    Esse código é equivalente à seguinte query em SQL:

    ```sql
    CREATE TABLE authors (
        id INTEGER PRIMARY KEY,
        name VARCHAR,
        email VARCHAR UNIQUE
    );

    CREATE TABLE author_book_association (
        author_id INTEGER,
        book_id INTEGER,
        PRIMARY KEY (author_id, book_id),
        FOREIGN KEY (author_id) REFERENCES authors(id),
        FOREIGN KEY (book_id) REFERENCES books(id)
    );
    ```

### 4.2 **Prática 2**: Model Book

Agora crie o model que representa a tabela de livros. Ele deve:

- Herdar de `Base`
- Definir os campos `id`, `title`, `isbn`
- Gerenciar o relacionamento com autores

??? tip "Dica: Relacionamento reverso"
    A tabela de associação já foi definida no código do autor, portanto, não é necessário criá-la novamente. Apenas use o mesmo nome da tabela de associação e o parâmetro `back_populates` para conectar os dois lados. Como em:
    ```python
    authors = relationship(
        "AuthorModel",
        secondary="author_book_association",
        back_populates="books"
    )
    ```

??? success "Solução"
    ```python title="app/models/book_model.py"
    from sqlalchemy import Column, Integer, String
    from sqlalchemy.orm import relationship
    from database.database import Base

    class BookModel(Base):
        __tablename__ = "books"  # Nome da tabela

        id = Column(Integer, primary_key=True, index=True)  # Chave primária
        title = Column(String, index=True)                  # Título do livro
        isbn = Column(String, unique=True, index=True)      # ISBN único
        
        # Relacionamento many-to-many com autores
        authors = relationship(
            "AuthorModel",
            secondary="author_book_association",
            back_populates="books"
        )

        def __repr__(self):
            return f"<BookModel(id={self.id}, title='{self.title}', isbn='{self.isbn}')>"
    ```

#### Explicação do Código

- `__tablename__`: Nome da tabela no banco
- `id`, `title`, `isbn`: Colunas principais
- `authors`: Relacionamento com autores
- `secondary`: Nome da tabela de associação
- `back_populates`: Conecta o relacionamento reverso

### 4.3 Inicialização dos Models

Para facilitar os imports e garantir que todos os models estejam registrados, crie o arquivo de inicialização:

```python title="app/models/__init__.py"
from .author_model import AuthorModel, author_book_association
from .book_model import BookModel

__all__ = ["AuthorModel", "BookModel", "author_book_association"]
```

### 4.4 Resumo Visual

```mermaid
erDiagram
    AUTHORS ||--o{ AUTHOR_BOOK_ASSOCIATION : wrote
    BOOKS ||--o{ AUTHOR_BOOK_ASSOCIATION : written_by
    AUTHORS {
        int id
        string name
        string email
    }
    BOOKS {
        int id
        string title
        string isbn
    }
    AUTHOR_BOOK_ASSOCIATION {
        int author_id
        int book_id
    }
```

### 4.5 Dicas e Armadilhas Comuns

- Sempre use `back_populates` nos dois lados do relacionamento
- O nome da tabela de associação deve ser igual nos dois models
- O SQLAlchemy não valida tipos automaticamente: use Pydantic nas entidades para isso
- O método `__repr__` ajuda no debug e nos prints

---

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
    Usamos `Generic[T]` para criar um repositório reutilizável para qualquer tipo de modelo. O código acima, por exemplo, pode ser usado tanto para `AuthorModel` quanto para `BookModel`, quanto para qualquer outro Modelo criado ao longo do projeto.

??? note "Equivalência em SQL"
    O método `get_all()` é  equivalente a:
    ```sql
    SELECT * FROM table_name;
    ```

    O método `get_by_id(id)` é equivalente a:
    ```sql
    SELECT * FROM table_name WHERE id = :id LIMIT 1;
    ```

    O método `add(entity)` é equivalente a:
    ```sql
    INSERT INTO table_name (columns...) VALUES (values...);
    ```

    O método `update(entity)` é equivalente a:
    ```sql
    UPDATE table_name SET column1 = value1, ... WHERE id = :id;
    ```

    O método `delete(id)` é equivalente a:
    ```sql
    DELETE FROM table_name WHERE id = :id;
    ```

### 5.2 Repository do Author

Crie `app/repositories/author_repository.py`:

```python title="app/repositories/author_repository.py"
from sqlalchemy.orm import Session
from models.author_model import AuthorModel
from repositories.base_repository import BaseRepository

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

??? note "Equivalência em SQL"
    O método `get_by_email(email)` é equivalente a:
    ```sql
    SELECT * FROM authors WHERE email = :email LIMIT 1;
    ```

    O método `get_by_name(name)` é equivalente a:
    ```sql
    SELECT * FROM authors WHERE name ILIKE '%' || :name || '%';
    ```

    O método `get_authors_with_books()` é equivalente a:
    ```sql
    SELECT DISTINCT a.* FROM authors a
    JOIN author_book_association aba ON a.id = aba.author_id
    JOIN books b ON aba.book_id = b.id;
    ```

### 5.3 **Prática 3**: Repository do Book

Crie o `BookRepository` no arquivo `app/repositories/book_repository.py`.

#### **Métodos adicionais necessários:**
- `get_by_title(title: str)`: Busca por título (parcial)
- `get_by_isbn(isbn: str)`: Busca por ISBN exato
- `get_books_with_authors()`: Livros com autores carregados

??? success "Solução"
    ```python title="app/repositories/book_repository.py"
    from sqlalchemy.orm import Session
    from models.book_model import BookModel
    from repositories.base_repository import BaseRepository

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

??? note "Equivalência em SQL"
    O método `get_by_title(title)` é equivalente a:
    ```sql
    SELECT * FROM books WHERE title ILIKE '%' || :title || '%';
    ```

    O método `get_by_isbn(isbn)` é equivalente a:
    ```sql
    SELECT * FROM books WHERE isbn = :isbn LIMIT 1;
    ```

    O método `get_books_with_authors()` é equivalente a:
    ```sql
    SELECT b.* FROM books b
    JOIN author_book_association aba ON b.id = aba.book_id
    JOIN authors a ON aba.author_id = a.id;
    ```
---

## Parte 6: Migrações com Alembic


### O que são Migrações de Banco de Dados?

Migração é o processo de **alterar a estrutura do banco de dados** de forma controlada e segura. Sempre que você precisa criar, modificar ou remover tabelas e colunas, é necessário aplicar uma migração.

**Por que migrações são importantes?**
- Permitem evoluir o banco sem perder dados
- Garantem que todos os ambientes (dev, prod, CI) fiquem sincronizados
- Facilitam o trabalho em equipe: todos aplicam as mesmas mudanças
- Evitam erros manuais e retrabalho

### Alembic: O Gerenciador de Migrações do SQLAlchemy

O **Alembic** é a ferramenta oficial de migração para projetos SQLAlchemy. Ele automatiza o processo de:
- Criar scripts de migração (Python + SQL)
- Versionar o schema do banco
- Aplicar e desfazer mudanças (upgrade/downgrade)
- Gerenciar histórico de alterações

**Vantagens de usar Alembic:**
- Não precisa escrever SQL "na mão" para cada alteração
- Permite gerar scripts automaticamente a partir dos models
- Ajuda a manter a saúde e integridade do banco
- Facilita rollback em caso de erro

---

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

```python title="alembic/env.py" hl_lines="21-24"
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

```python title="app/basic_example.py"
from database.database import SessionLocal, Base, engine
from models.author_model import AuthorModel
from models.book_model import BookModel
from repositories.author_repository import AuthorRepository
from repositories.book_repository import BookRepository

def main():
    # Criar uma sessão
    session = SessionLocal()
    
    try:
        # Instanciar repositórios
        author_repo = AuthorRepository(session)
        book_repo = BookRepository(session)
        author_book_association = Base.metadata.tables.get("author_book_association")

        # Apagar dados existentes para o exemplo
        session.query(AuthorModel).delete()
        session.query(BookModel).delete()
        session.query(author_book_association).delete()

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

Crie um exemplo em `app/advanced_example.py` que:

1. Cria múltiplos autores e livros
2. Estabelece relacionamentos many-to-many
3. Demonstra buscas complexas
4. Mostra operações de update e delete

??? success "Solução"
    ```python title="app/advanced_example.py"
    from database.database import SessionLocal, Base, engine
    from models.author_model import AuthorModel
    from models.book_model import BookModel
    from repositories.author_repository import AuthorRepository
    from repositories.book_repository import BookRepository
    from entities.author import Author
    from entities.book import Book

    def main():
        session = SessionLocal()

        try:
            author_repo = AuthorRepository(session)
            book_repo = BookRepository(session)
            author_book_association = Base.metadata.tables.get("author_book_association")
            
            # Limpar dados existentes para o exemplo
            session.query(AuthorModel).delete()
            session.query(BookModel).delete()
            session.query(author_book_association).delete()
            session.commit()

            # Criar autores usando entidades para validação
            authors_data = [
                {"name": "Clarice Lispector", "email": "clarice@email.com"},
                {"name": "Jorge Amado", "email": "jorge@email.com"},
                {"name": "Paulo Coelho", "email": "paulo@email.com"}
            ]

            authors = []
            for data in authors_data:
                # Usar entidade para validação
                author_entity = Author(**data)
                # Converter para modelo
                author_model = AuthorModel(name=author_entity.name, email=author_entity.email)
                author = author_repo.add(author_model)
                authors.append(author)
                print(f"Autor criado: {author.name}")

            # Criar livros usando entidades para validação
            books_data = [
                {"title": "A Hora da Estrela", "isbn": "978-8520925829"},
                {"title": "Gabriela, Cravo e Canela", "isbn": "978-8535902976"},
                {"title": "O Alquimista", "isbn": "978-8595081413"},
                {"title": "Água Viva", "isbn": "978-8520925836"}
            ]

            books = []
            for data in books_data:
                # Usar entidade para validação
                book_entity = Book(**data)
                # Converter para modelo
                book_model = BookModel(title=book_entity.title, isbn=book_entity.isbn)
                book = book_repo.add(book_model)
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

            # Update - Alterar email de um autor usando entidade para validação
            paulo = author_repo.get_by_email("paulo@email.com")
            # Validar com entidade
            updated_entity = Author(name=paulo.name, email="paulo.coelho@email.com")
            paulo.email = updated_entity.email
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
python app/basic_example.py

# Executar exemplo avançado
python app/advanced_example.py
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

```python title="Exemplo de integração futura em app/main.py"
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session
from database.database import SessionLocal
from repositories.author_repository import AuthorRepository
from entities.author import Author

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
    # ...
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