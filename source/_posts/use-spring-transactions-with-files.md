title: How to use Spring transactions with files
tags:
  - Java
  - Spring
  - Transactions
id: 120
categories:
  - Lessons learned
date: 2013-10-28 18:07:44
---

It's not uncommon to write data to the database and the filesystem in the same transaction. The challenge is to remove files from the filesystem when the transaction fails. It is possible to  combine database Spring transactions with files on the filesystem .
The following example has a Spring-managed transactional method which will first write some data to the database. After doing so it will create a file on the filesystem. 

{% codeblock lang:java %}
@Transactional
public void methodInTransaction() {
        Data data = writeDataToDataBase();      
    File outputFile= createFileBasedOnData(data);
}
{% endcodeblock %}

If an exception is thrown during the commit (let's say a constraint violation) then there will still be an outputFile present on the filesystem. In some cases this is not desirable. Now let's see how we can extend Spring transactions with files support. First we need to create a TransactionSynchronization listener.

{% codeblock lang:java %}
public class CleanupTransactionListener extends TransactionSynchronizationAdapter {
   private File outputFile;

   public CleanupTransactionListener(File outputFile) {
      this.outputFile = outputFile;
   }

   @Override
   public void afterCompletion(int status) {
    if (STATUS_COMMITTED != status) {
            if (outputFile.exists()) {
                if (!outputFile.delete()) {
                    System.out.println("Could not delete File" 
                                         + outputFile.getPath() + " after failed transaction");
                }
            }
        }
    }
}
{% endcodeblock %}

This listener will try to remove the given file after a failed transaction. We still need to register the listener to the transaction. The TransactionSynchronizationManager from Spring may be used to register a transaction listener to the current thread.

{% codeblock lang:java %}
@Transactional
public void methodInTransaction() {
   CleanupTransactionListener transactionListener = new CleanupTransactionListener(outputFile);
   TransactionSynchronizationManager.registerSynchronization(transactionListener);

   Data data = writeDataToDataBase();       
   File outputFile= createFileBasedOnData(data);
}
{% endcodeblock %}

This will cause the TransactionSynchronizationManager to call our listener after completing the transaction. By checking the status passed to the listener we can decide which actions (if any) to take.