import java.io.*;
import java.net.*;
import java.util.Base64;

public class DownloadFileService {
    private static final String STORAGE_DIR = "uploads/";

    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(5005);
        System.out.println("DownloadFileService uruchomiony na porcie 5005...");

        while (true) {
            Socket clientSocket = serverSocket.accept();
            new Thread(() -> handleClient(clientSocket)).start();
        }
    }

    private static void handleClient(Socket clientSocket) {
        try (BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) {

            String inputLine = in.readLine();
            if (inputLine == null || !inputLine.startsWith("type:pobierz_plik_request")) {
                out.println("500 Nieprawidłowe żądanie");
                return;
            }

            String[] parts = inputLine.split("#");
            String fileName = parts[2].split(":")[1];
            String wiadomosc_id = parts[1].split(":")[1];

            File file = new File(STORAGE_DIR + fileName);
            if (!file.exists()) {
                out.println("type:pobierz_plik_response wiadomosc_id:" + wiadomosc_id + " Status:400");
                return;
            }

            String requestBase = "type:pobierz_plik_response#"+"wiadomosc_id:"+wiadomosc_id +"#FileName:"+file.getName();

            try (FileInputStream fis = new FileInputStream(file)) {
                long fileSize = file.length();
                byte[] buffer = new byte[512];
                int bytesRead;
                long offset = 0;

                while ((bytesRead = fis.read(buffer)) != -1) {
                    String encodedData = Base64.getEncoder().encodeToString(buffer);
                    String request = requestBase + "#FileSize:" + fileSize + "#Offset:" + offset + "#Data:" + encodedData;
                    out.println(request);
                    out.flush();
                    offset += bytesRead;
                }
                out.println(requestBase + "#FileSize:" + fileSize + "#Offset:" + offset + "#EndOfFile:true");
                out.flush();

            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}