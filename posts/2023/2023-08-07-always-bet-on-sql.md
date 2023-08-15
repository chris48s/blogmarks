<!--
.. title: Always bet on SQL
.. slug: always-bet-on-sql
.. date: 2023-08-07 00:00:00
.. tags: sql,sql
.. category: sql
.. link: 
.. description: 
.. type: text
-->

I don't often write a spicy opinion piece, but it is spicy opinion time: ORMs are fine, but it is not worth investing too much time into becoming an expert in one.

I first learned SQL over 20 years ago. Everything I learned then is still true now. Lots of other things have also become true in the meantime, but everything I learned then still holds.

I've also used a number of different ORMs in some depth. Some free-standing. Some attached to a particular framework.

- Django ORM
- SQLAlchemy
- Doctrine
- CakePHP ORM

All of those use different design patterns and provide conceptually quite different abstractions over the same underlying SQL. There are some common ideas between them, but if you pick any two from that list there are probably more differences between them than similarities. Only a subset of the knowledge from learning one is directly transferable to another ORM.

ORMs come and go as different languages and frameworks gain or lose traction, but the underlying SQL is a constant.

So here's a rule of thumb: ORMs are fine. They do a useful job. Even if you don't like them, you'll probably end up using one. But any given ORM doesn't really warrant too much energy or attention. Focus on SQL. Understand how an ORM maps onto the underlying SQL and learn your current ORM well enough to make it generate the SQL statement you want under the hood.

There is a good chance that 5 years from now:

- Anything you learn about SQL today will remain valid and useful, even when transitioning to a new ORM
- You are likely to be using a different ORM, and much of your current ORM knowledge may not apply
