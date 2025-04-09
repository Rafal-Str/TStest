import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class baza_danych {
    private static final String URL = "jdbc:mysql://localhost:3306/cnapp";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "";

    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USERNAME, PASSWORD);
    }

    public static void main(String[] args) {
        try (Connection con = getConnection()) {
            System.out.println("Połączenie z bazą danych nawiązane!");
        } catch (SQLException e) {
            System.err.println("Błąd połączenia z bazą danych: " + e.getMessage());
        }
    }
}