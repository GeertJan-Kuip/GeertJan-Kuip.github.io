# Spring exam - JDBC

## Basic usage of JdbcTemplate

Using Spring with JDBC gives a lot of flexibility with regards to writing queries and mapping results to generic collections or to custom domain objects. Things are made easier by the limited number of query methods (.query, .queryForObject, .queryForMap, .queryForList and .update). If you want to iterate over resultsets yourself, you can with ResultSetExtractor. Error handling has been thought out well, it seems, with for me a new lesson about using unchecked exceptions. 

### Setting things up

The central element in the Jdbc approach is the JdbcTemplate. Create it once and pass it to the database repository class, after that you can use it over and over again:

```
public class JdbcCustomerRepository implements CustomerRepository {

	private JdbcTemplate jdbcTemplate; // store this in the class so you can use it as you like.

	public JdbcCustomerRepository(JdbcTemplate jdbcTemplate){
		this.jdbcTemplate = jdbcTemplate;
	}

	public int getCostumerCount(){
		String sql = "select count(*) from customer";
		return jdbcTemplate.queryForObject(sql, Integer.class);
	}
}
```

A matching configuration class can look like this:

```
@Configuration
public class AppConfig {

    @Bean
    public DataSource dataSource() {
        // Use any implementation, e.g. HikariDataSource, DriverManagerDataSource, etc.
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setDriverClassName("com.mysql.cj.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/mydb");
        ds.setUsername("user");
        ds.setPassword("pass");
        return ds;
    }

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean
    public CustomerRepository customerRepository(JdbcTemplate jdbcTemplate) {
        return new JdbcCustomerRepository(jdbcTemplate);
    }
}
```

So basically you have three beans, namely dataSource, jdbcTemplate and customerRepository. With Spring Boot it gets even easier, but that is for another chapter.

ChatGPT told me that CustomerRepository is not a Spring interface but something you might create yourself to define the contract about what this specific repository should do.

### CRUD methods

You can do a SELECT query for simple types, for collections/generic maps, and for domain objects. The latter is an object type defined in your application. The basic method is `queryForObject(..)`. 

For INSERT, UPDATE and DELETE you use the `update(..)` method. It is a sort of convenience that one method does several things.

#### queryForObject(..)

If you expect a single result and you do not need to pass variable, you can use `queryForObject(..)` with only two parameters, namely the sql query (type String) and the .class of the expected return type. Example:

```
public Date getOldest() {  // no arguments
	
	String sql = "select min(date_of_birth) from PERSON";
	return jdbcTemplate.queryForObject(sql, Date.class);  // query returns a single Date type object
}
```

If your query contains bind variables, you use another overload of queryForObject with some varargs addition:

```
public int getCountOfNationalsOver(Nationality nationality, int age) {
	
	String sql = "select count(*) from PERSON where age>?  and nationality=?";  // order of binds 

	return jdbcTemplate.queryForObject(sql, Integer.class, age, nationality.toString());  // matches with this order
}
```

Note that here the order bind variables in the sql statement matches the order of bind variables in the queryForObject method arguments. Again, the result is a single value (type int). Later on there will be examples where the resultset is more extensive.

#### update(..)

As said, update(..) can be used for INSERT, UPDATE and DELETE. Example:

```
public int insertPerson(Person person) {

	sql = "insert into PERSON (first_name, last_name, age) values (?,?,?)";
	
	return jdbcTemplate.update(sql, person.getFirstName(), person.getLastName, person.getAge());
}
```

Example using UPDATE:

```
public int updateAge(Person person) {

	sql = "update Person set age=? where id=?";
	
	return jdbcTemplate.update(sql, person.getAge, person.getId());
}
```

## Working with ResultSets using Callbacks

### Mapping to generic map or collection

For this the `queryForMap(..)` and `queryForList(..)` can be used. You get a map or list as return type of this method. A map will typically contain multiple columns from a single row, a list can contain multiple mapseach containing a single row. The map has the form of a Map<String, Object>, so it can store a diversity of types. Example:

#### queryForMap

```
public Map<String, Object> getPersonInfo(int d) {
	
	String sql = "select * from PERSON where id=?";
	return jdbcTemplate.queryForMap(sql, id);  // id is the bind variable, there can be multiple bind variables as well
}

// returns Map {ID=1, FIRST_NAME="JOHN", LAST_NAME="DOE"}
```

Note that you do not have to provide the return type of the query as argument anymore.

#### queryForList

You will get a List<Map<String, Object>> as return type. Example:

```
public List<Map<String, Object>> getAllPersonInfo() {
	
	String sql = "select * from PERSON";
	return jdbcTemplate.queryForList(sql); // just one argument, but there are possibly overloads with varargs for bind variables
}
// returns a list of maps, one map per row. 
```

### Mapping to a domain object

Now a callback method is required. This callback method is a functional interface and is used as the second argument of the `.query(..)` or `.queryForObject(..)` method.

#### RowMapper

RowMapper is a Spring functional interface for mapping a single row of a ResultSet to an object. Can be used in single and multiple row queries. It is parameterized to define its return type. Its only method is `.mapRow(..)` and the method will be iterated if there are multiple rows in the ResultSet.

```
public interface RowMapper<T> {
	T mapRow(ResultSet rs, int rowNum) throws SQLException;
}
```

Simple example using .queryForObject(..):

```
public Person getPerson(int id) {
	
	String sql = "select first_name, last_name from PERSON where id=?";
	return jdbcTemplate.queryForObject(
			sql, 
			(rs, rowNum) -> new Person(rs.getString("first_name"), rs.getString("last_name")), 
			id);
}
```

Using `.query(..)`. If you expect multiple rows instead of one you can change the method from .queryForObject to .query and the return value to List<Person>. Bind variables are optional, the example below doesn't have one.

```
public List<Person> getAllPersons() {
	
	String sql = "select first_name, last_name from PERSON";
	return jdbcTemplate.query(
			sql, 
			(rs, rowNum) -> new Person(rs.getString("first_name"), rs.getString("last_name")) 
			);
}
```

#### ResultSetExtractor

While RowMapper will map each row to a single domain object, ResultSetExtractor will map the whole ResultSet to a single domain object. To get the resultset into your object, you need to iterate over the resultset yourself. In this respect it looks much like working without Spring. The good thing is that you do not end up with a list filled with maps but instead can make a more efficient or appropriate object.

ResultSetExtractor is a functional interface just like RowMapper:

```
public interface ResultSetExtracto<T> {
	T extractData(ResultSet rs) throws SQLException, DataAccessException;
}
```

This is an example of its use:

```
public class JdbcOrederRepository {

  public Order findByConfirmationNumber(String number) {

    sql = "select ... [some quert returning resultset with multiple rows and columns] blabla=?";
          return jdbcTemplate.query(sql,
                    (ResultSetExtractor<Order>)(rs)->{   // cast needed
                        Order order = null;
                        while (rs.next()){
                            if (order==null)
                                 order = new Order(rs.getLong("ID"), rs.getString("NAME"), ...);
                            order.addItem(mapItem(rs));
                        }
                        return order;
                     },
                     number);
  }
}
```

You can make the code cleaner by creating the lambda implementation in another method and then call that method.

#### RowCallbackHandler

This is the third option, no example was provided. It works like a RowMapper but you don't return a value. This is usefull for example when you want to stream the data from a query. You process the data but do not store it in a domain object or a list of domain objects. This is the code of the functional interface:

```
public interface RowCallbackHandler {

	void processRow(ResultSet rs) throws SQLException;
}
```

## Exception Handling

Checked exceptions force developers to handle errors or to declare them if handling is not possible. In the latter case, intermediate methods must declare everything thrown to them by all methods below and pass it on to higher-level methods. This is an undesirable form of tight-coupling with a lot of extra code.

Unchecked exceptions do not have this problem. The compiler doesn't enforce the handling of exceptions, a RunTimeException() can be thrown in a low level method, get unnoticed by all intermediate methods, and be caught at the very top. You will never see 'throws' in any method signature, and as long as there is a catch for the RunTimeException anywhere, the program won't crash.

This was new for me. I was in the understanding that unchecked exceptions would always crash the program, or at least that to not crash the program, they should be handled with a lot of 'throws' in method signatures. Thus not. 

Spring uses this mechanism to loosen up coupling and to avoid boiler plate. It simply always throws unchecked exceptions instead of checked exceptions and handles them at the top. No trace of it is seen in signatures or intermediate methods.

### SQLException and DataAccessException

JDBC throws the very general SQLException, which is a checked exception. In the exception messages you will find information about the type of database you use and the vendor. You can see if JDBC, Hibernate or JPA is being used etc.

Spring doesn't want this, not the checked exception, not the generic nature of the exception, and not the specifics about database type and vendor. There fore it introduced DataAccessException, an unchecked exception extending RunTimeException that is the parent of a range of child exceptions that are more specific about the problem, but do not have information about database type in their messaging.

### Child exceptions of DataAccessException

These are five of the subtypes of DataAccessException. There are more but this was in the tutorial. It shows that very specific names are being used which help you to solve the problem:

- DataAccessResourceFailureException
- CleanupFailureDataAccessException
- OptimisticLockingFailureException
- DataIntegrityViolationException
- BadSqlGrammarException

This is it for the tutorial content.


