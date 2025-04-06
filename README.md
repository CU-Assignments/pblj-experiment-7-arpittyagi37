import java.sql.*;
import java.util.*;

public class ProductCRUD {
    static Connection conn;
    static Scanner sc = new Scanner(System.in);

  public static void main(String[] args) throws Exception {
        conn = DriverManager.getConnection("jdbc:h2:mem:test", "sa", "");
        conn.setAutoCommit(false);
        try (Statement st = conn.createStatement()) {
            st.execute("CREATE TABLE Product (ProductID INT PRIMARY KEY, ProductName VARCHAR(100), Price DOUBLE, Quantity INT)");
            conn.commit();
        }

  while (true) {
            System.out.println("\n1.Create 2.Read 3.Update 4.Delete 5.Exit");
            switch (sc.nextInt()) {
                case 1 -> create();
                case 2 -> read();
                case 3 -> update();
                case 4 -> delete();
                case 5 -> System.exit(0);
            }
        }
    }

  static void create() {
        try (PreparedStatement ps = conn.prepareStatement("INSERT INTO Product VALUES(?, ?, ?, ?)")) {
            System.out.print("ID Name Price Qty: ");
            ps.setInt(1, sc.nextInt());
            ps.setString(2, sc.next());
            ps.setDouble(3, sc.nextDouble());
            ps.setInt(4, sc.nextInt());
            ps.executeUpdate();
            conn.commit();
            System.out.println("Inserted.");
        } catch (Exception e) { rollback(e); }
    }

  static void read() {
        try (ResultSet rs = conn.createStatement().executeQuery("SELECT * FROM Product")) {
            while (rs.next())
                System.out.printf("%d %s %.2f %d%n", rs.getInt(1), rs.getString(2), rs.getDouble(3), rs.getInt(4));
        } catch (Exception e) { rollback(e); }
    }

  static void update() {
        try (PreparedStatement ps = conn.prepareStatement("UPDATE Product SET ProductName=?, Price=?, Quantity=? WHERE ProductID=?")) {
            System.out.print("ID NewName NewPrice NewQty: ");
            int id = sc.nextInt();
            ps.setString(1, sc.next());
            ps.setDouble(2, sc.nextDouble());
            ps.setInt(3, sc.nextInt());
            ps.setInt(4, id);
            ps.executeUpdate();
            conn.commit();
            System.out.println("Updated.");
        } catch (Exception e) { rollback(e); }
    }

  static void delete() {
        try (PreparedStatement ps = conn.prepareStatement("DELETE FROM Product WHERE ProductID=?")) {
            System.out.print("ID to delete: ");
            ps.setInt(1, sc.nextInt());
            ps.executeUpdate();
            conn.commit();
            System.out.println("Deleted.");
        } catch (Exception e) { rollback(e); }
    }

  static void rollback(Exception e) {
        try { conn.rollback(); } catch (Exception ex) {}
        System.out.println("Error: " + e.getMessage());
    }
}






import java.sql.*;
import java.util.*;

public class StudentApp {
    // --- Model ---
    static class Student {
        public int id;
        public String name, dept;
        public float marks;

   public Student(int id, String name, String dept, float marks) {
            this.id = id;
            this.name = name;
            this.dept = dept;
            this.marks = marks;
        }
    }

    // --- Controller ---
  static class StudentDAO {
        private Connection conn;

  public StudentDAO() throws Exception {
            conn = DriverManager.getConnection("jdbc:sqlite:students.db");
            conn.createStatement().execute("CREATE TABLE IF NOT EXISTS student (id INT PRIMARY KEY, name TEXT, dept TEXT, marks REAL)");
        }

  public void add(Student s) throws SQLException {
            var ps = conn.prepareStatement("INSERT INTO student VALUES (?, ?, ?, ?)");
            ps.setInt(1, s.id);
            ps.setString(2, s.name);
            ps.setString(3, s.dept);
            ps.setFloat(4, s.marks);
            ps.execute();
        }

  public void update(Student s) throws SQLException {
            var ps = conn.prepareStatement("UPDATE student SET name=?, dept=?, marks=? WHERE id=?");
            ps.setString(1, s.name);
            ps.setString(2, s.dept);
            ps.setFloat(3, s.marks);
            ps.setInt(4, s.id);
            ps.execute();
        }

  public void delete(int id) throws SQLException {
            conn.prepareStatement("DELETE FROM student WHERE id=" + id).execute();
        }

  public List<Student> list() throws SQLException {
            var rs = conn.createStatement().executeQuery("SELECT * FROM student");
            var list = new ArrayList<Student>();
            while (rs.next())
                list.add(new Student(rs.getInt(1), rs.getString(2), rs.getString(3), rs.getFloat(4)));
            return list;
        }
    }

  // --- View/Main ---
    public static void main(String[] args) throws Exception {
        var sc = new Scanner(System.in);
        var dao = new StudentDAO();
        while (true) {
            System.out.println("\n1-Add 2-Update 3-Delete 4-List 0-Exit:");
            switch (sc.nextInt()) {
                case 1 -> dao.add(read(sc));
                case 2 -> dao.update(read(sc));
                case 3 -> {
                    System.out.print("Enter ID to delete: ");
                    dao.delete(sc.nextInt());
                }
                case 4 -> dao.list().forEach(s -> System.out.println(s.id + " " + s.name + " " + s.dept + " " + s.marks));
                case 0 -> {
                    System.out.println("Exiting.");
                    System.exit(0);
                }
                default -> System.out.println("Invalid choice.");
            }
        }
    }

   l static Student read(Scanner sc) {
        System.out.print("Enter ID Name Dept Marks: ");
        return new Student(sc.nextInt(), sc.next(), sc.next(), sc.nextFloat());
    }
}
