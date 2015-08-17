title: Fetching collections in Hibernate
tags:
  - Hibernate
  - Java
  - Performance
id: 94
categories:
  - Thoughts
date: 2013-10-17 18:39:32
---

During quite a few projects I've ran into issues with the performance of fetching collections in Hibernate. In most cases these performance problems could be fixed by switching from the default fetching strategy to a more suitable alternative. To explain the different ways of fetching collections I've created a [Hibernate FetchMode explained by example guide](/hibernate-fetchmode-explained-example/ "Hibernate FetchMode explained by example"). For those not familiar with the subject it could provide a good place to start tweaking Hibernate performance.