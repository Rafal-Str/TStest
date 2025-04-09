import java.io.*;
import java.net.*;

public class API_Gateway {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(4576);
        System.out.println("API Gateway nasłuchuje na porcie 4576...");

        while (true) {
            Socket clientSocket = serverSocket.accept();
            new Thread(() -> handleClient(clientSocket)).start();
        }
    }

    private static void handleClient(Socket clientSocket) {
        try (BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) {

            String line;
            while ((line = in.readLine()) != null) {
                //System.out.println(line);
                String[] parts = line.split("#");

                String type = null;
                if (parts[0].startsWith("type:")) {
                    type = parts[0].split(":", 2)[1]; // Pobierz wartość po "type:"
                }

                if (type.equals("rejestracja_request")) {
                    forwardRequest(line, "localhost", 5000, out);
                } else if (type.equals("login_request")) {
                    forwardRequest(line, "localhost", 5001, out);
                }else if (type.equals("wysyłanie_post_request")) {
                    forwardRequest(line, "localhost", 5002, out);
                }else if (type.equals("pobierz_posty_request")) {
                    forwardRequest(line, "localhost", 5003, out);
                }else if (type.equals("wyslij_plik_request")) {
                    FileUpload(line,"localhost",5004, in,out);
                }else if(type.equals("pobierz_plik_request")) {
                    FileDownload(line, "localhost", 5005,in, out);
                }


                else {
                    out.println("Nieznane polecenie!");
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void forwardRequest(String request, String host, int port, PrintWriter clientOut) {
        try (Socket microserviceSocket = new Socket(host, port);
             PrintWriter microserviceOut = new PrintWriter(microserviceSocket.getOutputStream(), true);
             BufferedReader microserviceIn = new BufferedReader(new InputStreamReader(microserviceSocket.getInputStream()))) {

            microserviceOut.println(request);
            String response = microserviceIn.readLine();
            System.out.println(response);
            clientOut.println(response);
        } catch (IOException e) {
            clientOut.println("Błąd połączenia z mikroserwisem: " + e.getMessage());
        }
    }

    private static void FileUpload(String request, String host, int servicePort, BufferedReader in, PrintWriter out) {
        try (Socket serviceSocket = new Socket(host, servicePort);
             PrintWriter serviceOut = new PrintWriter(serviceSocket.getOutputStream(), true);
             BufferedReader serviceIn = new BufferedReader(new InputStreamReader(serviceSocket.getInputStream()))) {

            serviceOut.println(request);

            String inputLine;
            while ((inputLine = in.readLine()) != null) {

                System.out.println(inputLine);

                if (inputLine.contains("EndOfFile:true")) {
                    serviceOut.println(inputLine);
                    break;
                }

                serviceOut.println(inputLine);
            }

            String response = serviceIn.readLine();
            if (response != null) {
                out.println(response);
            } else {
                out.println("500 Błąd przetwarzania pliku");
            }

        } catch (IOException e) {
            System.err.println("Błąd podczas przekazywania pliku: " + e.getMessage());
            out.println("500 Błąd podczas przekazywania pliku");
            e.printStackTrace();
        }
    }

    private static void FileDownload(String request, String host, int servicePort, BufferedReader in, PrintWriter out) {
        try (Socket serviceSocket = new Socket(host, servicePort);
             PrintWriter serviceOut = new PrintWriter(serviceSocket.getOutputStream(), true);
             BufferedReader serviceIn = new BufferedReader(new InputStreamReader(serviceSocket.getInputStream()))) {
            serviceOut.println(request);

            String inputLine;
            while ((inputLine = serviceIn.readLine()) != null) {
                System.out.println("Otrzymano od serwisu: " + inputLine);


                out.println(inputLine);

                if (inputLine.contains("EndOfFile:true")) {
                    out.println("Odebrano cały plik");
                    break;
                }
            }

        } catch (IOException e) {
            System.err.println("Błąd podczas pobierania pliku: " + e.getMessage());
            out.println("500 Błąd podczas pobierania pliku");
            e.printStackTrace();
        }
    }

}