<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>API List Management</title>
</head>
<body style="font-family: Arial, sans-serif; background-color: #f4f6f8; margin: 0; padding: 0;">
    <div style="width: 90%; margin: 20px auto; background-color: #ffffff; padding: 20px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);">
        <h1 style="color: #005288; font-size: 24px; text-align: center;">API List Management</h1>
        
        <div style="margin-bottom: 20px; text-align: center;">
            <input type="text" id="searchInput" placeholder="Search by API Link" oninput="searchApi()" style="padding: 8px; width: 50%; border: 1px solid #ddd; border-radius: 4px;">
        </div>

        <form id="addForm" style="display: flex; justify-content: space-between; margin-bottom: 20px;">
            <input type="text" id="apiLink" placeholder="API Link" required style="padding: 8px; width: 30%; margin-right: 10px; border: 1px solid #ddd; border-radius: 4px;">
            <input type="email" id="mailContact" placeholder="Mail Contact Point" required style="padding: 8px; width: 30%; margin-right: 10px; border: 1px solid #ddd; border-radius: 4px;">
            <input type="text" id="lastMaintainer" placeholder="Last Maintainer" required style="padding: 8px; width: 30%; margin-right: 10px; border: 1px solid #ddd; border-radius: 4px;">
            <button type="submit" style="background-color: #005288; color: white; padding: 8px 12px; border: none; border-radius: 4px; cursor: pointer; font-size: 14px;">Add API</button>
        </form>
        
        <table id="apiTable" style="width: 100%; border-collapse: collapse; margin-top: 20px;">
            <thead>
                <tr style="background-color: #005288; color: white; font-weight: bold;">
                    <th style="padding: 12px 15px; text-align: left; border-bottom: 1px solid #ddd;">API Link</th>
                    <th style="padding: 12px 15px; text-align: left; border-bottom: 1px solid #ddd;">Mail Contact Point</th>
                    <th style="padding: 12px 15px; text-align: left; border-bottom: 1px solid #ddd;">Last Maintainer</th>
                    <th style="padding: 12px 15px; text-align: left; border-bottom: 1px solid #ddd;">Actions</th>
                </tr>
            </thead>
            <tbody style="background-color: #f9f9f9;"></tbody>
        </table>

        <div style="display: flex; justify-content: center; margin-top: 20px;">
            <button id="prevPage" onclick="changePage(-1)" style="background-color: #f4f6f8; border: 1px solid #ddd; color: #005288; padding: 8px 12px; cursor: pointer; margin: 0 5px;">Previous</button>
            <button id="nextPage" onclick="changePage(1)" style="background-color: #f4f6f8; border: 1px solid #ddd; color: #005288; padding: 8px 12px; cursor: pointer; margin: 0 5px;">Next</button>
        </div>
    </div>

    <script>
        let apiList = [];
        let filteredList = [];
        let currentPage = 1;
        const itemsPerPage = 5;

        async function fetchData() {
            try {
                const response = await fetch('/api/getApis');
                apiList = await response.json();
                filteredList = apiList;
                updateTable();
            } catch (error) {
                console.error("Error fetching data:", error);
            }
        }

        function updateTable() {
            const tableBody = document.querySelector("#apiTable tbody");
            tableBody.innerHTML = '';
            const start = (currentPage - 1) * itemsPerPage;
            const end = start + itemsPerPage;
            const currentData = filteredList.slice(start, end);
            
            currentData.forEach(api => {
                const row = document.createElement("tr");
                row.innerHTML = `
                    <td>${api.link}</td>
                    <td>${api.contact}</td>
                    <td>${api.maintainer}</td>
                    <td><button onclick="editApi(${api.id})">Edit</button></td>
                `;
                tableBody.appendChild(row);
            });
        }

        function searchApi() {
            const searchValue = document.querySelector("#searchInput").value.toLowerCase();
            filteredList = apiList.filter(api => api.link.toLowerCase().includes(searchValue));
            currentPage = 1;
            updateTable();
        }

        function changePage(direction) {
            currentPage += direction;
            updateTable();
        }

        document.querySelector("#addForm").addEventListener("submit", async function(event) {
            event.preventDefault();
            const apiLink = document.querySelector("#apiLink").value;
            const mailContact = document.querySelector("#mailContact").value;
            const lastMaintainer = document.querySelector("#lastMaintainer").value;
            await fetch('/api/addApi', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ link: apiLink, contact: mailContact, maintainer: lastMaintainer })
            });
            fetchData();
            event.target.reset();
        });

        fetchData();
    </script>
</body>
</html>



package com.example.apilist;

import jakarta.persistence.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import java.util.List;

@SpringBootApplication
public class ApiListApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiListApplication.class, args);
    }
}

@Entity
@Table(name = "api_list")
class ApiInfo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String link;
    private String contact;
    private String maintainer;
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getLink() { return link; }
    public void setLink(String link) { this.link = link; }
    public String getContact() { return contact; }
    public void setContact(String contact) { this.contact = contact; }
    public String getMaintainer() { return maintainer; }
    public void setMaintainer(String maintainer) { this.maintainer = maintainer; }
}

interface ApiInfoRepository extends JpaRepository<ApiInfo, Long> {
    List<ApiInfo> findByLinkContainingIgnoreCase(String link);
}

@RestController
@RequestMapping("/api")
class ApiController {
    @Autowired
    private ApiInfoRepository repository;

    @GetMapping("/getApis")
    public List<ApiInfo> getApis() {
        return repository.findAll();
    }

    @PostMapping("/addApi")
    public ApiInfo addApi(@RequestBody ApiInfo apiInfo) {
        return repository.save(apiInfo);
    }

    @PutMapping("/editApi/{id}")
    public ApiInfo editApi(@PathVariable Long id, @RequestBody ApiInfo updatedApi) {
        return repository.findById(id).map(api -> {
            api.setLink(updatedApi.getLink());
            api.setContact(updatedApi.getContact());
            api.setMaintainer(updatedApi.getMaintainer());
            return repository.save(api);
        }).orElseThrow(() -> new RuntimeException("API not found"));
    }

    @GetMapping("/search")
    public List<ApiInfo> searchApi(@RequestParam String query) {
        return repository.findByLinkContainingIgnoreCase(query);
    }
}


CREATE TABLE api_list (
    id SERIAL PRIMARY KEY,
    link VARCHAR(255) NOT NULL,
    contact VARCHAR(255) NOT NULL,
    maintainer VARCHAR(255) NOT NULL
);
