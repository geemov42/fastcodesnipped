optimize spring data jpa :
* open-in-view : false
* auto-commit : false (so it open the transaction one the first db call is executed)
* We can use TransactionTemplate to control which part of the code should be executed in a transaction
* If we have a service method transactional that call another service method with a transaction REQUIRED_NEW, we need to use TransactionTemplate to don't block the first transaction because in that case 2 connections are opened.
* Use the transactional to reduce the number of hibernate session created (one by query in a non transactional method)
* Use @Version private Long version to let hibernate that it is a new entity in the case you set the id manually.
* If you just want to fill an entity with an external entity in the database for the id, in place of using a findById that load the entity with a select, you can use a getReferenceById that don't execute a select and let hibernate execute an insert with the ids
* If we want to fetch without create the query with @Query and a String, we can use @EntityGraph(attributePaths = {"attr1", "attr2"}) to say to hibernate to fetch
* @DynamicUpdate will track which fields change and only execute a request with that fields (more optimize in case of big table)
* If you want to read and don't do update, use projection. Spring data jpa support to find a projection like NamesOnly (record) findNamesOnlyById(String id) where the id is the where. It will look by name inside the entity represented by the repository, so we don't need to specify a query by ourself
* If we don't want to have a lot of new methods inside the repository, we can use Dynamic projection.
* Jean Bisutti : quickperf to annoted junit test and define how many queries will be executed by the code under test

useful :
* https://github.com/gavlyukovskiy/spring-boot-data-source-decorator
