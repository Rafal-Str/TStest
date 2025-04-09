import java.io.*;
import java.net.*;
import java.sql.*;

public class FetchPostsService {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(5003); // Nasłuch na porcie 5004
        System.out.println("FetchPostsService uruchomiony na porcie 5003...");

        while (true) {
            Socket clientSocket = serverSocket.accept();
            new Thread(() -> handleClient(clientSocket)).start();
        }
    }

    static String wiadomoscid = null;
    private static void handleClient(Socket clientSocket) {
        try (BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) {

            String line = in.readLine();
            String[] parts = line.split("#");
            String type = null;


            // Pobieramy wartość "type" z zapytania
            for (String part : parts) {
                if (part.startsWith("type:")) {
                    type = part.split(":", 2)[1];
                }
                else if (part.startsWith("wiadomosc_id:")) {
                    wiadomoscid = part.split(":", 2)[1];
                }
            }

            // Obsługa typu żądania
            if ("pobierz_posty_request".equals(type)) {
                String postsData = fetchLatestPosts();
                System.out.println(postsData);
                out.println(postsData);
            } else {
                out.println("Nieznane żądanie!");
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static String fetchLatestPosts() {
        StringBuilder result = new StringBuilder();
        String query = "SELECT p.id AS idPost, u.login, p.Opis, p.data AS DataDodania " +
                "FROM post p " +
                "JOIN uzytkownicy u ON p.idUzytkownika = u.idUzytkownika " +
                "ORDER BY p.id DESC " +
                "LIMIT 10";

        try (Connection conn = baza_danych.getConnection();
             PreparedStatement ps = conn.prepareStatement(query);
             ResultSet rs = ps.executeQuery()) {

            while (rs.next()) {
                int postId = rs.getInt("idPost");
                String login = rs.getString("login");
                String content = rs.getString("Opis");
                Timestamp date = rs.getTimestamp("DataDodania");

                result.append("PostID: ").append(postId)
                        .append(", Autor: ").append(login)
                        .append(", Treść: ").append(content)
                        .append(", Data: ").append(date)
                        .append("## ");

            }
        } catch (SQLException e) {
            e.printStackTrace();
            return "Błąd podczas pobierania postów: " + e.getMessage();
        }
        System.out.println(result);
        return result.toString();
    }
}