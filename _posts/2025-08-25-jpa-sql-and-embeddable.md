# JPA, SQL and @Embeddable

I'm working on an application that provides information on dutch towns. Creating it is less straightforward as I initially thought, for example because town is not a geographical unit in the data structure of the Central Bureau of Statistics. It requires some jiggling with different datasets, also from PDOK, and some JOIN statements in my database queries. That's what this blog is about.

## Spring Boot and native SQL queries

I was always used to write SQL queries in Java, PHP or Python, with either MySql or SQLite as database. Databases are great, superfast and you can ask them [anything you want](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-01-26-you-can-ask-databases-anything.md). It requires some skill but you can become good at it.

### Using @Query with _nativeQuery=true_

When working with Spring Boot, you can still write old fashioned queries like this in your repository classes (example copied from ChatGPT):

```
@Query(value = "SELECT * FROM orders o " +
               "JOIN customers c ON o.customer_id = c.id " +
               "WHERE c.name = :name",
       nativeQuery = true)
List<Order> findOrdersByCustomerName(@Param("name") String name);
```

It is the nativeQuery parameter attribute that allows you to write a real sql query. Without `nativeQuery=true`, the query must be written in so-called JPQL language, which is sql language in which you use the names of your entity classes and its fields instead of the names that are in the database. Those names can be identical but due to all sorts of conversion between naming conventions, this is often not the case. 

Note that the difference between the names ot @Entity-annotated classes and its member fields can also be orchestrated by the use of the @Column annotation. This annotation is useful if you want to explicitly make a difference between column names and field names:

```
    @Column(name="cbs_wijkenenbuurten")
    private String cbsWijkenEnBuurten;
```

Note that if you autogenerate getter and setter for this field in Intellij, these methods will be named _getCbsWijkenEnBuurten_ and _setCbsWijkenEnBuurten_. Note the added capital.

### Using JDBC

Spring Boot autoconfigures JdbcTemplate if you (1) add `spring-boot-starter-jdbc` or `spring-boot-starter-data-jpa` to the class path and (2) if you provide the database connection properties in application.properties. The former results in the Jdbc dependencies being on the classpath, the latter in the autoconfiguration of a DataSource bean. If this bean is available then the JdbcTemplate can and will be autoconfigured as well.

Given that the JdbcTemplate bean exists, you can inject it into a Jcbc repository class. Of course constructor injection is the most proper way to do it but @Autowired is also possible. Note that if you combine Sjpring Boot JPA with hardcore Jdbc, you need an interface for the first and a class for the latter. This repository class can look like this (see also a dedicated [blog post](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-01-spring-exam-jdbc.md):

```
@Repository
public class JdbcCustomerRepository {

	private JdbcTemplate jdbcTemplate; // store/inject this in the class so you can use it as you like.

	public JdbcCustomerRepository(JdbcTemplate jdbcTemplate){
		this.jdbcTemplate = jdbcTemplate;
	}

	public int getCostumerCount(){
		String sql = "select count(*) from customer";
		return jdbcTemplate.queryForObject(sql, Integer.class);
	}
}
```

There are multiple methods to use, like:

- queryForObject()
- update()
- queryForMap()
- queryForList()
- query()

Furthermore you can use RowMapper, ResultSetExtractor and RowCallbackHandler as arguments for more advanced use. It is all in the [blog post](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-01-spring-exam-jdbc.md). Inject the Repository bean whereever you need it, it will often be in some service class.

## Preventing duplicate table entries

For my data structure I needed a cross table named _postcode4_ with two columns (town and postal code) and actually wondered (1) how to prevent double entries and (2) whether it would need a key. ChatGPT suggested I should first add something to the @Table annotation:

```
@Entity
@Table(name = "postcodes4",
        uniqueConstraints = @UniqueConstraint(columnNames = {"pdok_postcode4", "pdok_woonplaatscode"}))
public class Postcode4 {..}
```

Upon creation of the table a constraint is placed on it, meaning it will return an error on every attempt to add a duplicate. I was surprised that the errors really showed up in the terminal, as from my Python/Sqlite days I remember that placing a 'unique' constrained simply worked without error. So I asked ChatGPT how to prevent the errors, and that should be done by not trying to insert duplicates at all. Hibernate can do that, but it requires the introduction of a so called @Embeddable Id class that holds the key to the crosstable. The primary key of this table will not be either one of the columns, but the combination of the columns.

## The @Embeddable annotation

Normally, with Spring Boot JPA, you add a primary key to a table by adding this to the @Entity-annotated domain class. It adds an id-column to your table and the table will automatically be indexed:

```
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
```

This does not work if you want your key to be made up of multiple columns. In this case, what you do in your Spring Boot code is to create an extra class annotated with @Embeddable that holds the key columns. This class takes over some of the jobs of the domain class. The following elements must be included:

- The @Embeddable annotation
- Implementation of the Serializable interface
- Implementation of both _equals()_ and _hashCode_ meyhods (the defaults don't do the job)
- The member fields of the related domain class
- Two constructors, one zero-argument and one that sets all the member fields
- Get methods for those fields

In my case, it looks like this:

```
@Embeddable
public class Postcode4EmbeddableId implements Serializable {

    private String pdokPostcode4;
    private String pdokWoonplaatscode;

    public Postcode4EmbeddableId() {}

    public Postcode4EmbeddableId(String pdokPostcode4, String pdokWoonplaatscode) {
        this.pdokPostcode4 = pdokPostcode4;
        this.pdokWoonplaatscode = pdokWoonplaatscode;
    }

    public String getPdokPostcode4() {
        return pdokPostcode4;
    }

    public String getPdokWoonplaatscode() {
        return pdokWoonplaatscode;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Postcode4EmbeddableId that)) return false;
        return Objects.equals(pdokPostcode4, that.pdokPostcode4) &&
                Objects.equals(pdokWoonplaatscode, that.pdokWoonplaatscode);
    }

    @Override
    public int hashCode() {
        return Objects.hash(pdokPostcode4, pdokWoonplaatscode);
    }
}
```

I am not sure if omitting the setters is important, I think you can add them if you like.

The presence of this @Embeddable class has consequences for the domain class that now looks like this:

```
@Entity
@Table(name = "postcodes4",
        uniqueConstraints = @UniqueConstraint(columnNames = {"pdok_postcode4", "pdok_woonplaatscode"}))
public class Postcode4 {

    @EmbeddedId
    private Postcode4EmbeddableId id;

    public Postcode4() {}

    public Postcode4(String pdokPostcode4, String pdokWoonplaatscode) {
        this.id = new Postcode4EmbeddableId(pdokPostcode4, pdokWoonplaatscode);
    }

    public Postcode4EmbeddableId getId() {
        return id;
    }

    public void setId(Postcode4EmbeddableId id) {
        this.id = id;
    }

    // convenience methods
    public String getPdokPostcode4() {
        return id.getPdokPostcode4();
    }

    public String getPdokWoonplaatscode() {
        return id.getPdokWoonplaatscode();
    }
}
```

As you see this domain class now has the @Embeddable class as dependency. 

What happens now because of this all is the Hibernate uses the embedded id as a key in its identity map. To make sure this identity map only contains references to unique objects, it needs the correct hashCode() and equals() methods. The default implementations of these do not work correctly so they need to be overridden.

Ok, I'm happy that ChatGPT is there to explain it, really don't know how I would have learned this quickly. Anyhow, the entries are unique, no errors are thrown, and the table is properly indexed which imprives performance.



