import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class RegisterService {
    public static void main(String[] args)  throws IOException {
        ServerSocket servisSocket = new ServerSocket(5000);
        System.out.println("RegisterService uruchomiony na porcie 5000...");

        while (true) {
            Socket serverSocket = servisSocket.accept();
            new Thread(() -> handleClient(serverSocket)).start();
        }
    }

    private static void handleClient(Socket clientSocket) {
        try (BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) {

            String line = in.readLine();
            String[] parts = line.split("#");

            String type = null;
            String wiadomoscId = null;
            String username = null;
            String password = null;

            // Rozdzielamy każdą część po znaku :
            for (String part : parts) {
                if (part.startsWith("type:")) {
                    type = part.split(":", 2)[1]; // Pobierz wartość po "type:"
                } else if (part.startsWith("wiadomosc_id:")) {
                    wiadomoscId = part.split(":", 2)[1]; // Pobierz wartość po "wiadomosc_id:"
                } else if (part.startsWith("login:")) {
                    username = part.split(":", 2)[1]; // Pobierz wartość po "login:"
                } else if (part.startsWith("password:")) {
                    password = part.split(":", 2)[1]; // Pobierz wartość po "password:"
                }
            }

            if (type.equals("rejestracja_request")) {
                try {
                    String result = registerUser(username, password, wiadomoscId);
                    out.println(result);
                } catch (SQLException e) {
                    out.println("Błąd serwera: " + e.getMessage());
                }
            } else {
                out.println("Nieprawidłowe żądanie!");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static String registerUser(String username, String password, String wiadomoscid) throws SQLException {
        try (Connection con = baza_danych.getConnection()) {
            // Sprawdzenie, czy użytkownik już istnieje
            String checkQuery = "SELECT COUNT(*) FROM uzytkownicy WHERE login = ?";
            try (PreparedStatement checkStmt = con.prepareStatement(checkQuery)) {
                checkStmt.setString(1, username);
                ResultSet rs = checkStmt.executeQuery();
                rs.next();
                if (rs.getInt(1) > 0) {
                    return "type:rejestracja_response wiadomosc_id:"+wiadomoscid+" Status:400";
                }
            }

            // Wstawienie nowego użytkownika
            String insertQuery = "INSERT INTO uzytkownicy (login, haslo) VALUES (?, ?)";
            try (PreparedStatement insertStmt = con.prepareStatement(insertQuery)) {
                insertStmt.setString(1, username);
                insertStmt.setString(2, password);
                insertStmt.executeUpdate();
                return "type:rejestracja_response wiadomosc_id:"+wiadomoscid+" Status:200";
            }
        }
    }


}