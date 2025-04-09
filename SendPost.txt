import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class SendPost {
    public static void main(String[] args) throws IOException {
        ServerSocket servisSocket = new ServerSocket(5002);
        System.out.println("SendPost uruchomiony na porcie 5002...");

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
            String post = null;

            // Rozdzielamy każdą część po znaku ":"
            for (String part : parts) {
                if (part.startsWith("type:")) {
                    type = part.split(":", 2)[1];
                } else if (part.startsWith("wiadomosc_id:")) {
                    wiadomoscid = part.split(":", 2)[1];
                } else if (part.startsWith("login:")) {
                    username = part.split(":", 2)[1];
                } else if (part.startsWith("post:")) {
                    post = part.split(":", 2)[1];
                }
            }

            if ("wysyłanie_post_request".equals(type)) {
                // Wywołanie metody do zapisania posta w bazie
                boolean success = savePostToDatabase(username, post);
                if (success) {
                    out.println("type:wysyłanie_post_response#wiadomosc_id:" + wiadomoscid + "#Status:200");
                } else {
                    out.println("type:wysyłanie_post_response#wiadomosc_id:" + wiadomoscid + "#Status:400");
                }
            } else {
                out.println("Nieprawidłowe żądanie!");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static boolean savePostToDatabase(String username, String post) {
        try (Connection conn = baza_danych.getConnection()) {

            String userIdQuery = "SELECT idUzytkownika FROM uzytkownicy WHERE login = ?";
            try (PreparedStatement ps = conn.prepareStatement(userIdQuery)) {
                ps.setString(1, username);
                ResultSet rs = ps.executeQuery();
                if (rs.next()) {
                    int userId = rs.getInt("idUzytkownika");


                    String insertPostQuery = "INSERT INTO post (idUzytkownika, Opis) VALUES (?, ?)";
                    try (PreparedStatement insertPs = conn.prepareStatement(insertPostQuery)) {
                        insertPs.setInt(1, userId);
                        insertPs.setString(2, post);
                        insertPs.executeUpdate();
                        return true; // Operacja się powiodła
                    }
                } else {
                    System.err.println("Użytkownik nie istnieje: " + username);
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return false; // Operacja się nie powiodła
    }

}