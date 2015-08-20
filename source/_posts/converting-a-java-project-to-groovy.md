title: Converting a Java project to Groovy
tags:
  - Gradle
  - Groovy
  - Java
id: 158
categories:
  - Lessons learned
date: 2013-11-25 20:24:10
---

Lately I've been playing around with [Groovy ](http://groovy.codehaus.org/) and I tought I might try converting a Java project to Groovy. The example application which I've created to demonstrate [Hibernate FetchMode strategies](/2013/11/17/optimize-onetoone-mappings-hibernate/ "Hibernate FetchMode explained by example") seems ideal for the task. It contains a few Hibernate entities, which are basically just Java beans. And it has a small piece of 'application logic'. You may download the original code [here](/2013/11/17/optimize-onetoone-mappings-hibernate/Hibernate_Optimize_OneToOne_CodeExample.zip). For those who can't wait for the result the final code can be found [here](/2013/11/25/converting-a-java-project-to-groovy/Hibernate_FetchMode_Groovy.rar).

Now let's get started ..

<!-- more-->

#  Modify the Gradle build file 

In order to compile Groovy code we need to add the [Groovy plugin](http://www.gradle.org/docs/current/userguide/groovy_plugin.html) to the Gradle build: 
{% codeblock lang:groovy %}
   apply plugin:'groovy'
{% endcodeblock %}

We will also need the Groovy dependency on our projects classpath:
{% codeblock lang:groovy %}
   dependencies {
       compile group: 'org.codehaus.groovy', name: 'groovy-all', version: '2.2.0'
       ...
   }
{% endcodeblock %}

Now let's run our project to verify that the Groovy plugin is active:
{% codeblock lang:propertie %}
Executing: gradle run
:compileJava UP-TO-DATE
:compileGroovy UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:run
...
BUILD SUCCESSFUL
{% endcodeblock %} 

Notice that the compileGroovy task has been executed.

# Rename the source code

By default the Groovy plugin compiles *.java and *.groovy files found under src/main/groovy. So we rename the directory src/main/java to src/main/groovy and although it's not needed I've renamed all the *.java files to *.groovy. Reloading the project in the IDE shows that a Groovy source folder has been detected.
![convert_to_groovy_rename_sourcefile](/2013/11/25/converting-a-java-project-to-groovy/convert_to_groovy_rename_sourcefile.png)

# Change the source code

## Hibernate entities - Beans

For Groovy semicolons are optional and a bunch of imports are added by default which means that we can drop some clutter from the code. The real change however is that Groovy has build in [property support](http://groovy.codehaus.org/Groovy+Beans)! Something that has been missing from standard Java for way too long. By removing the access modifier (in our case private) from the field definitions we allow Groovy to use build in default. By default all fields are private and for each field a matching Get and Set method is generated.
So we can change our code from:
{% codeblock lang:java %}
@Entity
public class Invoice {
    @Id
    private Integer id

    @Column(columnDefinition="decimal", precision=128 , scale=0) 
    private BigDecimal total

    @ManyToOne
    @JoinColumn(name = "CUSTOMERID")
    private Customer customer

    public Integer getId() {
        return id
    }
    public BigDecimal getTotal() {
        return total
    }
    public Customer getCustomer() {
        return customer
    }  
}
{% endcodeblock %}

to:
{% codeblock lang:groovy %}
@Entity
class Invoice {

    @Id
    Integer id

    @Column(columnDefinition="decimal", precision=128 , scale=0) 
    BigDecimal total

    @ManyToOne
    @JoinColumn(name = "CUSTOMERID")
    Customer customer
}
{% endcodeblock %}
Without losing any functionality! If we look at the HibernateFetchModeExample class we will see that invoice.getTotal() is used without compile issues. Even tough we can't see the method in our source code it is there because the compiler added it for us.

## Example code - Iterations

Now let's take a look at the original example code which made a total of all the invoices:
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

In Groovy we can use something called a [GString ](http://groovy.codehaus.org/Strings+and+GString)to make our String formats more readable. And we can drop the brackets when we invoke a method as well, once again removing clutter from the code. The double loop which is used to calculate the total changes the most in our example. Groovy has support for [closures ](http://groovy.codehaus.org/Closures) so that the boiler-plate code needed to work with collections can be reduced to an absolute minimum.
Applying these changes makes our code look like this:
{% codeblock lang:groovy %}
    private void findTotalAmountForAllInvoice() throws HibernateException {
        def session = sessionFactory.openSession()

        println 'Load all Customers:'
        def crit = session.createCriteria(Customer.class)

        def customers = crit.list()
        println "Number of Customers: ${customers.size}"

        def allCustomerTotal = customers.sum{customer -> 
            println "Fetch the collection of Invoices for Customer ${customer.firstName} ${customer.lastName}"
            customer.invoices.sum{invoice -> invoice.total ?: 0}?: 0
        }
        println "Total amount for all invoices: ${allCustomerTotal}"
        session.close()
    }
{% endcodeblock %}
The code has changed quite a bit by using some basic Groovy features. And yes, it still works:
{% codeblock lang:propertie %}
:run
...
BUILD SUCCESSFUL
{% endcodeblock %} 

I've had a tremendous amount of fun converting this little piece of code. For me it seems that Groovy offers some very powerful language enhancements to Java even if I've only just scratched the surface. 
For those who missed it, you may download the final example code [here](/2013/11/25/converting-a-java-project-to-groovy/Hibernate_FetchMode_Groovy.rar).