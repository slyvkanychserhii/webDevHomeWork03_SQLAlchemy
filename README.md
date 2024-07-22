# Web development: домашние задание 3 (Python)

```python
from contextlib import contextmanager

from sqlalchemy import (
    create_engine,
    ForeignKey,
    Column,
    Integer,
    String,
    Numeric,
    Boolean
)
from sqlalchemy.orm import (
    sessionmaker,
    declarative_base,
    relationship
)

# Задача 1: Создайте экземпляр движка для подключения к SQLite базе данных в памяти.

engine = create_engine("sqlite:///:memory:", echo=True)


# Задача 2: Создайте сессию для взаимодействия с базой данных, используя созданный движок.

session_factory = sessionmaker(bind=engine)


@contextmanager
def session_scope():
    session = session_factory()
    try:
        yield session
        session.commit()
    except Exception as e:
        session.rollback()
        print(f"An error occurred: {e}")
        raise
    finally:
        session.close()


# Задача 3: Определите модель продукта Product со следующими типами колонок:
# id: числовой идентификатор
# name: строка (макс. 100 символов)
# price: числовое значение с фиксированной точностью
# in_stock: логическое значение

Base = declarative_base()


class Product(Base):
    __tablename__ = "products"
    id = Column(Integer, primary_key=True)
    name = Column(String(100))
    price = Column(Numeric(10, 2))
    in_stock = Column(Boolean, default=True)

    def __repr__(self):
        return "<Product(name='%s', price='%s', in_stock='%s')>" % (
            self.name,
            self.price,
            self.in_stock,
        )


# Задача 4: Определите связанную модель категории Category со следующими типами колонок:
# id: числовой идентификатор
# name: строка (макс. 100 символов)
# description: строка (макс. 255 символов)

class Category(Base):
    __tablename__ = "categories"
    id = Column(Integer, primary_key=True)
    name = Column(String(100))
    description = Column(String(255))

    def __repr__(self):
        return "<Category(name='%s', description='%s')>" % (
            self.name,
            self.description,
        )


# Задача 5: Установите связь между таблицами Product и Category с помощью колонки category_id.

Product.category_id = Column(Integer, ForeignKey("categories.id"))
Product.category = relationship("Category", back_populates="products")
Category.products = relationship("Product", order_by=Product.id, back_populates="category")


Base.metadata.drop_all(engine)
Base.metadata.create_all(engine)


def insert_data():
    product1 = Product(name="Laptop", price=1200.00)
    product2 = Product(name="Smartphone", price=800.00)
    product3 = Product(name="Headphones", price=150.00, in_stock=False)
    product4 = Product(name="E-Book Reader", price=120.00)
    product5 = Product(name="Fantasy Novel", price=20.00)
    product6 = Product(name="Science Textbook", price=75.00, in_stock=False)
    product7 = Product(name="Jeans", price=40.00)
    product8 = Product(name="T-Shirt", price=15.00)
    product9 = Product(name="Jacket", price=100.00, in_stock=False)

    categories = [
        Category(name="Electronics", description="Devices and gadgets",
                 products=[product1, product2, product3]),
        Category(name="Books", description="Printed and digital books",
                 products=[product4, product5, product6]),
        Category(name="Clothing", description="Men's and women's clothing",
                 products=[product7, product8, product9])
    ]

    with session_scope() as session:
        session.add_all(categories)


insert_data()


def get_data():
    with session_scope() as session:
        all_categories = session.query(Category).all()
        for category in all_categories:
            print(category)
            for product in category.products:
                print(f"\t{product}")


get_data()

```
