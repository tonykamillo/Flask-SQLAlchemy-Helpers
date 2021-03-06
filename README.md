# Description

Helpers classes and functions to make work with SQLAlchemy and Flask easier and less repetitive (DRY).


## Installing
```bash
pip install flask-sqlalchemy-helpers
```

## Usage
To explain how to use a lib, nothing is better than a complete example. The comments in the example code will guide you to how it works. It's really simple. Let's assume we have app.py module with  model abstraction the respresents a simple user permission system as follow:
```python
    import os
    from datetime import datetime
    from flask import Flask

    # Import JoinedInheritanceMixin to get the Joined Inheritance and SQLAlchemyDRY instead SQLAlchemy.
    from flask_sqlalchemy_helpers import SQLAlchemyDRY, JoinedInheritanceMixin


    app = Flask(__name__)
    app.config['BASE_FOLDER'] = os.path.dirname(os.path.abspath(__file__))
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///%s' % os.path.join(app.config['BASE_FOLDER'], 'db.sqlite3')

    # Instantiate SQLAlchemyDRY
    db = SQLAlchemyDRY(app)

    # Start create your mdoels.
    # If your want to implement Joined Inheritance just make your Base model inherits from JoinedInheritanceMixin and db.Model, otherwise just inherit from db.Model
    class Base(JoinedInheritanceMixin, db.Model):
        name = db.Column(db.String(100), nullable=False, index=True)
        created_at = db.Column(db.DateTime, default=datetime.now, nullable=False, index=True)
        updated_at = db.Column(db.DateTime, default=datetime.now, onupdate=datetime.now, nullable=False, index=True)

        def __repr__(self):
            return '%s' % self.name


    # As we can see there is no need to define a primary key field because db.Model does it for us.
    # Primary key are UUID based, wich is perfect for isolated microservices world. In this way we can garantee the identification is unique in the entire world wich makes URI to access the data more reliable
    class Department(Base):
        phone = db.Column(db.String(50))


    class User(Base):
        email = db.Column(db.String(100), nullable=False, index=True)
        password = db.Column(db.String(40), nullable=False, index=True)
        active = db.Column(db.Boolean, nullable=False, default=False, index=True)

        # For ForeignKey it's necessary to use UUID field type
        department_id = db.Column(db.UUID, db.ForeignKey('department.id'))
        department = db.relationship('Department', backref=db.backref('employees', lazy='dynamic'), foreign_keys=[department_id])

        def __repr__(self):
            return '%s - <%s>' % (self.name, self.email)


    class Permission(Base):
        pass


    class Group(Base):

        # For Many To Many situation we have the m2m helper that creates the intermediate table required by secondary parameter in relationship function.
        # The m2m helper requires 2 parameters that must to be the names of both tables involved in the relationship.
        # In the case bellow Permission and Group tables.
        permissions = db.relationship('Permission',
            secondary=db.m2m('Permission', 'Group'), lazy='dynamic',
            backref=db.backref('groups', lazy='dynamic'))

        users = db.relationship('User',
            secondary=db.m2m('User', 'Group'), lazy='dynamic',
            backref=db.backref('groups', lazy='dynamic'))


    db.create_all()
```
Everything else remains the same flask_sqlalchemy.SQLAlchemy's api.

That's it. Have fun.
