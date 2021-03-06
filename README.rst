=========
Quick ORM
=========


**News**: quick_orm is fully compatible with the newest SQLAlchemy 0.7.6.


|

Introduction
************
A python ORM which enables you to get started in less than a minute! 

Super easy to setup and super easy to use, yet super powerful! 

You would regret that you didn't discorver it earlier!


|

Features
********
- quick: you could get and play with it in less than a minute. It couldn't be more straightforward.
- easy: you don't have to write any SQL statements, including those "create table xxx ..." ones.
- simple: the core code counts only 208 lines including comments and pydocs, there is no room for bugs.
- free: released under BSD license, you are free to use it and distribute it.
- powerful: built upon SQLAlchemy and doesn't compromise its power.
- support relationships by means of python decorators.
- support table inheritance in a most natural way.
- support multiple databases: you can map your models to many databases without difficulty.
- write less, do more: taking advantage of python metaclass reduces data modeling code dramatically.
- long-term maintained: Continous efforts are taken to improve and maintain it.


|

Quick Start 
***********

::

    pip install quick_orm    

Refer to the following examples to write your own database manipulation code.


|

Hello World example
*******************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String
    
    __metaclass__ = Database.DefaultMeta
    
    class User:
        name = Column(String(30))
    
    Database.register()
    
    if __name__ == '__main__':
        db = Database('sqlite://')
        db.create_tables()
        
        user = User(name = 'Hello World')
        db.session.add_then_commit(user)
        
        user = db.session.query(User).get(1)
        print 'My name is', user.name


|

Foreign key example
*******************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String, Text
    
    __metaclass__ = Database.DefaultMeta
    
    class Question:
        title = Column(String(70))
        content = Column(Text)
    
    @Database.foreign_key(Question)
    class Answer:
        content = Column(Text)
    
    Database.register()
    
    if __name__ == '__main__':
        db = Database('sqlite://')
        db.create_tables()
        
        question = Question(title = 'What is Quick ORM ?', content = 'What is Quick ORM ?')
        answer = Answer(question = question, content = 'Quick ORM is a python ORM which enables you to get started in less than a minute!')
        db.session.add_then_commit(answer)
        
        question = db.session.query(Question).get(1)
        print 'The question is:', question.title
        print 'The answer is:', question.answers.first().content


|

Foreign key options example
***************************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String, Text
    
    __metaclass__ = Database.DefaultMeta
    
    class Question:
        title = Column(String(70))
        content = Column(Text)
    
    @Database.foreign_key(Question, ref_name = 'question', backref_name = 'answers')
    class Answer:
        content = Column(Text)
    
    Database.register()
    
    if __name__ == '__main__':
        db = Database('sqlite://')
        db.create_tables()
        
        question = Question(title = 'What is Quick ORM ?', content = 'What is Quick ORM ?')
        answer = Answer(question = question, content = 'Quick ORM is a python ORM which enables you to get started in less than a minute!')
        db.session.add_then_commit(answer)
        
        question = db.session.query(Question).get(1)
        print 'The question is:', question.title
        print 'The answer is:', question.answers.first().content


|

Foreign key to oneself example
******************************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String
    
    __metaclass__ = Database.DefaultMeta
    
    @Database.foreign_key('Node', ref_name = 'parent_node', backref_name = 'children_nodes')
    class Node:
        name = Column(String(70))
    
    Database.register()
    
    if __name__ == '__main__':
        db = Database('sqlite://')
        db.create_tables()
    
        root_node = Node(name = 'root')
        node1 = Node(name = 'node1', parent_node = root_node)
        node2 = Node(name = 'node2', parent_node = root_node)
        db.session.add_then_commit(root_node)
    
        root_node = db.session.query(Node).filter_by(name = 'root').one()
        print 'Root node have {0} children nodes, they are {1}'\
            .format(root_node.children_nodes.count(), ', '.join(node.name for node in root_node.children_nodes))


|

Many-to-many relationship example
*********************************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String
    
    __metaclass__ = Database.DefaultMeta
    
    class User:
        name = Column(String(30))
    
    @Database.many_to_many(User)
    class Role:
        name = Column(String(30))
    
    Database.register()
    
    if __name__ == '__main__':
        db = Database('sqlite://')
        db.create_tables()
        
        user1 = User(name = 'Tyler Long')
        user2 = User(name = 'Peter Lau')
        role = Role(name = 'Administrator', users = [user1, user2])
        db.session.add_then_commit(role)
    
        admin_role = db.session.query(Role).filter_by(name = 'Administrator').one()
        print ', '.join([user.name for user in admin_role.users]), 'are admintrators'


|

Many-to-many relationship options example
*****************************************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String
    
    __metaclass__ = Database.DefaultMeta
    
    class User:
        name = Column(String(30))
    
    @Database.many_to_many(User, ref_name = 'users', backref_name = 'roles', middle_table_name = 'user_role')
    class Role:
        name = Column(String(30))
    
    Database.register()
    
    if __name__ == '__main__':
        db = Database('sqlite://')
        db.create_tables()
        
        user1 = User(name = 'Tyler Long')
        user2 = User(name = 'Peter Lau')
        role = Role(name = 'Administrator', users = [user1, user2])
        db.session.add_then_commit(role)
    
        admin_role = db.session.query(Role).filter_by(name = 'Administrator').one()
        print ', '.join([user.name for user in admin_role.users]), 'are admintrators'


|

Many-to-many relationship with oneself example
**********************************************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String
    
    __metaclass__ = Database.DefaultMeta
    
    @Database.many_to_many('User', ref_name = 'users_i_follow', backref_name = 'users_follow_me')
    class User:
        name = Column(String(30))
    
    Database.register()
    
    if __name__ == '__main__':
        db = Database('sqlite://')
        db.create_tables()
        
        peter = User(name = 'Peter Lau')
        mark = User(name = 'Mark Wong', users_i_follow = [peter, ])
        tyler = User(name = 'Tyler Long', users_i_follow = [peter, ], users_follow_me = [mark, ])
        db.session.add_then_commit(tyler)
    
        tyler = db.session.query(User).filter_by(name = 'Tyler Long').one()
        print 'Tyler Long is following:', ', '.join(user.name for user in tyler.users_i_follow)
        print 'People who are following Tyler Long:', ', '.join(user.name for user in tyler.users_follow_me)
        mark = db.session.query(User).filter_by(name = 'Mark Wong').one()
        print 'Mark Wong is following:', ', '.join(user.name for user in mark.users_i_follow)


|

One-to-one relationship example
*******************************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String
    
    __metaclass__ = Database.DefaultMeta
    
    class User:
        name = Column(String(30))
    
    @Database.foreign_key(User, one_to_one = True)
    class Contact:
        email = Column(String(70))
        address = Column(String(70))
    
    Database.register()
    
    if __name__ == '__main__':
        db = Database('sqlite://')
        db.create_tables()
    
        contact = Contact(email = 'quick.orm.feedback@gmail.com', address = 'Shenzhen, China')
        user = User(name = 'Tyler Long', contact = contact)
        db.session.add_then_commit(user)
        
        user = db.session.query(User).get(1)
        print 'User:', user.name
        print 'Email:', user.contact.email
        print 'Address:', user.contact.address


|

Multiple foreign keys example
*****************************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String, Text
    
    __metaclass__ = Database.DefaultMeta
    
    class User:
        name = Column(String(30))
    
    @Database.foreign_key(User, ref_name = 'author', backref_name = 'articles_authored')
    @Database.foreign_key(User, ref_name = 'editor', backref_name = 'articles_edited')
    class Article:
        title = Column(String(80))
        content = Column(Text)
    
    Database.register()
    
    if __name__ == '__main__':
        db = Database('sqlite://')
        db.create_tables()
        
        author = User(name = 'Tyler Long')
        editor = User(name = 'Peter Lau')
        article = Article(author = author, editor = editor, title = 'Quick ORM is super quick and easy', 
            content = 'Quick ORM is super quick and easy. Believe it or not.')
        db.session.add_then_commit(article)
        
        article = db.session.query(Article).get(1)
        print 'Article:', article.title
        print 'Author:', article.author.name
        print 'Editor:', article.editor.name


|

Performing raw sql query example
********************************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String
    
    __metaclass__ = Database.DefaultMeta
    
    class User:
        name = Column(String(70))
    
    Database.register()
    
    if __name__ == '__main__':
        db = Database('sqlite://')
        db.create_tables()
        
        count = db.engine.execute('select count(name) from user').scalar()
        print 'There are {0} users in total'.format(count)


|

Multiple databases example
**************************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String
    
    __metaclass__ = Database.DefaultMeta
    
    class User:
        name = Column(String(30))
    
    Database.register()
    
    if __name__ == '__main__':
        db1 = Database('sqlite://')
        db1.create_tables()
    
        db2 = Database('sqlite://')
        db2.create_tables()
        
        user1 = User(name = 'user in db1')
        user2 = User(name = 'user in db2')
        db1.session.add_then_commit(user1)
        db2.session.add_then_commit(user2)
        
        print 'I am', db1.session.query(User).get(1).name
        print 'I am', db2.session.query(User).get(1).name


|

Table inheritance example
*************************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String, Text
    
    __metaclass__ = Database.DefaultMeta
    
    class User:
        name = Column(String(70))
    
    @Database.foreign_key(User)
    class Post:
        content = Column(Text)
    
    class Question(Post):
        title = Column(String(70))    
    
    @Database.foreign_key(Question)
    class Answer(Post):
        pass
    
    @Database.foreign_key(Post)
    class Comment(Post):
        pass
    
    @Database.many_to_many(Post)
    class Tag:
        name = Column(String(70))
        
    Database.register()
    
    if __name__ == '__main__':
        db = Database('sqlite://')
        db.create_tables()
    
        user1 = User(name = 'Tyler Long')
        user2 = User(name = 'Peter Lau')
        
        tag1 = Tag(name = 'quick_orm')
        tag2 = Tag(name = 'nice')
        
        question = Question(user = user1, title = 'What is quick_orm ?', content = 'What is quick_orm ?', tags = [tag1, ])
        question2 = Question(user = user1, title = 'Have you tried quick_orm ?', content = 'Have you tried quick_orm ?', tags = [tag1, ])
    
        answer = Answer(user = user1, question = question, tags = [tag1, ],
            content = 'quick_orm is a python ORM which enables you to get started in less than a minute!')
        
        comment1 = Comment(user = user2, content = 'good question', post = question)
        comment2 = Comment(user = user2, content = 'nice answer', post = answer, tags = [tag2, ])
    
        db.session.add_all_then_commit([question, question2, answer, comment1, comment2, tag1, tag2, ])
    
        question = db.session.query(Question).get(1)
        print 'tags for question "{0}": "{1}"'.format(question.title, ', '.join(tag.name for tag in question.tags))
        print 'new comment on question:', question.comments.first().content
        print 'new comment on answer:', question.answers.first().comments.first().content
    
        user = db.session.query(User).filter_by(name = 'Peter Lau').one()
        print 'Peter Lau has posted {0} comments'.format(user.comments.count())
    
        tag = db.session.query(Tag).filter_by(name = 'quick_orm').first()
        print '{0} questions are tagged "quick_orm"'.format(tag.questions.count())


|

MetaBuilder to avoid duplicate code example
*******************************************

::

    from quick_orm.core import Database
    from sqlalchemy import Column, String, DateTime, func
    
    class DefaultModel:
        name = Column(String(70))
        created = Column(DateTime, default = func.now(), nullable = False)
    
    __metaclass__ = Database.MetaBuilder(DefaultModel)
    
    class User:
        pass
    
    class Group:
        pass
    
    Database.register()
    
    if __name__ == '__main__':
        db = Database('sqlite://')
        db.create_tables()
        user = User(name = 'tylerlong')
        db.session.add(user)
        group = Group(name = 'python')
        db.session.add_then_commit(group)
    
        print user.name, user.created
        print group.name, group.created


|

Examples from real life
***********************
- Everblog_ is a personal blogging platform taking advantage of evernote, it chooses quick_orm as its ORM framework. Refer to `everblog's database model file`_ for more detail.

.. _Everblog: https://github.com/tylerlong/everblog
.. _`everblog's database model file`: https://github.com/tylerlong/everblog/blob/master/everblog/models.py

If you know any other successful stories about quick_orm, do tell me and I will list them above. 
   
  
|

Where to learn more about quick_orm?
************************************
As said above, quick_orm is built upon SQLAlchemy. Quick ORM never tries to hide SQLAlchemy's flexibility and power. Everything availiable in SQLAlchemy is still available in quick_orm. 

So please read the documents of SQLAlchemy, you would learn much more there than you could here.  

Read quick_orm's source code, try to improve it.


|

You wanna involve? 
******************
Quick ORM is released under BSD lisence.

The source code is hosted on github: https://github.com/tylerlong/quick_orm


|

Acknowledgements
****************
Quick ORM is built upon SQLAlchemy - the famous Python SQL Toolkit and Object Relational Mapper. All of the glory belongs to the SQLAlchemy development team and the SQLAlchemy community! My contribution to Quich ORM becomes trivial compared with theirs( to SQLAlchemy).


|

Feedback 
********
Comments, suggestions, questions, free beer, t-shirts, kindles, ipads ... are all welcome! 

Email: quick.orm.feedback@gmail.com 
