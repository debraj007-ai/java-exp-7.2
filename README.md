import java.sql.*;
import java.util.Scanner;

public class ProductCRUD {
    private static final String URL = "jdbc:mysql://localhost:3306/productdb";
    private static final String USER = "root";
    private static final String PASSWORD = "your_password";

    public static void main(String[] args) {
        try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
             Scanner sc = new Scanner(System.in)) {
            conn.setAutoCommit(false);

            int choice;
            do {
                System.out.println("\n====== Product Management System ======");
                System.out.println("1. Add Product");
                System.out.println("2. Display All Products");
                System.out.println("3. Update Product");
                System.out.println("4. Delete Product");
                System.out.println("5. Exit");
                System.out.print("Enter your choice: ");
                choice = sc.nextInt();
                sc.nextLine(); 

                switch (choice) {
                    case 1:
                        addProduct(conn, sc);
                        break;
                    case 2:
                        displayProducts(conn);
                        break;
                    case 3:
                        updateProduct(conn, sc);
                        break;
                    case 4:
                        deleteProduct(conn, sc);
                        break;
                    case 5:
                        System.out.println("Exiting program...");
                        break;
                    default:
                        System.out.println("Invalid choice! Try again.");
                }

            } while (choice != 5);

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    private static void addProduct(Connection conn, Scanner sc) {
        System.out.print("Enter product name: ");
        String name = sc.nextLine();
        System.out.print("Enter price: ");
        double price = sc.nextDouble();
        System.out.print("Enter quantity: ");
        int quantity = sc.nextInt();

        String sql = "INSERT INTO Product (ProductName, Price, Quantity) VALUES (?, ?, ?)";
        try (PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setString(1, name);
            ps.setDouble(2, price);
            ps.setInt(3, quantity);
            ps.executeUpdate();
            conn.commit();
            System.out.println("Product added successfully!");
        } catch (SQLException e) {
            try {
                conn.rollback();
                System.out.println("Transaction rolled back due to error.");
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
            e.printStackTrace();
        }
    }


    private static void displayProducts(Connection conn) {
        String sql = "SELECT * FROM Product";
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {

            System.out.println("\n--- Product List ---");
            System.out.printf("%-10s %-20s %-10s %-10s\n", "ID", "Name", "Price", "Quantity");
            System.out.println("----------------------------------------------");

            while (rs.next()) {
                System.out.printf("%-10d %-20s %-10.2f %-10d\n",
                        rs.getInt("ProductID"),
                        rs.getString("ProductName"),
                        rs.getDouble("Price"),
                        rs.getInt("Quantity"));
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }


    private static void updateProduct(Connection conn, Scanner sc) {
        System.out.print("Enter Product ID to update: ");
        int id = sc.nextInt();
        sc.nextLine(); 

        System.out.print("Enter new product name: ");
        String name = sc.nextLine();
        System.out.print("Enter new price: ");
        double price = sc.nextDouble();
        System.out.print("Enter new quantity: ");
        int quantity = sc.nextInt();

        String sql = "UPDATE Product SET ProductName = ?, Price = ?, Quantity = ? WHERE ProductID = ?";
        try (PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setString(1, name);
            ps.setDouble(2, price);
            ps.setInt(3, quantity);
            ps.setInt(4, id);

            int rows = ps.executeUpdate();
            if (rows > 0) {
                conn.commit();
                System.out.println("Product updated successfully!");
            } else {
                System.out.println("No product found with ID " + id);
            }
        } catch (SQLException e) {
            try {
                conn.rollback();
                System.out.println("Transaction rolled back due to error.");
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
            e.printStackTrace();
        }
    }


    private static void deleteProduct(Connection conn, Scanner sc) {
        System.out.print("Enter Product ID to delete: ");
        int id = sc.nextInt();

        String sql = "DELETE FROM Product WHERE ProductID = ?";
        try (PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setInt(1, id);

            int rows = ps.executeUpdate();
            if (rows > 0) {
                conn.commit();
                System.out.println("Product deleted successfully!");
            } else {
                System.out.println("No product found with ID " + id);
            }
        } catch (SQLException e) {
            try {
                conn.rollback();
                System.out.println("Transaction rolled back due to error.");
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
            e.printStackTrace();
        }
    }
}
