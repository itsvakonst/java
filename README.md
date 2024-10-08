## Reader \ Book [ManyToMany]

### Book
```
import jakarta.persistence.*;      
import java.util.HashSet;      
import java.util.Set;    

@Entity      
@Table(name = "books")     
public class Book {    
    @Id      
    @GeneratedValue(strategy = GenerationType.IDENTITY)   
    private Long id;

    @Column(nullable = false)
    private String title;

    @ManyToMany(mappedBy = "books")
    private Set<Reader> readers = new HashSet<>();

    // Constructors, getters, setters
    public Book() {}

    public Book(String title) {
        this.title = title;
    }

    public Long getId() {
        return id;
    }

    public String getTitle() {
        return title;
    }

    public Set<Reader> getReaders() {
        return readers;
    }
}
```
### Reader
```
import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "readers")
public class Reader {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @ManyToMany
    @JoinTable(
        name = "readers_books",
        joinColumns = @JoinColumn(name = "reader_id"),
        inverseJoinColumns = @JoinColumn(name = "book_id")
    )
    private Set<Book> books = new HashSet<>();

    // Constructors, getters, setters
    public Reader() {}

    public Reader(String name) {
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public Set<Book> getBooks() {
        return books;
    }

    public void addBook(Book book) {
        this.books.add(book);
        book.getReaders().add(this);
    }
}
```
### Main
```
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class LibraryManagement {
    public static void main(String[] args) {
        // Create session factory
        SessionFactory sessionFactory = new Configuration().configure().buildSessionFactory();

        // Add books and readers
        try (Session session = sessionFactory.openSession()) {
            session.beginTransaction();

            // Create books
            Book[] books = new Book[10];
            for (int i = 0; i < 10; i++) {
                books[i] = new Book("Book " + (i + 1));
                session.persist(books[i]);
            }

            // Create readers
            Reader reader1 = new Reader("Reader 1");
            Reader reader2 = new Reader("Reader 2");

            // Reader 1 borrows 7 books
            for (int i = 0; i < 7; i++) {
                reader1.addBook(books[i]);
            }

            // Reader 2 borrows 7 books
            for (int i = 3; i < 10; i++) {
                reader2.addBook(books[i]);
            }

            session.persist(reader1);
            session.persist(reader2);

            session.getTransaction().commit();
            printData(session);
        }

        // Remove a reader
        try (Session session = sessionFactory.openSession()) {
            session.beginTransaction();

            Reader reader = session.get(Reader.class, 1L);
            session.remove(reader);

            session.getTransaction().commit();
            printData(session);
        }

        sessionFactory.close();
    }

    // Helper method to print data
    private static void printData(Session session) {
        System.out.println("Books:");
        session.createQuery("from Book", Book.class).list().forEach(book -> {
            System.out.println(book.getTitle() + " - Borrowed by: " +
                    book.getReaders().stream().map(Reader::getName).toList());
        });

        System.out.println("Readers:");
        session.createQuery("from Reader", Reader.class).list().forEach(reader -> {
            System.out.println(reader.getName() + " - Borrowed books: " +
                    reader.getBooks().stream().map(Book::getTitle).toList());
        });
    }
}
```

## Album \ Customer  [ManyToMany]
### Album
```
import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "albums")
public class Album {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @ManyToMany(mappedBy = "albums")
    private Set<Customer> customers = new HashSet<>();

    // Constructors, getters, setters
    public Album() {}

    public Album(String title) {
        this.title = title;
    }

    public Long getId() {
        return id;
    }

    public String getTitle() {
        return title;
    }

    public Set<Customer> getCustomers() {
        return customers;
    }
}
```

### Customer
```
import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "customers")
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @ManyToMany
    @JoinTable(
        name = "customers_albums",
        joinColumns = @JoinColumn(name = "customer_id"),
        inverseJoinColumns = @JoinColumn(name = "album_id")
    )
    private Set<Album> albums = new HashSet<>();

    // Constructors, getters, setters
    public Customer() {}

    public Customer(String name) {
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public Set<Album> getAlbums() {
        return albums;
    }

    public void buyAlbum(Album album) {
        this.albums.add(album);
        album.getCustomers().add(this);
    }
}
```

### Main
```
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class MusicStoreManagement {
    public static void main(String[] args) {
        // Create session factory
        SessionFactory sessionFactory = new Configuration().configure().buildSessionFactory();

        // Add albums and customers
        try (Session session = sessionFactory.openSession()) {
            session.beginTransaction();

            // Create albums
            Album[] albums = new Album[6];
            for (int i = 0; i < 6; i++) {
                albums[i] = new Album("Album " + (i + 1));
                session.persist(albums[i]);
            }

            // Create customers
            Customer customer1 = new Customer("Customer 1");
            Customer customer2 = new Customer("Customer 2");

            // Customer 1 buys 4 albums
            for (int i = 0; i < 4; i++) {
                customer1.buyAlbum(albums[i]);
            }

            // Customer 2 buys 4 albums
            for (int i = 2; i < 6; i++) {
                customer2.buyAlbum(albums[i]);
            }

            session.persist(customer1);
            session.persist(customer2);

            session.getTransaction().commit();
            printData(session);
        }

        // Remove a customer
        try (Session session = sessionFactory.openSession()) {
            session.beginTransaction();

            Customer customer = session.get(Customer.class, 1L);
            session.remove(customer);

            session.getTransaction().commit();
            printData(session);
        }

        sessionFactory.close();
    }

    // Helper method to print data
    private static void printData(Session session) {
        System.out.println("Albums:");
        session.createQuery("from Album", Album.class).list().forEach(album -> {
            System.out.println(album.getTitle() + " - Bought by: " +
                    album.getCustomers().stream().map(Customer::getName).toList());
        });

        System.out.println("Customers:");
        session.createQuery("from Customer", Customer.class).list().forEach(customer -> {
            System.out.println(customer.getName() + " - Purchased albums: " +
                    customer.getAlbums().stream().map(Album::getTitle).toList());
        });
    }
}
```
![image](https://github.com/user-attachments/assets/db9c9387-18ba-4b5c-9226-5025f71cb308)
