import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.util.*;

class Book implements Serializable {
    int id;
    String title;
    String author;
    boolean available = true;

    Book(int id, String title, String author) {
        this.id = id;
        this.title = title;
        this.author = author;
    }

    public String toString() {
        return id + " | " + title + " | " + author + " | " + (available ? "✅ Available" : "❌ Issued");
    }
}

class Student implements Serializable {
    String username;
    String password;
    ArrayList<Integer> issuedBooks = new ArrayList<>();

    Student(String username, String password) {
        this.username = username;
        this.password = password;
    }
}

public class LibrarySystem extends JFrame {

    static ArrayList<Book> books = new ArrayList<>();
    static ArrayList<Student> students = new ArrayList<>();

    static final String BOOK_FILE = "books.dat";
    static final String STUDENT_FILE = "students.dat";
    static final String ADMIN_USER = "admin";
    static final String ADMIN_PASS = "1234";

    JLabel background;

    public LibrarySystem() {
        loadData();
        setTitle("📚 Library Management System - SIT GIRIDIH");
        setSize(800, 500);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        ImageIcon bg = new ImageIcon("background.jpg");
        Image scaled = bg.getImage().getScaledInstance(800, 500, Image.SCALE_SMOOTH);
        background = new JLabel(new ImageIcon(scaled));
        background.setLayout(new BorderLayout());
        add(background);

        // ----- LOGO ADD KIYA -----
        ImageIcon logo = new ImageIcon("logo.png");
        Image logoScaled = logo.getImage().getScaledInstance(100, 100, Image.SCALE_SMOOTH);
        JLabel logoLabel = new JLabel(new ImageIcon(logoScaled));
        logoLabel.setHorizontalAlignment(SwingConstants.CENTER);
        background.add(logoLabel, BorderLayout.NORTH);

        showMainMenu();
    }

    void showMainMenu() {
        background.removeAll();
        JPanel panel = new JPanel(new GridLayout(4, 1, 10, 10));
        panel.setOpaque(false);
        panel.setBorder(BorderFactory.createEmptyBorder(100, 200, 100, 200));

        JButton admin = makeButton("👨‍💼 Admin Login");
        JButton student = makeButton("🎓 Student Login");
        JButton register = makeButton("📝 Register Student");

        admin.addActionListener(e -> adminLogin());
        student.addActionListener(e -> studentLogin());
        register.addActionListener(e -> registerStudent());

        panel.add(admin);
        panel.add(student);
        panel.add(register);

        background.add(panel, BorderLayout.CENTER);
        revalidate();
        repaint();
        setVisible(true);
    }

    JButton makeButton(String text) {
        JButton b = new JButton(text);
        b.setBackground(new Color(0, 0, 0, 140));
        b.setForeground(Color.WHITE);
        b.setFont(new Font("Arial", Font.BOLD, 18));
        b.setFocusPainted(false);
        b.setBorder(BorderFactory.createLineBorder(Color.WHITE, 2));
        return b;
    }

    void adminLogin() {
        String user = JOptionPane.showInputDialog(this, "Enter Admin Username:");
        String pass = JOptionPane.showInputDialog(this, "Enter Admin Password:");
        if (ADMIN_USER.equals(user) && ADMIN_PASS.equals(pass)) {
            adminMenu();
        } else JOptionPane.showMessageDialog(this, "❌ Invalid Admin Credentials!");
    }

    void adminMenu() {
        background.removeAll();
        JPanel panel = new JPanel(new GridLayout(3, 1, 10, 10));
        panel.setOpaque(false);
        panel.setBorder(BorderFactory.createEmptyBorder(100, 200, 100, 200));

        JButton addBook = makeButton("➕ Add Book");
        JButton viewBooks = makeButton("📚 View Books");
        JButton back = makeButton("⬅️ Back");

        addBook.addActionListener(e -> addBookGUI());
        viewBooks.addActionListener(e -> viewBooksGUI());
        back.addActionListener(e -> showMainMenu());

        panel.add(addBook);
        panel.add(viewBooks);
        panel.add(back);

        background.add(panel);
        revalidate();
        repaint();
    }

    // ✅ BOOK ADD TEXT VISIBLE FIXED
    void addBookGUI() {
        JTextField idF = new JTextField();
        JTextField titleF = new JTextField();
        JTextField authorF = new JTextField();

        idF.setForeground(Color.BLACK);
        titleF.setForeground(Color.BLACK);
        authorF.setForeground(Color.BLACK);

        Object[] fields = {
                "Book ID:", idF,
                "Book Title:", titleF,
                "Author Name:", authorF
        };

        int option = JOptionPane.showConfirmDialog(this, fields, "Add New Book", JOptionPane.OK_CANCEL_OPTION);
        if (option == JOptionPane.OK_OPTION) {
            try {
                int id = Integer.parseInt(idF.getText());
                books.add(new Book(id, titleF.getText(), authorF.getText()));
                saveData();
                JOptionPane.showMessageDialog(this, "✅ Book Added!");
            } catch (Exception e) {
                JOptionPane.showMessageDialog(this, "⚠️ Invalid Input!");
            }
        }
    }

    void viewBooksGUI() {
        StringBuilder sb = new StringBuilder();
        for (Book b : books)
            sb.append(b).append("\n");

        JTextArea ta = new JTextArea(sb.toString());
        ta.setEditable(false);
        JOptionPane.showMessageDialog(this, new JScrollPane(ta), "📚 Books List", JOptionPane.INFORMATION_MESSAGE);
    }

    void registerStudent() {
        JTextField u = new JTextField();
        JTextField p = new JTextField();
        u.setForeground(Color.BLACK);
        p.setForeground(Color.BLACK);

        Object[] form = {
                "Username:", u,
                "Password:", p
        };

        int opt = JOptionPane.showConfirmDialog(this, form, "Register Student", JOptionPane.OK_CANCEL_OPTION);
        if (opt == JOptionPane.OK_OPTION) {
            students.add(new Student(u.getText(), p.getText()));
            saveData();
            JOptionPane.showMessageDialog(this, "✅ Student Registered!");
        }
    }

    void studentLogin() {
        String user = JOptionPane.showInputDialog(this, "Username:");
        String pass = JOptionPane.showInputDialog(this, "Password:");
        for (Student s : students)
            if (s.username.equals(user) && s.password.equals(pass)) {
                studentMenu(s);
                return;
            }
        JOptionPane.showMessageDialog(this, "❌ Invalid Credentials!");
    }

    void studentMenu(Student s) {
        background.removeAll();
        JPanel panel = new JPanel(new GridLayout(5, 1, 10, 10));
        panel.setOpaque(false);
        panel.setBorder(BorderFactory.createEmptyBorder(80, 200, 80, 200));

        JButton view = makeButton("📖 View Books");
        JButton issue = makeButton("📗 Issue Book");
        JButton ret = makeButton("📕 Return Book");
        JButton logout = makeButton("🚪 Logout");

        view.addActionListener(e -> viewBooksGUI());
        issue.addActionListener(e -> issueBookGUI(s));
        ret.addActionListener(e -> returnBookGUI(s));
        logout.addActionListener(e -> showMainMenu());

        panel.add(view);
        panel.add(issue);
        panel.add(ret);
        panel.add(logout);

        background.add(panel);
        revalidate();
        repaint();
    }

    void issueBookGUI(Student s) {
        String idStr = JOptionPane.showInputDialog(this, "Enter Book ID:");
        try {
            int id = Integer.parseInt(idStr);
            for (Book b : books)
                if (b.id == id) {
                    if (b.available) {
                        b.available = false;
                        s.issuedBooks.add(id);
                        saveData();
                        JOptionPane.showMessageDialog(this, "✅ Book Issued!");
                        return;
                    } else JOptionPane.showMessageDialog(this, "❌ Already Issued!");
                }
        } catch (Exception e) {}
    }

    void returnBookGUI(Student s) {
        String idStr = JOptionPane.showInputDialog(this, "Enter Book ID to return:");
        try {
            int id = Integer.parseInt(idStr);
            if (!s.issuedBooks.contains(id)) {
                JOptionPane.showMessageDialog(this, "❌ You didn't issue this!");
                return;
            }
            for (Book b : books)
                if (b.id == id)
                    b.available = true;

            s.issuedBooks.remove((Integer) id);
            saveData();
            JOptionPane.showMessageDialog(this, "✅ Returned!");
        } catch (Exception e) {}
    }

    @SuppressWarnings("unchecked")
    static void loadData() {
        try (ObjectInputStream o = new ObjectInputStream(new FileInputStream(BOOK_FILE))) { books = (ArrayList<Book>) o.readObject(); } catch (Exception e) {}
        try (ObjectInputStream o = new ObjectInputStream(new FileInputStream(STUDENT_FILE))) { students = (ArrayList<Student>) o.readObject(); } catch (Exception e) {}
    }

    static void saveData() {
        try (ObjectOutputStream o = new ObjectOutputStream(new FileOutputStream(BOOK_FILE))) { o.writeObject(books); } catch (Exception e) {}
        try (ObjectOutputStream o = new ObjectOutputStream(new FileOutputStream(STUDENT_FILE))) { o.writeObject(students); } catch (Exception e) {}
    }

    public static void main(String[] args) {
        new LibrarySystem();
    }
}
# my-first-project
this project was helpful to library management system
