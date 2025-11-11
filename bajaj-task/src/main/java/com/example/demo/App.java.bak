package com.example.demo;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;

import java.util.HashMap;
import java.util.Map;

@SpringBootApplication
public class App implements CommandLineRunner {
    private final RestTemplate rest = new RestTemplate();
    private final ObjectMapper mapper = new ObjectMapper();

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

    @Override
    public void run(String... args) {
        try {
            // Step 1: Generate webhook
            String genUrl = "https://bfhldevapigw.healthrx.co.in/hiring/generateWebhook/JAVA";

            Map<String, String> genBody = new HashMap<>();
            genBody.put("name", "Sandhya P");
            genBody.put("regNo", "PES1UG22CS523");
            genBody.put("email", "sandhyaph15@gmail.com");

            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);

            HttpEntity<Map<String, String>> request = new HttpEntity<>(genBody, headers);
            ResponseEntity<String> genResp = rest.postForEntity(genUrl, request, String.class);

            if (genResp.getStatusCode().is2xxSuccessful() && genResp.getBody() != null) {
                Map<String, Object> respMap = mapper.readValue(genResp.getBody(),
                        new TypeReference<Map<String, Object>>() {});
                String webhook = respMap.get("webhook") != null ? respMap.get("webhook").toString() : null;
                String accessToken = respMap.get("accessToken") != null ? respMap.get("accessToken").toString() : null;

                System.out.println("generateWebhook response: " + genResp.getBody());
                System.out.println("Webhook -> " + webhook);
                System.out.println("AccessToken -> " + (accessToken != null ? "[REDACTED]" : null));

                if (webhook == null || accessToken == null) {
                    System.err.println("Missing webhook or accessToken in response. Exiting.");
                    return;
                }

                // Step 2: SQL Query
                String finalQuery = "SELECT P.AMOUNT AS SALARY, " +
                        "CONCAT(E.FIRST_NAME, ' ', E.LAST_NAME) AS NAME, " +
                        "TIMESTAMPDIFF(YEAR, E.DOB, CURDATE()) AS AGE, " +
                        "D.DEPARTMENT_NAME " +
                        "FROM PAYMENTS P " +
                        "JOIN EMPLOYEE E ON P.EMP_ID = E.EMP_ID " +
                        "JOIN DEPARTMENT D ON E.DEPARTMENT = D.DEPARTMENT_ID " +
                        "WHERE DAY(P.PAYMENT_TIME) <> 1 " +
                        "ORDER BY P.AMOUNT DESC " +
                        "LIMIT 1;";

                // Step 3: Submit the solution
                HttpHeaders submitHeaders = new HttpHeaders();
                submitHeaders.setContentType(MediaType.APPLICATION_JSON);
                submitHeaders.set("Authorization", accessToken);

                Map<String, String> submitBody = new HashMap<>();
                submitBody.put("finalQuery", finalQuery);

                HttpEntity<Map<String, String>> submitReq = new HttpEntity<>(submitBody, submitHeaders);
                ResponseEntity<String> submitResp = rest.postForEntity(webhook, submitReq, String.class);

                System.out.println("Submission status: " + submitResp.getStatusCode());
                System.out.println("Submission response body: " + submitResp.getBody());
            } else {
                System.err.println("generateWebhook failed: status=" + genResp.getStatusCode() + " body=" + genResp.getBody());
            }
        } catch (Exception e) {
            System.err.println("Exception during flow:");
            e.printStackTrace();
        } finally {
            System.exit(0);
        }
    }
}
