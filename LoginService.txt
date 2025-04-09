import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class LoginService {
    public static void main(String[] args) throws IOException {
        ServerSocket servisSocket = new ServerSocket(5001);
        System.out.println("LoginService uruchomiony na porcie 5001...");

        while (true) {
            Socket serverSocket = servisSocket.accept();
            new Thread(() -> handleClient(serverSocket)).start();
        }
    }

    static String wiadomoscid = null;

    private static void handleClient(Socket clientSocket) {
        try (BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) {

            String line = in.readLine();
            String[] parts = line.split("#");

            String type = null;
            String username = null;
            String password = null;

            // Rozdzielamy każdą część po znaku :
            for (String part : parts) {
                if (part.startsWith("type:")) {
                    type = part.split(":", 2)[1]; // Pobierz wartość po "type:"
                } else if (part.startsWith("wiadomosc_id:")) {
                    wiadomoscid = part.split(":", 2)[1]; // Pobierz wartość po "wiadomosc_id:"
                } else if (part.startsWith("login:")) {
                    username = part.split(":", 2)[1]; // Pobierz wartość po "login:"
                } else if (part.startsWith("password:")) {
                    password = part.split(":", 2)[1]; // Pobierz wartość po "password:"
                }
            }
            if (type.equals("login_request")) {

                if (validateLogin(username, password)) {
                    out.println("type:login_response wiadomosc_id:"+wiadomoscid+" Status:200");
                } else {
                    out.println("type:login_response wiadomosc_id:"+wiadomoscid+" Status:400");
                }


            } else {
                out.println("Nieprawidłowe żądanie!");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static boolean validateLogin(String username, String password) {
        String query = "SELECT * FROM uzytkownicy WHERE login = ? AND haslo = ?";
        try (Connection con = baza_danych.getConnection();
             PreparedStatement stmt = con.prepareStatement(query)) {
            stmt.setString(1, username);
            stmt.setString(2, password);
            try (ResultSet rs = stmt.executeQuery()) {
                return rs.next(); // Jeśli dane się zgadzają, zwraca true
            }
        } catch (SQLException e) {
            System.err.println("Błąd połączenia z bazą danych: " + e.getMessage());
        }
        return false;
    }

}