//Book class
package library;

import java.io.Serializable;
import java.time.LocalDate;

public class Book implements Serializable {
    private String title;
    private String author;
    private String ISBN;
    private boolean isIssued;
    private LocalDate issueDate;

    public Book(String title, String author, String ISBN) {
        this.title = title;
        this.author = author;
        this.ISBN = ISBN;
        this.isIssued = false;
        this.issueDate = null;
    }

    public String getTitle() {
        return title;
    }

    public boolean isIssued() {
        return isIssued;
    }

    public LocalDate getIssueDate() {
        return issueDate;
    }

    public void issue(LocalDate issuedDate) {
        this.isIssued = true;
        this.issueDate = issuedDate;
    }

    public void returnBook() {
        this.isIssued = false;
        this.issueDate = null;
    }

    @Override
    public String toString() {
        return title + " by " + author + " (ISBN: " + ISBN + ")";
    }
}






//Member class
package library;

import java.io.Serializable;

public class Member implements Serializable {
    private String name;
    private String contact;
    private int memberId;
    private int penaltyPoints;

    public Member(String name, String contact, int memberId) {
        this.name = name;
        this.contact = contact;
        this.memberId = memberId;
        this.penaltyPoints = 0;
    }

    public String getName() {
        return name;
    }

    public int getPenaltyPoints() {
        return penaltyPoints;
    }

    public void addPenalty(int points) {
        this.penaltyPoints += points;
    }

    @Override
    public String toString() {
        return "Member ID: " + memberId + ", Name: " + name + ", Contact: " + contact + ", Penalty Points: " + penaltyPoints;
    }
}




//Library class
package library;

import library.exceptions.BookNotAvailableException;
import library.exceptions.BookNotIssuedException;

import java.io.*;
import java.time.LocalDate;
import java.time.temporal.ChronoUnit;
import java.util.*;

public class Library implements Serializable {
    private HashMap<String, Book> books = new HashMap<>();
    private ArrayList<Member> members = new ArrayList<>();
    private HashMap<Member, ArrayList<Book>> issuedBooks = new HashMap<>();

    public void addBook(Book book) {
        books.put(book.getTitle().toLowerCase(), book);
        System.out.println("Book added successfully!");
    }

    public void displayBooks() {
        if (books.isEmpty()) {
            System.out.println("No books available!");
            return;
        }
        int count = 1;
        for (Book book : books.values()) {
            System.out.println("Book " + count + ": " + book);
            count++;
        }
    }

    public void addMember(Member member) {
        members.add(member);
        System.out.println("Member added successfully!");
    }

    public ArrayList<Member> getMembers() {
        return members;
    }

    public ArrayList<Book> getBooks() {
        ArrayList<Book> availableBooks = new ArrayList<>();
        for (Book b : books.values()) {
            if (!b.isIssued()) {
                availableBooks.add(b);
            }
        }
        return availableBooks;
    }

    public void issueBook(Member member, Book book, LocalDate issuedDate) throws BookNotAvailableException {
        if (book.isIssued()) {
            throw new BookNotAvailableException("This book is already issued!");
        }
        book.issue(issuedDate);
        issuedBooks.computeIfAbsent(member, k -> new ArrayList<>());
        issuedBooks.get(member).add(book);
        System.out.println("Book issued to " + member.getName() + " on " + book.getIssueDate());
    }

    public void returnBook(Member member, Book book) throws BookNotIssuedException {
        if (!issuedBooks.containsKey(member) || !issuedBooks.get(member).contains(book)) {
            throw new BookNotIssuedException("This book was not issued to " + member.getName());
        }

        LocalDate returnDate = LocalDate.now();
        LocalDate issueDate = book.getIssueDate();
        long daysKept = ChronoUnit.DAYS.between(issueDate, returnDate);
        int penaltyDays = (daysKept > 7) ? (int) (daysKept - 7) : 0;
        member.addPenalty(penaltyDays);

        book.returnBook();
        issuedBooks.get(member).remove(book);

        System.out.println("Book returned by " + member.getName() + " on " + returnDate);
        if (penaltyDays > 0) {
            System.out.println("Overdue by " + penaltyDays + " days. Penalty added!");
        } else {
            System.out.println("Returned on time. No penalty.");
        }
    }

    public void saveLibraryData(String filename) throws IOException {
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(filename))) {
            out.writeObject(this);
            System.out.println("Library data saved!");
        }
    }

    public static Library loadLibraryData(String filename) throws IOException, ClassNotFoundException {
        try (ObjectInputStream in = new ObjectInputStream(new FileInputStream(filename))) {
            return (Library) in.readObject();
        }
    }
}




//Main 

package library;

import library.exceptions.BookNotAvailableException;
import library.exceptions.BookNotIssuedException;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.util.Scanner;

public class LibraryManagementSystem {
    public static void main(String[] args) {
        Library library = new Library();
        Scanner scanner = new Scanner(System.in);

        try {
            library = Library.loadLibraryData("libraryData.dat");
            System.out.println("Library data loaded successfully!");
        } catch (Exception e) {
            System.out.println("Starting a new library.");
        }

        while (true) {
            System.out.println("\n1. Add Book\n2. Display Books\n3. Add Member\n4. Issue Book\n5. Return Book\n6. Save and Exit");
            System.out.print("Enter your choice: ");
            int choice = scanner.nextInt();
            scanner.nextLine(); 

            try {
                switch (choice) {
                    case 1 -> {
                        System.out.print("Enter book title: ");
                        String title = scanner.nextLine();
                        System.out.print("Enter author: ");
                        String author = scanner.nextLine();
                        System.out.print("Enter ISBN: ");
                        String ISBN = scanner.nextLine();
                        library.addBook(new Book(title, author, ISBN));
                    }
                    case 2 -> library.displayBooks();
                    case 3 -> {
                        System.out.print("Enter member name: ");
                        String name = scanner.nextLine();
                        System.out.print("Enter contact: ");
                        String contact = scanner.nextLine();
                        System.out.print("Enter member ID: ");
                        int memberId = scanner.nextInt();
                        library.addMember(new Member(name, contact, memberId));
                    }
                    case 4 -> {
                        System.out.print("Enter Member Name: ");
                        String memberName = scanner.nextLine();
                        System.out.print("Enter Book Title to Issue: ");
                        String bookTitle = scanner.nextLine();
                        System.out.print("Enter Issued Date (yyyy-MM-dd): ");
                        LocalDate issuedDate = LocalDate.parse(scanner.nextLine(), DateTimeFormatter.ISO_LOCAL_DATE);

                        Book book = library.getBooks().stream().filter(b -> b.getTitle().equalsIgnoreCase(bookTitle)).findFirst().orElse(null);
                        Member member = library.getMembers().stream().filter(m -> m.getName().equalsIgnoreCase(memberName)).findFirst().orElse(null);

                        if (book != null && member != null) {
                            library.issueBook(member, book, issuedDate);
                        } else {
                            System.out.println("Invalid book or member!");
                        }
                    }
                    case 5 -> {
                        System.out.print("Enter Member Name: ");
                        String memberName = scanner.nextLine();
                        System.out.print("Enter Book Title to Return: ");
                        String bookTitle = scanner.nextLine();
                        Book book = library.getBooks().stream().filter(b -> b.getTitle().equalsIgnoreCase(bookTitle)).findFirst().orElse(null);
                        Member member = library.getMembers().stream().filter(m -> m.getName().equalsIgnoreCase(memberName)).findFirst().orElse(null);

                        if (book != null && member != null) {
                            library.returnBook(member, book);
                        } else {
                            System.out.println("Invalid book or member!");
                        }
                    }
                    case 6 -> {
                        library.saveLibraryData("libraryData.dat");
                        return;
                    }
                }
            } catch (Exception e) {
                System.out.println("Error: " + e.getMessage());
            }
        }
    }
}
























public class Book {
    private String title;
    private String author;
    private String isbn;

    public Book(String title, String author, String isbn) {
        this.title = title;
        this.author = author;
        this.isbn = isbn;
    }

    public String getTitle() {
        return title;
    }

    public String getAuthor() {
        return author;
    }

    public String getIsbn() {
        return isbn;
    }

    public void displayBook() {
        System.out.println("Title: " + title + ", Author: " + author + ", ISBN: " + isbn);
    }
}


import java.util.ArrayList;

public class Library {
    private ArrayList<Book> books;

    public Library() {
        books = new ArrayList<>();
    }

    public void addBook(Book book) {
        books.add(book);
        System.out.println("Book added successfully!");
    }

    public void displayAllBooks() {
        if (books.isEmpty()) {
            System.out.println("The library has no books.");
        } else {
            System.out.println("Books in the Library:");
            for (Book book : books) {
                book.displayBook();
            }
        }
    }

    public void searchBook(String keyword) {
        boolean found = false;
        for (Book book : books) {
            if (book.getTitle().toLowerCase().contains(keyword.toLowerCase()) ||
                book.getAuthor().toLowerCase().contains(keyword.toLowerCase())) {
                book.displayBook();
                found = true;
            }
        }

        if (!found) {
            System.out.println("No book found matching: " + keyword);
        }
    }
}








import java.util.Scanner;

public class LibraryManagementSystem {
    public static void main(String[] args) {
        Library library = new Library();
        Scanner scanner = new Scanner(System.in);

        while (true) {
            System.out.println("\nLibrary Management System");
            System.out.println("1. Add Book");
            System.out.println("2. Display All Books");
            System.out.println("3. Search Book");
            System.out.println("4. Exit");
            System.out.print("Choose an option: ");

            int choice = scanner.nextInt();
            scanner.nextLine(); // Consume newline

            switch (choice) {
                case 1:
                    System.out.print("Enter book title: ");
                    String title = scanner.nextLine();
                    System.out.print("Enter author: ");
                    String author = scanner.nextLine();
                    System.out.print("Enter ISBN: ");
                    String isbn = scanner.nextLine();

                    Book book = new Book(title, author, isbn);
                    library.addBook(book);
                    break;

                case 2:
                    library.displayAllBooks();
                    break;

                case 3:
                    System.out.print("Enter title or author to search: ");
                    String keyword = scanner.nextLine();
                    library.searchBook(keyword);
                    break;

                case 4:
                    System.out.println("Exiting the system...");
                    scanner.close();
                    System.exit(0);

                default:
                    System.out.println("Invalid option. Please try again.");
            }
        }
    }
}












///////////////
package library;

public class Person {
    protected String name;
    protected String contact;

    public Person(String name, String contact) {
        this.name = name;
        this.contact = contact;
    }

    @Override
    public String toString() {
        return "Name: " + name + ", Contact: " + contact;
    }
}





package library;

public class Member extends Person {
    private int memberId;

    public Member(String name, String contact, int memberId) {
        super(name, contact);
        this.memberId = memberId;
    }

    public int getMemberId() {
        return memberId;
    }

    @Override
    public String toString() {
        return "Member ID: " + memberId + ", " + super.toString();
    }
}





package library;

public class Book implements Comparable<Book> {
    private String title;
    private String author;
    private String isbn;

    public Book(String title, String author, String isbn) {
        this.title = title;
        this.author = author;
        this.isbn = isbn;
    }

    public String getTitle() {
        return title;
    }

    public String getAuthor() {
        return author;
    }

    public String getIsbn() {
        return isbn;
    }

    public void displayBook() {
        System.out.println("Title: " + title + ", Author: " + author + ", ISBN: " + isbn);
    }

    @Override
    public int compareTo(Book other) {
        return this.title.compareToIgnoreCase(other.title);
    }

    @Override
    public String toString() {
        return "Title: " + title + ", Author: " + author + ", ISBN: " + isbn;
    }
}






package library;

import java.util.*;

public class Library {
    private ArrayList<Book> books;
    private ArrayList<Member> members;
    private HashMap<Member, ArrayList<Book>> issuedBooks;

    public Library() {
        books = new ArrayList<>();
        members = new ArrayList<>();
        issuedBooks = new HashMap<>();
    }

    // Add book to library
    public void addBook(Book book) {
        books.add(book);
        System.out.println("Book added successfully!");
    }

    // Display all books
    public void displayAllBooks() {
        if (books.isEmpty()) {
            System.out.println("The library has no books.");
        } else {
            System.out.println("Books in the Library:");
            for (Book book : books) {
                System.out.println(book);
            }
        }
    }

    // Search books by title/author
    public void searchBook(String keyword) {
        boolean found = false;
        for (Book book : books) {
            if (book.getTitle().toLowerCase().contains(keyword.toLowerCase()) ||
                book.getAuthor().toLowerCase().contains(keyword.toLowerCase())) {
                System.out.println(book);
                found = true;
            }
        }

        if (!found) System.out.println("No book found matching: " + keyword);
    }

    // Add a new member
    public void addMember(Member member) {
        members.add(member);
        System.out.println("Member added successfully!");
    }

    // Issue a book to a member
    public void issueBook(Member member, Book book) {
        if (!books.contains(book)) {
            System.out.println("Book not available!");
            return;
        }

        books.remove(book);
        issuedBooks.computeIfAbsent(member, k -> new ArrayList<>()).add(book);
        System.out.println("Book issued to " + member.getName());
    }

    // Return a book from a member
    public void returnBook(Member member, Book book) {
        if (issuedBooks.containsKey(member) && issuedBooks.get(member).contains(book)) {
            issuedBooks.get(member).remove(book);
            books.add(book);
            System.out.println("Book returned successfully!");
        } else {
            System.out.println("This book wasn't issued to this member.");
        }
    }

    // Sort books by title
    public void sortBooks() {
        Collections.sort(books);
        System.out.println("Books sorted by title!");
    }

    // Display issued books for a member
    public void displayIssuedBooks(Member member) {
        if (issuedBooks.containsKey(member)) {
            System.out.println("Books issued to " + member.getName() + ": " + issuedBooks.get(member));
        } else {
            System.out.println("No books issued to this member.");
        }
    }
}







package library;

import java.util.Scanner;

public class LibraryManagementSystem {
    public static void main(String[] args) {
        Library library = new Library();
        Scanner scanner = new Scanner(System.in);

        while (true) {
            System.out.println("\nLibrary Management System");
            System.out.println("1. Add Book");
            System.out.println("2. Display All Books");
            System.out.println("3. Search Book");
            System.out.println("4. Add Member");
            System.out.println("5. Issue Book");
            System.out.println("6. Return Book");
            System.out.println("7. Display Issued Books");
            System.out.println("8. Sort Books by Title");
            System.out.println("9. Exit");
            System.out.print("Choose an option: ");

            int choice = scanner.nextInt();
            scanner.nextLine();

            switch (choice) {
                case 1:
                    System.out.print("Enter book title: ");
                    String title = scanner.nextLine();
                    System.out.print("Enter author: ");
                    String author = scanner.nextLine();
                    System.out.print("Enter ISBN: ");
                    String isbn = scanner.nextLine();
                    library.addBook(new Book(title, author, isbn));
                    break;

                case 2:
                    library.displayAllBooks();
                    break;

                case 3:
                    System.out.print("Enter title or author to search: ");
                    library.searchBook(scanner.nextLine());
                    break;

                case 4:
                    System.out.print("Enter member name: ");
                    String name = scanner.nextLine();
                    System.out.print("Enter contact: ");
                    String contact = scanner.nextLine();
                    System.out.print("Enter member ID: ");
                    int memberId = scanner.nextInt();
                    library.addMember(new Member(name, contact, memberId));
                    break;

                case 5:
                    System.out.println("Issue book functionality...");
                    break;

                case 6:
                    System.out.println("Return book functionality...");
                    break;

                case 7:
                    System.out.println("Display issued books functionality...");
                    break;

                case 8:
                    library.sortBooks();
                    break;

                case 9:
                    System.out.println("Exiting...");
                    scanner.close();
                    System.exit(0);

                default:
                    System.out.println("Invalid option. Try again.");
            }
        }
    }
}





private ArrayList<Book> books;
private ArrayList<Member> members;
private HashMap<Member, ArrayList<Book>> issuedBooks;



public void issueBook(Member member, Book book) {
    if (!books.contains(book)) {
        System.out.println("Book not available!");
        return;
    }

    // Remove book from library and add it to member's issued books
    books.remove(book);
    issuedBooks.computeIfAbsent(member, k -> new ArrayList<>()).add(book);

    System.out.println("Book issued to " + member.getName());
}




public void returnBook(Member member, Book book) {
    if (issuedBooks.containsKey(member) && issuedBooks.get(member).contains(book)) {
        issuedBooks.get(member).remove(book);
        books.add(book);
        System.out.println("Book returned successfully!");
    } else {
        System.out.println("This book wasn't issued to this member.");
    }
}




public void displayIssuedBooks(Member member) {
    if (issuedBooks.containsKey(member) && !issuedBooks.get(member).isEmpty()) {
        System.out.println("Books issued to " + member.getName() + ":");
        for (Book book : issuedBooks.get(member)) {
            System.out.println(book);
        }
    } else {
        System.out.println("No books issued to this member.");
    }
}




case 5:
    System.out.print("Enter member ID: ");
    int issueMemberId = scanner.nextInt();
    scanner.nextLine();

    Member issueMember = null;
    for (Member m : library.getMembers()) {
        if (m.getMemberId() == issueMemberId) {
            issueMember = m;
            break;
        }
    }

    if (issueMember == null) {
        System.out.println("Member not found!");
        break;
    }

    System.out.print("Enter book title to issue: ");
    String issueTitle = scanner.nextLine();

    Book issueBook = null;
    for (Book b : library.getBooks()) {
        if (b.getTitle().equalsIgnoreCase(issueTitle)) {
            issueBook = b;
            break;
        }
    }

    if (issueBook == null) {
        System.out.println("Book not found!");
    } else {
        library.issueBook(issueMember, issueBook);
    }
    break;

case 6:
    System.out.print("Enter member ID: ");
    int returnMemberId = scanner.nextInt();
    scanner.nextLine();

    Member returnMember = null;
    for (Member m : library.getMembers()) {
        if (m.getMemberId() == returnMemberId) {
            returnMember = m;
            break;
        }
    }

    if (returnMember == null) {
        System.out.println("Member not found!");
        break;
    }

    System.out.print("Enter book title to return: ");
    String returnTitle = scanner.nextLine();

    Book returnBook = null;
    for (Book b : library.getBooks()) {
        if (b.getTitle().equalsIgnoreCase(returnTitle)) {
            returnBook = b;
            break;
        }
    }

    if (returnBook == null) {
        System.out.println("Book not found!");
    } else {
        library.returnBook(returnMember, returnBook);
    }
    break;

case 7:
    System.out.print("Enter member ID: ");
    int displayMemberId = scanner.nextInt();
    scanner.nextLine();

    Member displayMember = null;
    for (Member m : library.getMembers()) {
        if (m.getMemberId() == displayMemberId) {
            displayMember = m;
            break;
        }
    }

    if (displayMember == null) {
        System.out.println("Member not found!");
    } else {
        library.displayIssuedBooks(displayMember);
    }
    break;



    public ArrayList<Book> getBooks() {
    return books;
}

public ArrayList<Member> getMembers() {
    return members;
}





public void returnBook(Member member, String bookTitle) {
    if (issuedBooks.containsKey(member)) {
        ArrayList<Book> memberBooks = issuedBooks.get(member);

        // Search for the book in the member's issued books
        Book bookToReturn = null;
        for (Book book : memberBooks) {
            if (book.getTitle().equalsIgnoreCase(bookTitle)) {
                bookToReturn = book;
                break;
            }
        }

        if (bookToReturn != null) {
            memberBooks.remove(bookToReturn);
            books.add(bookToReturn);  // Return the book to the library’s main list
            System.out.println("Book returned successfully!");
        } else {
            System.out.println("This book wasn't issued to this member.");
        }

    } else {
        System.out.println("This member has no books issued.");
    }
}




case 6:
    System.out.print("Enter member ID: ");
    int returnMemberId = scanner.nextInt();
    scanner.nextLine(); // Consume newline

    Member returnMember = null;
    for (Member m : library.getMembers()) {
        if (m.getMemberId() == returnMemberId) {
            returnMember = m;
            break;
        }
    }

    if (returnMember == null) {
        System.out.println("Member not found!");
        break;
    }

    System.out.print("Enter book title to return: ");
    String returnTitle = scanner.nextLine();

    library.returnBook(returnMember, returnTitle);
    break;



    



