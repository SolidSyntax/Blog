title: Hibernate FetchMode explained by example
id: 46
date: 2013-10-17 18:23:21
---

The way [Hibernate ](http://www.hibernate.org/)fetches data when accessing a collection is highly customizable. Most commonly a decision is made to either load the collection when loading the entity ([FetchType.EAGER](http://docs.oracle.com/javaee/6/api/javax/persistence/FetchType.html#EAGER)) or to delay the loading until the collection is used ([FetchType.LAZY](http://docs.oracle.com/javaee/6/api/javax/persistence/FetchType.html)).

It is also possible to specify how the data should be fetched by setting the Hibernate FetchMode. This will customize the amount of queries generated and how much data will be retrieved. Therefore it might be an important tool to optimize your Hibernate application.

## Example application

To demonstrate the different fetchmodes I've created a small example application. This example has a Hibernate mapping for the following database schema:
[![ExampleDB_Diagram](/hibernate-fetchmode-explained-example/index/ExampleDB_Diagram.jpg)](/hibernate-fetchmode-explained-example/index/ExampleDB_Diagram.jpg)

A customer has zero or more invoices and each invoice has a total amount. This example will retrieve all customers, get their invoices, and calculate the total amount over all customers.

{% codeblock lang:java %}
    private void findTotalAmountForAllInvoice() throws HibernateException {
        Session session = sessionFactory.openSession();

        System.out.println("Load all Customers:");

        Criteria crit = session.createCriteria(Customer.class);
        @SuppressWarnings("unchecked")
        List<Customer> customers = crit.list();

        System.out.println("Number of Customers: " + customers.size());
        BigDecimal allCustomerTotal = BigDecimal.ZERO;
        for (Customer customer : customers) {
            System.out.println(String.format("Fetch the collection of Invoices for Customer %s %s", customer.getFirstName(), customer.getLastName()));
            Set<Invoice> invoices = customer.getInvoices();
            for (Invoice invoice : invoices) {
                allCustomerTotal = allCustomerTotal.add(invoice.getTotal());
            }
        }
        System.out.println("Total amount for all invoices: "+ allCustomerTotal.toString());
        session.close();
    }
{% endcodeblock %}
You may download the example [here](/hibernate-fetchmode-explained-example/index/Hibernate_FetchMode_CodeExample.zip). The zip contains a [Gradle project](http://www.gradle.org/). To run the example [install gradle ](http://www.gradle.org/installation) and run **gradle run** in the directory containing the build.gradle file.

## Default Hibernate FetchMode: SELECT

You can explicitly set the Hibernate FetchMode in the Customer class by annotating the invoices collection:
{% codeblock lang:java %}
    @OneToMany(mappedBy = "customer")
    @Fetch(FetchMode.SELECT)
    private Set<Invoice> invoices;
{% endcodeblock %}

Running the example gives the following output:
 {% codeblock lang:propertie %}
Load all Customers:
Hibernate: select this_.id as id1_0_0_, this_.city as city2_0_0_, this_.firstName as firstNam3_0_0_, this_.lastName as lastName4_0_0_, this_.street as street5_0_0_ from Customer this_
Number of Customers: 50

Fetch the collection of Invoices for Customer Laura Steel
Hibernate: select invoices0_.CUSTOMERID as CUSTOMER3_0_1_, invoices0_.id as id1_1_1_, invoices0_.id as id1_1_0_, invoices0_.CUSTOMERID as CUSTOMER3_1_0_, invoices0_.total as total2_1_0_ from Invoice invoices0_ where invoices0_.CUSTOMERID=?

Fetch the collection of Invoices for Customer Susanne King
Hibernate: select invoices0_.CUSTOMERID as CUSTOMER3_0_1_, invoices0_.id as id1_1_1_, invoices0_.id as id1_1_0_, invoices0_.CUSTOMERID as CUSTOMER3_1_0_, invoices0_.total as total2_1_0_ from Invoice invoices0_ where invoices0_.CUSTOMERID=?

... and so for each Customer

Total amount for all invoices: 121157
{% endcodeblock %}

The Hibernate FetchMode SELECT generates a query for each Invoice collection loaded. In total that gives 1 query to load the Customers and 50 additional queries to load the Invoice collections. 
This behavior is commonly named the N + 1 select problem. Executing 1 query will trigger N additional queries, where N is the amount of results returned from the first query.

## Hibernate FetchMode: SELECT with BatchSize

SELECT has an optional configuration annotation called BatchSize:
{% codeblock lang:java %}
    @OneToMany(mappedBy = "customer")
    @Fetch(FetchMode.SELECT)
    @BatchSize(size=25)
    private Set<Invoice> invoices;
{% endcodeblock %}

This changes the output to:
 {% codeblock lang:propertie %}
Load all Customers:
Hibernate: select this_.id as id1_0_0_, this_.city as city2_0_0_, this_.firstName as firstNam3_0_0_, this_.lastName as lastName4_0_0_, this_.street as street5_0_0_ from Customer this_
Number of Customers: 50

Fetch the collection of Invoices for Customer Laura Steel
Hibernate: select invoices0_.CUSTOMERID as CUSTOMER3_0_1_, invoices0_.id as id1_1_1_, invoices0_.id as id1_1_0_, invoices0_.CUSTOMERID as CUSTOMER3_1_0_, invoices0_.total as total2_1_0_ from Invoice invoices0_ where invoices0_.CUSTOMERID in (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
Fetch the collection of Invoices for Customer Susanne King
Fetch the collection of Invoices for Customer Anne Miller
.. 23 additional collection fetches without a query

Fetch the collection of Invoices for Customer Sylvia Steel
Hibernate: select invoices0_.CUSTOMERID as CUSTOMER3_0_1_, invoices0_.id as id1_1_1_, invoices0_.id as id1_1_0_, invoices0_.CUSTOMERID as CUSTOMER3_1_0_, invoices0_.total as total2_1_0_ from Invoice invoices0_ where invoices0_.CUSTOMERID in (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
Fetch the collection of Invoices for Customer James Clancy
Fetch the collection of Invoices for Customer Bob Sommer
.. 23 additional collection fetches without a query

Total amount for all invoices: 121157
{% endcodeblock %}

In this example BatchSize reduces the total amount of queries to 3\. One to load the Customers and two additional queries to load the Invoice collections for all customers. To put it simple: When an Invoice collection is loaded for a specific Customer, Hibernate will try to load the Invoice collection for up to 25 additional Customer entities which are currently in the session. The example has 50 Customers so loading the 50 collections of Invoices takes 2 queries. 

## Hibernate FetchMode: JOIN

Hibernate FetchMode JOIN tries to load the Customers and the Invoice collections in one query

{% codeblock lang:java %}
    @OneToMany(mappedBy = "customer")
    @Fetch(FetchMode.JOIN)
    private Set<Invoice> invoices;  
{% endcodeblock %}

Output:
{% codeblock lang:propertie %}
Load all Customers:
Hibernate: select this_.id as id1_0_1_, this_.city as city2_0_1_, this_.firstName as firstNam3_0_1_, this_.lastName as lastName4_0_1_, this_.street as street5_0_1_, invoices2_.CUSTOMERID as CUSTOMER3_0_3_, invoices2_.id as id1_1_3_, invoices2_.id as id1_1_0_, invoices2_.CUSTOMERID as CUSTOMER3_1_0_, invoices2_.total as total2_1_0_ from Customer this_ left outer join Invoice invoices2_ on this_.id=invoices2_.CUSTOMERID

Number of Customers: 71
Fetch the collection of Invoices for Customer Laura Steel
Fetch the collection of Invoices for Customer Susanne King
Fetch the collection of Invoices for Customer Anne Miller
Fetch the collection of Invoices for Customer Michael Clancy
Fetch the collection of Invoices for Customer Sylvia Ringer
Fetch the collection of Invoices for Customer Sylvia Ringer
Fetch the collection of Invoices for Customer Sylvia Ringer
... No additional queries
Total amount for all invoices: 262599
{% endcodeblock %}

The amount of queries has been reduced to 1 by joining the Customers and Invoice collections. FetchMode JOIN always triggers an EAGER load so the Invoices are loaded when the Customers are.  But when we look at the result there seems to be something wrong. There where 71 Customers found and the total amount seems to be different as well. This can be explained by the fact that **FetchMode JOIN returns duplicate results** when an entity has more then one record in the joined collection! In this example the Customer named Sylvia Ringer has 3 Invoices so she is included 3 times in the result. You'll have to remove the duplicates yourself (e.g. storing the result in a Set).

## Hibernate FetchMode: SUBSELECT

The final Hibernate FetchMode available is the SUBSELECT
{% codeblock lang:java %}
    @OneToMany(mappedBy = "customer")
    @Fetch(FetchMode.SUBSELECT)
    private Set<Invoice> invoices;
{% endcodeblock %}

Output:
{% codeblock lang:propertie %}
Hibernate: select this_.id as id1_0_0_, this_.city as city2_0_0_, this_.firstName as firstNam3_0_0_, this_.lastName as lastName4_0_0_, this_.street as street5_0_0_ from Customer this_
Number of Customers: 50

Fetch the collection of Invoices for Customer Laura Steel
Hibernate: select invoices0_.CUSTOMERID as CUSTOMER3_0_1_, invoices0_.id as id1_1_1_, invoices0_.id as id1_1_0_, invoices0_.CUSTOMERID as CUSTOMER3_1_0_, invoices0_.total as total2_1_0_ from Invoice invoices0_ where invoices0_.CUSTOMERID in (select this_.id from Customer this_)
Fetch the collection of Invoices for Customer Susanne King
Fetch the collection of Invoices for Customer Anne Miller
... No additional queries
Total amount for all invoices: 121157
{% endcodeblock %}

A SUBSELECT generates one query to load the Customers and one additional query to fetch all the Invoice collections. It is important to notice that all Invoices are loaded for which there is a corresponding Customer in the database. So even Invoice collections for who there are no matching Customers in the session will be retrieved.

## Which Hibernate FetchMode should I use?

Which FetchMode to use depends heavily on the application, environment and typical usage. The following guideline should be seen as a rough indication of where to start. Try to play with the setting to see what works best in your application / environment:

**FetchMode SELECT**
Use this when you want a quick response time when working on a single entity. SELECT creates small queries and only fetches the data which is absolutely needed. The use-case in our example could be an application to which displays one Customer with its Invoices.

**BatchSize**
BatchSize is useful when working with a fixed set of data. When you have a batch processing 10 Customers at a time a BatchSize of 10 will drastically reduced the number of queries needed.If the BatchSize is not set too high the query will most likely return a manageable amount of data in a reasonable time.  

**FetchMode JOIN**
As indicated you'll have to worry about duplicated results. On the other hand JOIN creates the least amount of queries. In a high latency environment a single JOIN could be considerable faster then multiple SELECTS. Keep in mind that joining too much data could put a strain on the database.

**FetchMode SUBSELECT**
If you've got an entity of which you know that there aren't that many of them, and almost all of them are in the session, then SUBSELECT should be a good choice. Just keep in mind that all collections are fetched, even if the parent is not in the session. A SUBSELECT when having a single Customer in session while there are 1000+ in the database will be wasteful.

 