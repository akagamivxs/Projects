package com.example.imagereading;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;
import org.springframework.beans.factory.annotation.Value;

import java.util.HashMap;
import java.util.Map;

//classe principal
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

//controlador rest
@RestController
@RequestMapping("/api")
class ImageReadingController {

    private final RestTemplate restTemplate = new RestTemplate();

    @Value("${AIzaSyCKremcl-oG5jMq9-UZxbReK9DxmKV2WCU}")
    private String geminiApiUrl;
//upload 
    @PostMapping("/upload")
    public ResponseEntity<Map<String, Object>> uploadImage(@RequestBody Map<String, Object> request) {
        String image = (String) request.get("image");
        String customerCode = (String) request.get("customer_code");
        String measureDatetime = (String) request.get("measure_datetime");
        String measureType = (String) request.get("measure_type");

        if (image == null || customerCode == null || measureDatetime == null || measureType == null ||
                (!measureType.equalsIgnoreCase("WATER") && !measureType.equalsIgnoreCase("GAS"))) {
            Map<String, String> error = new HashMap<>();
            error.put("error_code", "INVALID_DATA");
            error.put("error_description", "Dados fornecidos são inválidos");
            return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
        }

        try {
            // Chamar a api gemini
            HttpHeaders headers = new HttpHeaders();
            headers.set("Content-Type", "application/json");
            Map<String, String> body = new HashMap<>();
            body.put("image", image);
            HttpEntity<Map<String, String>> requestEntity = new HttpEntity<>(body, headers);
            ResponseEntity<Map> response = restTemplate.exchange(geminiApiUrl, HttpMethod.POST, requestEntity, Map.class);

            @SuppressWarnings("unchecked")
            Map<String, Object> responseBody = response.getBody();

            // Simulação de verificação de leitura duplicada
            boolean exists = false; // Substitua com lógica real de banco de dados
            if (exists) {
                Map<String, String> error = new HashMap<>();
                error.put("error_code", "DOUBLE_REPORT");
                error.put("error_description", "Leitura do mês já realizada");
                return new ResponseEntity<>(error, HttpStatus.CONFLICT);
            }

            return new ResponseEntity<>(responseBody, HttpStatus.OK);
        } catch (Exception e) {
            Map<String, String> error = new HashMap<>();
            error.put("error_code", "INTERNAL_ERROR");
            error.put("error_description", "Erro ao processar a imagem");
            return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    @PatchMapping("/confirm")
    public ResponseEntity<Map<String, Object>> confirmMeasurement(@RequestBody Map<String, Object> request) {
        String measureUuid = (String) request.get("measure_uuid");
        Integer confirmedValue = (Integer) request.get("confirmed_value");

        if (measureUuid == null || confirmedValue == null) {
            Map<String, String> error = new HashMap<>();
            error.put("error_code", "INVALID_DATA");
            error.put("error_description", "Dados fornecidos são inválidos");
            return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
        }

        try {
            // Simulação de verificação de leitura
            boolean measureExists = true; // Substitua com lógica real de banco de dados
            boolean alreadyConfirmed = false; // Substitua com lógica real de banco de dados
            if (!measureExists) {
                Map<String, String> error = new HashMap<>();
                error.put("error_code", "MEASURE_NOT_FOUND");
                error.put("error_description", "Leitura não encontrada");
                return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
            }
            if (alreadyConfirmed) {
                Map<String, String> error = new HashMap<>();
                error.put("error_code", "CONFIRMATION_DUPLICATE");
                error.put("error_description", "Leitura já confirmada");
                return new ResponseEntity<>(error, HttpStatus.CONFLICT);
            }

            // Simulação de salvamento da leitura
            System.out.println("Leitura confirmada: " + measureUuid + " com valor " + confirmedValue);

            Map<String, Object> response = new HashMap<>();
            response.put("success", true);
            return new ResponseEntity<>(response, HttpStatus.OK);
        } catch (Exception e) {
            Map<String, String> error = new HashMap<>();
            error.put("error_code", "INTERNAL_ERROR");
            error.put("error_description", "Erro ao confirmar a leitura");
            return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    @GetMapping("/{customer_code}/list")
    public ResponseEntity<Map<String, Object>> listMeasurements(@PathVariable String customer_code, @RequestParam(required = false) String measure_type) {
        if (measure_type != null && !measure_type.equalsIgnoreCase("WATER") && !measure_type.equalsIgnoreCase("GAS")) {
            Map<String, String> error = new HashMap<>();
            error.put("error_code", "INVALID_TYPE");
            error.put("error_description", "Tipo de medição não permitido");
            return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
        }

        try {
            // Simulação de busca de medições
            Map<String, Object> measures = new HashMap<>();
            measures.put("measure_uuid", "uuid1");
            measures.put("measure_datetime", "2024-08-27T00:00:00Z");
            measures.put("measure_type", "WATER");
            measures.put("has_confirmed", false);
            measures.put("image_url", "http://example.com/image1.jpg");

            // Filtragem opcional
            boolean filterByType = measure_type != null;
            Map<String, Object> response = new HashMap<>();
            response.put("customer_code", customer_code);
            response.put("measures", filterByType ? measures : measures);

            return new ResponseEntity<>(response, HttpStatus.OK);
        } catch (Exception e) {
            Map<String, String> error = new HashMap<>();
            error.put("error_code", "INTERNAL_ERROR");
            error.put("error_description", "Erro ao listar as medidas");
            return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}