import java.io.*;
import java.net.*;
import java.util.Base64;
public class CLI {
    private static final String STORAGE_DIR = "download/";

    public static void main(String[] args) {
        try (Socket socket = new Socket("localhost", 4576);
             BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
             BufferedReader sc = new BufferedReader(new InputStreamReader(System.in))) {

            boolean running = true;
            boolean boollogin = false;
            String username = "";
            String password = "";
            String wybor = "";
            String filePath = "";
            int wiadomosc_id = 1;
            String linie = null;
            String parts[] = null;
            String post = null;
            String login_true = null;

            while (running) {
                if(boollogin!=true)
                {   System.out.println("Wybierz operacje programu ");
                    System.out.println("1. Rejestracja");
                    System.out.println("2. Logowanie");
                    System.out.println("3. Wyjście");


                    wybor = sc.readLine();
                    switch (wybor) {

                        case "1":
                            System.out.print("Podaj nazwę użytkownika: ");
                            username = sc.readLine();
                            System.out.print("Podaj hasło: ");
                            password = sc.readLine();
                            out.println("type:rejestracja_request#"+"wiadomosc_id:" +wiadomosc_id+"#login:"+username+"#password:"+password);
                            linie = in.readLine();
                            System.out.println(linie);
                            parts = linie.split(" ");
                            // System.out.println(parts[2]);
                            login_true = parts[2];
                            boollogin = true;
                            wiadomosc_id++;
                            break;
                        case "2":
                            System.out.print("Podaj nazwę użytkownika: ");
                            username = sc.readLine();
                            System.out.print("Podaj hasło: ");
                            password = sc.readLine();
                            out.println("type:login_request#"+"wiadomosc_id:" +wiadomosc_id+"#login:"+username+"#password:"+password);
                            linie = in.readLine();
                            System.out.println(linie);
                            parts = linie.split(" ");
                            // System.out.println(parts[2]);
                            login_true = parts[2];
                            boollogin = true;
                            wiadomosc_id++;
                            break;
                        case "3":
                            running = false;
                            break;
                        default:
                            System.out.println("Nieprawidłowa opcja!");
                            break;
                    }
                }
                else if(boollogin==true && login_true.equals("Status:200") && username!=null)
                {
                    System.out.println("Wybierz operacje programu ");
                    System.out.println("1. Wyslij post");
                    System.out.println("2. Wyswietl 10 ostatnich postow");
                    System.out.println("3. Wyslij plik na serwer");
                    System.out.println("4. Pobierz plik na serwera");
                    System.out.println("6. Wyloguj sie");
                    System.out.println("5. Wyjście");

                    wybor = sc.readLine();
                    switch (wybor) {
                        case "1":
                            System.out.println("Podaj tresc posta");
                            post = sc.readLine();
                            out.println("type:wysyłanie_post_request#"+"wiadomosc_id:" +wiadomosc_id+"#login:"+username+"#post:"+post);
                            System.out.println(in.readLine());
                            wiadomosc_id++;
                            break;
                        case "2":
                            out.println("type:pobierz_posty_request#"+"wiadomosc_id:" +wiadomosc_id+"#login:"+username);
                            String response;
                            if ((response = in.readLine()) != null) {
                                String[] posts = response.split("##");
                                for (String post2 : posts) {
                                    System.out.println(post2);
                                }
                            }
                            wiadomosc_id++;
                            break;
                        case "3":

                            System.out.println("Podaj ścieżkę do pliku do wysłania:");
                            filePath = sc.readLine();
                            File plik = new File(filePath);

                            if (!plik.exists() ) {
                                System.out.println("Plik nie istnieje.");
                                return;
                            }
                            String requestBase = "type:wyslij_plik_request#"+"wiadomosc_id:"+wiadomosc_id +"#FileName:"+plik.getName();
                            try (FileInputStream fis = new FileInputStream(plik)) {
                                long fileSize = plik.length();
                                System.out.println("Plik w trakcie wysyłania (" + fileSize + " bajtów)...");
                                byte[] buffer = new byte[512];
                                int bytesRead;
                                long bytesSent = 0;
                                while ((bytesRead = fis.read(buffer)) != -1) {
                                    String encodedData = Base64.getEncoder().encodeToString(buffer);
                                    String request = requestBase + "#FileSize:" + fileSize + "#Offset:" + bytesSent + "#Data:" + encodedData;
                                    out.println(request);
                                    out.flush();
                                    bytesSent += bytesRead;
                                }
                                out.println(requestBase + "#FileSize:" + fileSize + "#Offset:" + bytesSent + "#EndOfFile:true");
                                out.flush();
                                System.out.println("Plik wysłany.");
                                String response2 = in.readLine();
                                if (response2 != null && response2.contains("200")) {
                                    System.out.println("Plik wysłany pomyślnie.");
                                } else {
                                    System.out.println("Błąd wysyłania pliku: " + response2);
                                }
                            }
                            wiadomosc_id++;
                            break;
                        case "4":
                            System.out.println("Podaj nazwę pliku, który chcesz pobrać:");
                            String fileName = sc.readLine();
                            out.println("type:pobierz_plik_request#wiadomosc_id:" + wiadomosc_id + "#FileName:" + fileName);
                            String inputLine;
                            while ((inputLine = in.readLine()) != null) {
                                if (inputLine.isEmpty()) {
                                    continue;
                                }
                                parts = inputLine.split("#");
                                if (parts.length < 5) {
                                    System.err.println("Nieprawidłowy format zaptania");
                                    continue;
                                }
                                String type = parts[0];
                                String masageid = parts[1].split(":")[1];
                                String filename = parts[2].split(":")[1];
                                int offset = Integer.parseInt(parts[4].split(":")[1]);
                                String data = parts[5].split(":")[1];
                                boolean endOfFile = inputLine.endsWith("EndOfFile:true");
                                if("type:pobierz_plik_response".equalsIgnoreCase(type))
                                {
                                    handleUpload(filename,offset,data);
                                    if(endOfFile)
                                    {
                                        System.out.println(in.readLine());
                                        break;
                                    }
                                }else
                                {
                                    System.err.println("Nieznany typ wiadomosci: " + type);
                                }
                            }
                            wiadomosc_id++;
                            break;
                        case "5":
                            running = false;
                            break;
                        case "6":
                            boollogin=false;
                            username = null;
                            password = null;
                        default:
                            break;
                    }
                }
            }
        } catch (IOException e) {
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