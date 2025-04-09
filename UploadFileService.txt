import java.io.*;
import java.net.*;
import java.util.Base64;

public class UploadFileService {
    private static final String STORAGE_DIR = "uploads/";
    public static void main(String[] args) throws IOException {

        ServerSocket serverSocket = new ServerSocket(5004); // Nasłuch na porcie 5004
        System.out.println("UploadFileService uruchomiony na porcie 5004...");

        while (true) {
            Socket clientSocket = serverSocket.accept();
            new Thread(() -> handleClient(clientSocket)).start();
        }
    }

    private static void handleClient(Socket clientSocket) {
        try(
                BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
                PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true))
        {

            System.out.println("Zakceptoano polaczenie");

            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                if (inputLine.isEmpty()) {
                    continue;
                }
                String[] parts = inputLine.split("#");
                if (parts.length < 5) {
                    System.err.println("Nieprawidlowy format zaptania");
                    continue;
                }

                String type = parts[0];
                String masageid = parts[1].split(":")[1];
                String filename = parts[2].split(":")[1];
                int offset = Integer.parseInt(parts[4].split(":")[1]);
                String data = parts[5].split(":")[1];
                boolean endOfFile = inputLine.endsWith("EndOfFile:true");

                if("type:wyslij_plik_request".equalsIgnoreCase(type))
                {
                    handleUpload(filename,offset,data);
                    if(endOfFile)
                    {
                        System.out.println("Odebrano cały plik.");
                        out.println("200");
                    }
                }else
                {
                    System.err.println("Nieznany typ wiadomosci: " + type);
                }

            }
        }catch (IOException e) {
            e.printStackTrace();
        }

    }
    private static void handleUpload(String fileName, int offset, String data) {
        try {

            File file = new File(STORAGE_DIR + fileName);
            if (!file.exists()) {
                file.getParentFile().mkdirs();
                file.createNewFile();
            }

            try (RandomAccessFile fileStream = new RandomAccessFile(file, "rw")) {

                fileStream.seek(offset);


                byte[] decodedData = Base64.getDecoder().decode(data);
                fileStream.write(decodedData);


                if (data.equals("true")) {
                    System.out.println("Plik został zapisany: " + file.getAbsolutePath());
                }
            }

        } catch (IOException e) {
            e.printStackTrace();
        }


    }


}